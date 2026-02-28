# 🤖 Model-Based AI Agent

Agente de IA basado en modelos con estado persistente, orquestación n8n y canal WhatsApp Business. Este agente no es un chatbot tradicional: **razona, recuerda y decide** usando un modelo mental del usuario, reglas y LLM, con escalamiento inteligente a humanos.

[Arquitectura](#-arquitectura) • [Quick Start](#-quick-start) • [Cómo Funciona](#-cómo-funciona) • [API Reference](#-api-reference) • [Configuración](#%EF%B8%8F-configuración) • [Roadmap](#-roadmap)

---

## 📋 Tabla de Contenidos

- [Descripción](#-descripción)
- [¿Por qué un agente basado en modelos?](#-por-qué-un-agente-basado-en-modelos)
- [Comparación: Chatbot vs Agente](#-comparación-chatbot-vs-agente)
- [Arquitectura](#-arquitectura)
- [Flujo de Datos](#-flujo-de-datos)
- [Pipeline de Decisión](#-pipeline-de-decisión)
- [Persistencia de Estado](#-persistencia-de-estado)
- [Tech Stack](#-tech-stack)
- [Quick Start](#-quick-start)
- [Roadmap](#-roadmap)
- [Contribución](#-contribución)
- [Licencia](#-licencia)

---

## 📖 Descripción

**Model-Based AI Agent** es un sistema de agente inteligente que mantiene un **modelo interno del mundo** (estado del usuario, contexto, hechos conocidos), toma **decisiones estructuradas** basadas en reglas y LLM, y escala a humanos cuando es necesario. El estado, la historia y las decisiones se almacenan en PostgreSQL, permitiendo trazabilidad y auditoría total.

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

El agente implementa el patrón *Model-Based Reflex Agent* de Russell & Norvig, manteniendo un modelo interno del mundo para tomar mejores decisiones y escalar a humanos cuando es necesario.

---

## 🆚 Comparación: Chatbot vs Agente

```mermaid
flowchart TD
    subgraph Comparación
        Chatbot[Chatbot Tradicional]
        Agente[Agente Model-Based]
    end
    Chatbot-->|Sin memoria|Agente
    Chatbot-->|Responde solo al input|Agente
    Chatbot-->|Sin control sobre LLM|Agente
    Chatbot-->|Sin escalamiento|Agente
    Chatbot-->|Caja negra|Agente
    Agente-->|Estado persistente|Chatbot
    Agente-->|Decide con reglas|Chatbot
    Agente-->|Guardrails|Chatbot
    Agente-->|Escala a humanos|Chatbot
    Agente-->|Auditable|Chatbot
```

---

## 🏗 Arquitectura

```mermaid
flowchart TD
    subgraph Docker Compose
        AGENT_SERVICE(FastAPI)
        n8n[n8n]
        WhatsApp[WhatsApp Business API]
        PostgreSQL[(PostgreSQL)]
        LLM(OpenAI GPT-4o)
        Slack[Slack/Email]
    end
    WhatsApp-->|msg|n8n
    n8n-->|trigger|AGENT_SERVICE
    AGENT_SERVICE-->|state/history/decisions|PostgreSQL
    AGENT_SERVICE-->|reasoning|LLM
    AGENT_SERVICE-->|reply|n8n
    AGENT_SERVICE-->|escalate|Slack
    n8n-->|response|WhatsApp
    AGENT_SERVICE-->|log|PostgreSQL
```

---

## 🔄 Flujo de Datos

```mermaid
flowchart TD
    Usuario-->|mensaje|WhatsApp
    WhatsApp-->|Webhook|n8n
    n8n-->|Pipeline|AGENT_SERVICE
    AGENT_SERVICE-->|reply|n8n
    AGENT_SERVICE-->|escalate|Slack
    n8n-->|respuesta|WhatsApp
    n8n-->|alerta|Slack
    AGENT_SERVICE-->|persistencia|PostgreSQL
```

---

## 🧩 Pipeline de Decisión

```mermaid
flowchart TD
    subgraph Pipeline
        Perception[1. Perception]
        StateManager[2. State Manager]
        PreLLM[3. Pre-LLM Rules]
        Reasoner[4. Reasoner (LLM)]
        PostLLM[5. Post-LLM Rules]
        ActionSelect[6. Action Select]
        Persist[7. Persist]
        Log[8. Log]
    end
    Perception-->StateManager-->PreLLM-->Reasoner-->PostLLM-->ActionSelect-->Persist-->Log
```

---

## 🗄️ Persistencia de Estado

```mermaid
flowchart TD
    subgraph Persistencia
        Usuario[Usuario]
        Estado[Estado]
        Historia[Historia]
        Decisiones[Decisiones]
        Escalaciones[Escalaciones]
        PostgreSQL[(PostgreSQL)]
    end
    Usuario-->|actualiza|Estado
    Usuario-->|genera|Historia
    Usuario-->|provoca|Decisiones
    Usuario-->|provoca|Escalaciones
    Estado-->|guarda|PostgreSQL
    Historia-->|guarda|PostgreSQL
    Decisiones-->|guarda|PostgreSQL
    Escalaciones-->|guarda|PostgreSQL
```

---

## 🛠 Tech Stack

```mermaid
flowchart TD
    subgraph TechStack
        Python[Python 3.12 + FastAPI]
        Pydantic[Pydantic v2]
        OpenAI[OpenAI GPT-4o + Instructor]
        PostgreSQL[(PostgreSQL 16)]
        SQLAlchemy[SQLAlchemy 2.0]
        n8n[n8n]
        WhatsApp[WhatsApp Business Cloud API]
        Docker[Docker + Docker Compose]
        Slack[Slack/Email]
    end
    Python-->Pydantic
    Python-->OpenAI
    Python-->SQLAlchemy
    Python-->PostgreSQL
    Python-->n8n
    n8n-->WhatsApp
    n8n-->Slack
    Docker-->Python
    Docker-->n8n
    Docker-->PostgreSQL
```

---

## 🚀 Quick Start

### Prerrequisitos

- [Docker](https://docs.docker.com/get-docker/) 20.10+
- [Docker Compose](https://docs.docker.com/compose/install/) 2.0+
- API Key de OpenAI (o proveedor LLM compatible)
- Credenciales de WhatsApp Business API

### 1. Clonar el repositorio

```bash
git clone https://github.com/mmendeza-beep/ideal-journey.git
cd ideal-journey
```

### 2. Configurar variables de entorno

Edita el archivo `.env` con tus credenciales y configuración.

### 3. Levantar los servicios

```bash
docker-compose up --build
```

### 4. Acceder a los servicios

- FastAPI: http://localhost:8000
- n8n: http://localhost:5678

---

## 🛣️ Roadmap

- Integración con más canales (Telegram, Web)
- Mejoras en la observabilidad y métricas
- Políticas de decisión personalizables
- Plugins para nuevos dominios

---

## 🤝 Contribución

¡Las contribuciones son bienvenidas! Abre un issue o PR para sugerencias y mejoras.

---

## 📄 Licencia

Este proyecto está bajo la licencia MIT.

