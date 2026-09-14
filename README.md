# 🥗 NutriSearch — Bot Nutricional Inteligente con n8n, Selenium & RAG

**NutriSearch** es una plataforma integral y automatizada para la asistencia nutricional, comparación de alimentos y generación de planes de alimentación personalizados basada en el catálogo real de supermercados (ej. *Dia*, *Ahorramas*)[cite: 10, 12, 13]. 

El sistema utiliza flujos de trabajo en **n8n**, un motor de extracción visual con **Selenium** y visión artificial multimodal con **Llama-4 en Groq**, bases de datos internas en n8n Data Tables, y un agente conversacional **RAG (Retrieval-Augmented Generation)** desplegado en **Telegram** con integración a **Google Calendar** y **SMTP (Email)**.

---

## 📋 Tabla de Contenidos
1. [Características Principales](#-características-principales)
2. [Arquitectura del Sistema](#-arquitectura-del-sistema)
3. [Flujos de Trabajo (Workflows de n8n)](#-flujos-de-trabajo-workflows-de-n8n)
   - [1. Ingesta y Web Scraping Masivo](#1-ingesta-y-web-scraping-masivo)
   - [2. Motor de Extracción Visual (Selenium + Groq Llama-4)](#2-motor-de-extracción-visual-selenium--groq-llama-4)
   - [3. Normalización y Scoring Nutricional](#3-normalización-y-scoring-nutricional)
   - [4. Bot Nutricional en Telegram v2 (RAG Agent)](#4-bot-nutricional-en-telegram-v2-rag-agent)
4. [Estructura de Datos](#-estructura-de-datos)
5. [Requisitos Previos e Instalación](#-requisitos-previos-e-instalación)
6. [Configuración de Credenciales](#-configuración-de-credenciales)
7. [Buenas Prácticas de Seguridad y Roadmap](#-buenas-prácticas-de-seguridad-y-roadmap)

---

## 🚀 Características Principales

- **Web Scraping Anti-Bot:** Extracción mediante navegador automatizado (*Selenium Standalone Chrome*) con emulación de navegador real, evasión de `navigator.webdriver` e inyección de cookies de sesión.
- **Extracción Multimodal con IA:** Conversión de capturas de pantalla de fichas de producto en datos estructurados (`titulo`, `precio`, `proteínas`, `calorías`, `grasas`, etc.) usando `meta-llama/llama-4-scout-17b-16e-instruct` a través de Groq.
- **Pipeline de Limpieza y Normalización:** Limpieza de datos no numéricos, símbolos de moneda y generación de campos de búsqueda indexables.
- **Score Nutricional Automatizado:** Cálculo algorítmico (0 a 100) ponderando proteínas, grasas, calorías y fibra.
- **Detección de Intenciones en Lenguaje Natural:** Clasificación de consultas en:
  - 🔍 **Búsqueda:** Localización de productos y alternativas saludables.
  - ⚖️ **Comparación:** Análisis nutricional comparativo con tabla de macros y ganador.
  - 🎯 **Objetivos Específicos:** Recomendaciones para pérdida de grasa, hipertrofia, dietas cetogénicas, veganas, sin gluten, etc.
  - 🏆 **Top Productos:** Rankings por precio, mayor densidad proteica o menor valor calórico.
  - 📅 **Planes Semanales:** Generación de menús de 7 días con desglose diario (desayuno, comida, cena) y coste estimado.
  - 📧 **Exportación Omnicanal:** Sincronización automática con eventos de Google Calendar y envío del menú por correo electrónico (SMTP)[cite: 10, 13].

---

## 🏗 Arquitectura del Sistema

```text
  ┌─────────────────────────────────────────────────────────────┐
  │ 1. NutriSearch - Dia Scraper Masivo                         │
  │    (Lista de URLs de categorías -> Batching -> Webhook)      │
  └──────────────────────────────┬──────────────────────────────┘
                                 │ HTTP POST
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ 2. My workflow 3 (Ultimate Scraper Engine)                  │
  │    - Selenium Session (Headless Chrome + Bypass)            │
  │    - Screenshot (Base64)                                    │
  │    - Groq Vision (Llama-4 Scout Multimodal)                 │
  │    - Parseo de Atributos & Inserción                        │
  └──────────────────────────────┬──────────────────────────────┘
                                 │ Inserta Registros
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ n8n Data Table: "datos" (Catálogo Nutricional)              │
  └──────────────────────────────┬──────────────────────────────┘
                                 │
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ 3. NutriSearch - Preprocessing                              │
  │    - Sanitización y conversión de tipos numéricos           │
  │    - Cálculo de 'score_nutricional'                         │
  │    - Generación de 'texto_busqueda'                         │
  └──────────────────────────────┬──────────────────────────────┘
                                 │ Catálogo Limpio
                                 ▼
  ┌─────────────────────────────────────────────────────────────┐
  │ 4. NutriSearch — Telegram Bot Nutricional v2                │
  │    - Telegram Trigger -> Clasificador de Intención          │
  │    - Context Builder (Inyección de productos relevantes)    │
  │    - Agente LangChain + Groq LLM + Buffer Memory            │
  │    - Generador de Enlace Google Calendar                    │
  │    - Notificaciones Telegram / Envío de Email (SMTP)        │
  └─────────────────────────────────────────────────────────────┘
