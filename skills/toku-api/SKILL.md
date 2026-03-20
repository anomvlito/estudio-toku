---
name: toku-api
description: "Experto completo en la API de Toku: plataforma de cobros recurrentes para Chile, México y Brasil. Orquesta conocimiento de endpoints, webhooks, canales de pago, integración backend y mejores prácticas."
user-invocable: true
---

# Toku API — Skill Orquestadora

Toku es una plataforma de cobros recurrentes y gestión de pagos para empresas.
**Base URL**: `https://api.trytoku.com`
**Docs oficiales**: https://toku.readme.io

---

## Cuándo usar este skill

- Diseñar o implementar una integración con Toku desde cero
- Consultar endpoints específicos, parámetros o estructuras de respuesta
- Configurar y manejar webhooks de eventos de pago
- Elegir el canal frontend correcto para mostrar pagos a clientes
- Resolver errores de integración o comportamientos inesperados
- Prepararse para una entrevista técnica sobre la plataforma Toku
- Entender el modelo de datos: customers, subscriptions, invoices, transactions

## Cuándo NO usar este skill

- Preguntas de negocio sobre precios o contratos Toku → hablar con ventas
- Problemas de acceso al dashboard → soporte Toku
- Implementación en frameworks específicos sin relación a la API (ej: "cómo hacer un form en React") → otro skill

---

## Routing — Qué recurso usar según el problema

| Situación | Ir a |
|-----------|------|
| Entender entidades, glosario, modelo de datos | `resources/architecture.md` |
| Listar, crear o modificar customers/invoices/subscriptions/payments | `resources/endpoints-core.md` |
| Endpoints específicos de Chile, México o Brasil | `resources/endpoints-countries.md` |
| Recibir notificaciones de pagos en tiempo real | `resources/webhooks.md` |
| Mostrar checkout al cliente (web, app, WhatsApp) | `resources/frontend.md` |
| Decidir cómo integrar el backend (API vs SFTP vs manual) | `resources/integration.md` |
| Depurar errores, firmas inválidas, webhooks que no llegan | `resources/troubleshooting.md` |

---

## Instrucciones para Claude

1. **Identificar el contexto**: ¿El usuario quiere integrar, consultar un endpoint, manejar eventos, o mostrar una UI de pago?
2. **Leer el recurso correspondiente** según la tabla de routing.
3. **Combinar recursos** cuando la pregunta cruza dominios (ej: "cómo cobrar a un cliente chileno con webhook" → `endpoints-core.md` + `endpoints-countries.md` + `webhooks.md`).
4. **Siempre incluir ejemplos de código** con curl o Python cuando se habla de endpoints.
5. **Mencionar consideraciones por país** cuando el contexto lo requiera (Chile/México/Brasil tienen endpoints diferentes).
6. **Priorizar buenas prácticas** de la sección `integration.md`: idempotencia, procesamiento async, deduplicación de eventos.

---

## Recursos

- `resources/architecture.md` — modelo de entidades, glosario, relaciones
- `resources/endpoints-core.md` — Customers, Subscriptions, Invoices, Payment Methods, Transactions, Settlements, Webhooks config
- `resources/endpoints-countries.md` — Chile, México, Brasil: endpoints regionales
- `resources/webhooks.md` — eventos disponibles, estructuras JSON, handlers, reintentos
- `resources/frontend.md` — Web Link, Iframe, QR, Mensajería, WebView
- `resources/integration.md` — métodos de integración, flujos típicos, buenas prácticas
- `resources/troubleshooting.md` — errores comunes y soluciones
