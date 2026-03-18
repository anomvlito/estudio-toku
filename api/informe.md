# Informe: API de Toku

**Preparado para**: Entrevista de trabajo en Toku  
**Fecha**: Marzo 2026  
**Fuente**: [toku.readme.io](https://toku.readme.io) — Documentación oficial

---

## 1. ¿Qué es Toku?

**Toku** es un **SaaS de automatización de cobranza recurrente** para Latinoamérica (Chile, México, Brasil). Permite a empresas automatizar el cobro mensual a sus clientes, reducir morosidad y eliminar procesos manuales de conciliación.

### Los 4 pilares de valor

| Pilar | Descripción |
|-------|-------------|
| **Aumentar la recaudación** | Reintentos automáticos, múltiples medios de pago |
| **Disminuir los costos** | Elimina planillas manuales, conciliación automática |
| **Mejor experiencia** | Canales omnicanal, notificaciones automáticas |
| **Seguridad** | Cumplimiento PCI, HTTPS, firma HMAC en webhooks |

### Industrias objetivo

Seguros, Créditos, Educación, Inmobiliarias, Salud, Utilities, Membresías — cualquier empresa con **cobros recurrentes**.

---

## 2. Modelo de Datos

Las 4 entidades clave del sistema:

```
Organization (empresa que contrata Toku)
  └── Customer (el pagador)
       ├── Subscription / Producto  → contenedor lógico de deudas (product_id)
       │    └── Invoice / Deuda     → cobro individual puntual
       │         ├── Payment Intent → intento de cobro ligado a 1 invoice
       │         └── Transaction    → intento que puede agrupar varias invoices
       └── Payment Method
            ├── PAC / PAT (Chile)
            ├── VCC / CLABE (México)
            ├── Card (Internacional)
            ├── Bank Account
            ├── Pix (Brasil)
            └── Boleto (Brasil)
```

### Definiciones clave

| Término | Definición |
|---------|------------|
| **Organization** | La empresa que usa Toku (el cliente de Toku) |
| **Customer** | El pagador final de la organización |
| **Subscription** | Contenedor lógico de deudas — **NO** es membresía. Se identifica con `product_id` |
| **Invoice** | Deuda individual. Se identifica por `external_id` o `product_id + fecha_vencimiento` |
| **Payment Intent** | Intento de cobro ligado a una sola invoice |
| **Transaction** | Agrupador de intentos de pago (puede cubrir múltiples invoices) |
| **Settlement** | Liquidación: cuando Toku deposita los fondos recaudados a la organización |
| **PAC** | Pago Automático de Cuenta — débito bancario automático (Chile) |
| **PAT** | Pago Automático Tarjeta — cobro automático a tarjeta (Chile) |
| **VCC / CLABE** | Validación de Cuenta Bancaria (México) |

---

## 3. API REST

**Base URL**: `https://api.trytoku.com`  
**Autenticación**: `Authorization: Bearer YOUR_API_KEY`  
**Content-Type**: `application/json`

### Endpoints principales

#### Clientes
```
POST   /customers          → Crear cliente
GET    /customers          → Listar clientes
GET    /customers/{id}     → Obtener detalles
PUT    /customers/{id}     → Actualizar cliente
DEL    /customers/{id}     → Eliminar cliente
GET    /customers/{id}/invoices  → Deudas de un cliente
```

#### Suscripciones
```
POST   /subscriptions          → Crear suscripción
GET    /subscriptions          → Listar
GET    /subscriptions/{id}     → Obtener
PUT    /subscriptions/{id}     → Actualizar
DEL    /subscriptions/{id}     → Eliminar
```

#### Deudas (Invoices)
```
POST   /invoices               → Crear deuda
GET    /invoices               → Listar
GET    /invoices/{id}          → Obtener
PUT    /invoices/{id}          → Editar
DEL    /invoices/{id}          → Eliminar
POST   /invoices/{id}/void     → Anular
```

#### Transacciones & Cobros
```
GET    /transactions           → Listar transacciones
GET    /transactions/{id}      → Obtener
POST   /transactions           → Cobro
POST   /transactions/bulk      → Cobro masivo
```

#### Medios de Pago
```
GET    /payment_methods              → Listar
GET    /payment_methods/{id}         → Obtener
DEL    /payment_methods/{id}         → Eliminar
POST   /payment_methods/associate    → Asociar a suscripción
```

#### Webhooks
```
POST   /webhooks              → Crear endpoint
GET    /webhooks              → Listar
GET    /webhooks/{id}         → Obtener
PUT    /webhooks/{id}         → Actualizar
DEL    /webhooks/{id}         → Eliminar
POST   /webhooks/test         → Probar webhook
```

### Endpoints por región

**Chile**: Checkout Sessions, Customer Collection Account (PAC/PAT), Cards, Redirections  
**México**: Cards, Bank Accounts (CLABE), Cash (referencias), Facturación (PPD/PUE), Nota de Crédito  
**Brasil**: Pix Automático, Boleto, Débito Automático, Configuraciones

---

## 4. Formas de Integración Backend

| Método | Tiempo | Complejidad | Cuándo usarlo |
|--------|--------|-------------|---------------|
| **API REST** | 1-2 semanas | Alta | Integración tiempo real con sistemas propios |
| **SFTP** (CSV/JSON) | 3-5 días | Media | Cargas batch diarias/semanales, sistemas legacy |
| **Formulario manual** | Inmediato | Baja | Pruebas, volumen ocasional, entrada puntual |

### Flujo típico API

1. Crear `Customer` → `POST /customers`
2. Crear `Subscription` → `POST /subscriptions`
3. Crear `Invoice` → `POST /invoices`
4. Cliente paga (via frontend channel)
5. Toku notifica resultado → Webhook `invoice.paid` o `invoice.failed`

### Formato SFTP

```
Host: sftp.toku.com | Puerto: 22 | Ruta: /incoming/
Formato archivos: customers_YYYYMMDD.csv
```

---

## 5. Canales Frontend (Cómo el cliente paga)

| Canal | Conversión | Setup | Control UI | Cuándo usar |
|-------|-----------|-------|------------|-------------|
| **Web Link** | ~80% | < 1 día | Ninguno | Email, SMS, notificaciones |
| **Iframe** | ~95% | 2-3 días | Total | Sitio web con branding propio |
| **QR Code** | ~70% | < 1 día | Ninguno | Boletas físicas, papelería |
| **Mensajería (WA/SMS)** | ~90-98% | 2-3 días | Mínimo | Contacto directo con el cliente |
| **Mobile WebView** | ~96% | 3-5 días | Total | App iOS/Android nativa |

### Implementación Iframe (más común)

```html
<div id="toku-payment-container"></div>
<script src="https://sdk.toku.com/js/checkout.js"></script>
<script>
  TokuCheckout.createCheckout({
    sessionId: 'sess_123456',
    container: '#toku-payment-container',
    theme: { primaryColor: '#007bff' },
    onSuccess: (result) => { /* pago exitoso */ },
    onError: (error) => { /* manejar error */ }
  });
</script>
```

### URL de checkout

```
https://pay.toku.com/checkout/{SESSION_ID}
```

- URL única por sesión
- Validez configurable (15 min – 7 días)
- Soporte múltiples medios de pago y multiidioma

---

## 6. Sistema de Webhooks

Los webhooks permiten que Toku notifique a tu servidor en **tiempo real** cuando ocurren eventos, eliminando la necesidad de polling.

### Eventos disponibles (15+)

| Categoría | Eventos |
|-----------|---------|
| **Customer** | `customer.created`, `customer.updated` |
| **Subscription** | `subscription.created`, `subscription.updated` |
| **Invoice** | `invoice.created`, `invoice.paid`, `invoice.partially_paid`, `invoice.failed` |
| **Payment Method** | `payment_method.created`, `payment_method.updated`, `payment_method.deleted` |
| **Transaction** | `transaction.completed`, `transaction.failed`, `transaction.pending` |
| **Settlement** | `settlement.created` |

### Estructura de un evento

```json
{
  "id": "evt_1234567895",
  "type": "invoice.paid",
  "data": {
    "invoice_id": "inv_001",
    "customer_id": "cust_123",
    "amount": 50000,
    "currency": "CLP",
    "paid_at": "2024-12-25T14:30:00Z",
    "transaction_id": "txn_123"
  },
  "timestamp": "2024-12-25T14:30:00Z"
}
```

### Configurar un webhook

```bash
# Registrar endpoint en Toku
curl -X POST https://api.trytoku.com/webhooks \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://tu-dominio.com/webhooks/toku",
    "events": ["invoice.paid", "invoice.failed", "transaction.completed"],
    "active": true,
    "secret": "tu_secreto_webhook"
  }'
```

### Validar firma (seguridad)

Toku envía header `X-Toku-Signature` (HMAC-SHA256):

```python
import hmac, hashlib

def verify_webhook(request, secret):
    signature = request.headers.get('X-Toku-Signature')
    body = request.get_data()  # ← IMPORTANTE: raw body, no JSON parseado
    expected = hmac.new(secret.encode(), body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(signature, expected)
```

### Reintentos automáticos

| Intento | Retardo |
|---------|---------|
| 1 | Inmediato |
| 2 | 5 minutos |
| 3 | 30 minutos |
| 4 | 2 horas |
| 5 | 5 horas |

Máximo 5 intentos en 24 horas. Tu endpoint debe retornar `200 OK`.

---

## 7. Buenas Prácticas

### Integración

| Aspecto | Recomendación |
|---------|---------------|
| **Calidad de datos** | Validar antes de enviar; sin duplicados, sin incompletos |
| **Sincronización** | Preferir webhooks sobre polling o batch |
| **Testing** | Usar `/webhooks/test` y ngrok para pruebas locales |
| **IDs** | Usar `external_id` únicos y trazables en tu sistema |
| **Medios de pago** | Habilitar todos los disponibles en tu región |

### Webhooks

- ✅ **Validar firma siempre** — rechazar con 401 si falla
- ✅ **Procesar asíncronamente** — retornar 200 inmediato, usar queue (Celery, BullMQ, etc.)
- ✅ **Idempotencia** — guardar `event_id` para evitar procesar duplicados
- ✅ **Solo HTTPS** — nunca HTTP
- ✅ **Logging** — registrar todos los eventos recibidos
- ✅ **Alertas** — especialmente para `invoice.failed` y `transaction.failed`

---

## 8. Seguridad

- Usar **API Keys** con rotación periódica
- Nunca hardcodear keys en código fuente → usar variables de entorno
- Implementar RBAC (Role-Based Access Control)
- Validar firma HMAC en cada webhook recibido
- Whitelisting de IPs si aplica
- Usar certificado SSL válido en tu servidor de webhooks
- Para tarjetas: solo organizaciones **PCI Compliant** pueden manejar datos de tarjeta directamente

---

## 9. Testing

```bash
# Probar un webhook específico
curl -X POST https://api.trytoku.com/webhooks/test \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"event_type": "invoice.paid", "webhook_id": "hook_123"}'
```

**Herramientas útiles:**
- `ngrok http 5000` → tunnel local para recibir webhooks en desarrollo
- [webhook.cool](https://webhook.cool) → inspeccionar requests en tiempo real
- Dashboard de Toku → monitoreo de éxito/fallo de webhooks

---

## 10. Recursos

| Recurso | URL |
|---------|-----|
| Sitio oficial | https://trytoku.com |
| Documentación | https://toku.readme.io |
| API Base URL | https://api.trytoku.com |
| API Reference | https://toku.readme.io/reference |
| Getting Started | https://toku.readme.io/docs/getting-started |

### Documentación local

- [`api/api-reference.md`](./api/api-reference.md) — Referencia completa de endpoints (60+)
- [`api/backend-integration.md`](./api/backend-integration.md) — Guía de integración backend
- [`api/webhooks.md`](./api/webhooks.md) — Sistema de webhooks y eventos
- [`api/frontend-channels.md`](./api/frontend-channels.md) — Canales de pago frontend
- [`buenas-practicas.md`](./buenas-practicas.md) — Mejores prácticas
- [`glosario.md`](./glosario.md) — Glosario de términos
- [`casos-de-uso.md`](./casos-de-uso.md) — Casos de uso por industria
- [`getting-started.md`](./getting-started.md) — Guía de inicio

---

*Documentación compilada de [toku.readme.io](https://toku.readme.io) — Última actualización: Marzo 2026*
