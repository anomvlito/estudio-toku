# Plan de Estudio — Entrevista Toku 2026

## Orden recomendado de estudio

| # | Módulo | Prioridad | Por qué |
|---|--------|-----------|---------|
| 1 | **Dominio: Pagos Recurrentes** | ALTA | Sin entender el negocio, todo lo demás carece de contexto |
| 2 | **PostgreSQL Esencial** | ALTA | Lo usas todo el tiempo, tu punto débil más impactante |
| 3 | **Webhooks e Integraciones** | ALTA | El rol de Integrations Engineer lo exige explícitamente |
| 4 | **TypeScript para Pythonistas** | MEDIA | Sabes Python, TS es un puente, lo aprenderás rápido |
| 5 | **FastAPI en Profundidad** | MEDIA | Ya sabes Python; esto es refinamiento de patrones |
| 6 | **Simulacros de Entrevista** | ALTA (final) | Haz esto justo antes de la entrevista |

## Cómo compilar los PDFs

```bash
# Necesitas pdflatex instalado
sudo apt install texlive-full   # Ubuntu/WSL

# Compilar todo
cd /home/fabian/src/documentos/toku/estudio
make all

# Limpiar auxiliares
make clean

# Compilar uno solo
cd 01_typescript_para_pythonistas
pdflatex typescript.tex
```

## Estructura de carpetas

```
estudio/
├── 01_typescript_para_pythonistas/
│   └── typescript.tex        → Para pythonistas: TS, arrow fn, async, Vue.js
├── 02_postgresql_esencial/
│   └── postgresql.tex        → Modelo de datos Toku, transacciones, índices
├── 03_dominio_pagos_recurrentes/
│   └── dominio_toku.tex      → PAC/SPEI/PIX, flujos de cobro, idempotencia
├── 04_webhooks_e_integraciones/
│   └── webhooks.tex          → HMAC, OAuth 2.0, reintentos, sync con ERP
├── 05_fastapi_avanzado/
│   └── fastapi.tex           → Pydantic v2, DI, manejo errores, testing
└── 06_simulacros_entrevista/
    └── simulacros.tex        → Preguntas reales, coding challenges, SQL en vivo
```

## Tu perfil vs lo que Toku necesita

| Habilidad | Tu nivel | Nivel requerido | Brecha |
|-----------|----------|-----------------|--------|
| Python | ✅ Bueno | ✅ Esencial | Ninguna |
| FastAPI | ✅ Básico | 🔶 Intermedio | Pequeña |
| TypeScript | ⚠️ Débil | 🔶 Intermedio | Media |
| Vue.js | ⚠️ Débil | 🔶 Básico-Intermedio | Media |
| PostgreSQL | ⚠️ Básico | 🔶 Intermedio | Media |
| GCP | ℹ️ Sabes Vercel | 🔶 Básico GCP | Pequeña |
| Webhooks/Integraciones | ℹ️ Conoces Supabase | 🔶 Intermedio | Media |
| Dominio pagos | ❌ Desconocido | ✅ Esencial | Alta |

## Recursos adicionales

- API de Toku scrapeada: `../api/`
- Análisis de ofertas históricas: `../ANALISIS_TOKU_ENTREVISTA.md`
- Sitio de empleos: https://jobs.trytoku.com
