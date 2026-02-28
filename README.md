<div align="center">

# 🤖 Model-Based AI Agent

**Agente de IA basado en modelos con estado persistente, orquestación n8n y canal WhatsApp**

<br/>

**No es un chatbot. Es un agente que razona, recuerda y decide.**

[Arquitectura](#-arquitectura) •
[Quick Start](#-quick-start) •
[Cómo Funciona](#-cómo-funciona) •
[API Reference](#-api-reference) •
[Configuración](#%EF%B8%8F-configuración) •
[Roadmap](#-roadmap)

</div>

---

## 📋 Tabla de Contenidos

- [Descripción](#-descripción)
- [¿Por qué un agente basado en modelos?](#-por-qué-un-agente-basado-en-modelos)
- [Arquitectura](#-arquitectura)
- [Tech Stack](#-tech-stack)
- [Quick Start](#-quick-start)
- [Estructura del Proyecto](#-estructura-del-proyecto)
- [Cómo Funciona](#-cómo-funciona)
- [API Reference](#-api-reference)
- [Flujos n8n](#-flujos-n8n)
- [Políticas de Decisión](#-políticas-de-decisión)
- [Human-in-the-Loop](#-human-in-the-loop)
- [Observabilidad y Métricas](#-observabilidad-y-métricas)
- [Configuración](#%EF%B8%8F-configuración)
- [Testing](#-testing)
- [Roadmap](#-roadmap)
- [Contribución](#-contribución)
- [Licencia](#-licencia)

---

## 📖 Descripción

**Model-Based AI Agent** es un sistema de agente inteligente que va más allá de un chatbot reactivo. Mantiene un **modelo interno del mundo** (estado del usuario, contexto, hechos conocidos), toma **decisiones estructuradas** basadas en reglas + LLM, y escala a humanos cuando es necesario.

| Característica | Descripción |
|---|---|
| 🧠 **Estado persistente** | Modelo mental del usuario almacenado en PostgreSQL |
| ⚡ **Decisiones estructuradas** | Salida JSON validada con Pydantic + Instructor |
| 🛡️ **Guardrails deterministas** | Reglas pre-LLM y post-LLM que controlan al agente |
| 👤 **Human-in-the-Loop** | Escalamiento automático con contexto completo |
| 📊 **Observabilidad total** | Cada decisión queda logueada y auditable |
| 🔄 **Orquestación n8n** | Flujos visuales para integrar canales y servicios |
| 📱 **WhatsApp nativo** | Canal de entrada/salida vía Business Cloud API |
| 🐳 **100% containerizado** | Docker Compose para desarrollo y producción |

---

## 🤔 ¿Por qué un agente basado en modelos?

| Chatbot Tradicional | Este Agente (Model-Based) |
|---|---|
| ❌ Sin memoria entre mensajes | ✅ Estado persistente por usuario |
| ❌ Responde solo al input actual | ✅ Decide con: input + estado + reglas + objetivos |
| ❌ Sin control sobre el LLM | ✅ Guardrails pre/post LLM |
| ❌ Sin escalamiento inteligente | ✅ Escala a humanos con contexto completo |
| ❌ Caja negra | ✅ Cada decisión es auditable |

> **Referencia teórica:** Este diseño implementa el patrón *Model-Based Reflex Agent* de Russell & Norvig (Artificial Intelligence: A Modern Approach), donde el agente mantiene un modelo interno del mundo para tomar mejores decisiones.

---

## 🏗 Arquitectura

          ┌─────────────────────────────────────────────┐
               │              DOCKER COMPOSE                  │
               │                                             │
┌──────────┐ │ ┌─────────┐ ┌──────────────────────┐ │ │ │ msg │ │ │ │ AGENT SERVICE │ │ │ WhatsApp ├──────┼─►│ n8n ├───►│ (FastAPI) │ │ │ Business │ │ │ :5678 │ │ :8000 │ │ │ API │◄─────┼──┤ │◄───┤ │ │ │ │ resp │ │ │ │ ┌────────────────┐ │ │ └──────────┘ │ └────┬────┘ │ │ Decision Engine│ │ │ │ │ │ │ │ │ │ │ │ │ │ Pre-LLM Rules │ │ │ ┌──────────┐ │ │ │ │ LLM (OpenAI) │ │ │ │ Actor │◄─────┼───────┘ │ │ Post-LLM Rules │ │ │ │ Humano │ │ (escalate) │ └───────┬────────┘ │ │ │ Slack/ ├──────┼────────────────┼──────────┘ │ │ │ Email │ │ └──────────┬───────────┘ │ └──────────┘ │ │ │ │ ┌──────▼──────┐ │ │ │ PostgreSQL │ │ │ │ :5432 │ │ │ │ │ │ │ │ • state │ │ │ │ • history │ │ │ │ • decisions │ │ │ │ • escalations│ │ │ └─────────────┘ │ └─────────────────────────────────────────────┘

### Flujo de datos

Usuario envía mensaje (WhatsApp) │ ▼ ┌─────────┐ │ n8n │ ← Webhook trigger └────┬────┘ │ ▼ ┌──────────────────────────────────────────┐ │ AGENT SERVICE PIPELINE │ │ │ │ 1. PERCEPTION → Normaliza input │ │ 2. STATE MANAGER → Lee estado de DB │ │ 3. PRE-LLM RULES → Filtros/bypass │ │ 4. REASONER → LLM + prompt │ │ 5. POST-LLM RULES → Validación │ │ 6. ACTION SELECT → reply|ask|escalate │ │ 7. PERSIST → Guarda nuevo estado │ │ 8. LOG → Registra decisión │ │ │ └────────────┬─────────────────────────────┘ │ ▼ ┌─────────────────┐ │ n8n Switch │ │ │ │ reply/ask ──► WhatsApp (respuesta) │ escalate ──► Slack/Email (humano) └─────────────────┘



---

## 🛠 Tech Stack

| Capa | Tecnología | Propósito |
|------|-----------|-----------|
| **Agente** | Python 3.12 + FastAPI | Servicio core del agente |
| **Validación** | Pydantic v2 | Modelos de datos y validación |
| **LLM** | OpenAI GPT-4o + Instructor | Razonamiento con salida estructurada |
| **Base de datos** | PostgreSQL 16 | Estado persistente y logs |
| **ORM** | SQLAlchemy 2.0 | Acceso a datos |
| **Orquestación** | n8n | Flujos, webhooks, integraciones |
| **Canal** | WhatsApp Business Cloud API | Comunicación con usuarios |
| **Contenedores** | Docker + Docker Compose | Infraestructura reproducible |
| **Notificaciones** | Slack / Email (vía n8n) | Alertas de escalamiento |

---

## 🚀 Quick Start

### Prerrequisitos

- [Docker](https://docs.docker.com/get-docker/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) 2.0+
- API Key de OpenAI (o proveedor LLM compatible)
- WhatsApp Business API credentials (para integración completa)

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/model-based-agent.git
cd model-based-agent

