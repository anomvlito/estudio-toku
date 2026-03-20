# Endpoints Core — Toku API

Base URL: `https://api.trytoku.com`

---

## Customers

| Método | Ruta | Descripción |
|--------|------|-------------|
| `POST` | `/customers` | Crear customer |
| `GET` | `/customers` | Listar customers |
| `GET` | `/customers/{id}` | Obtener customer |
| `PUT` | `/customers/{id}` | Editar customer |
| `DELETE` | `/customers/{id}` | Eliminar customer |
| `GET` | `/customers/external/{external_id}` | Buscar por external_id |
| `GET` | `/customers/{id}/invoices` | Invoices de un customer |
| `GET` | `/customers/{id}/subscriptions` | Subscriptions de un customer |
| `GET` | `/customers/{id}/payment_methods` | Payment methods de un customer |

```bash
# Crear customer
curl -X POST https://api.trytoku.com/customers \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "external_id": "cliente-001",
    "email": "juan@empresa.cl",
    "name": "Juan Pérez",
    "phone": "+56912345678"
  }'
```

---

## Subscriptions

| Método | Ruta | Descripción |
|--------|------|-------------|
| `POST` | `/subscriptions` | Crear subscription |
| `GET` | `/subscriptions` | Listar |
| `GET` | `/subscriptions/{id}` | Obtener |
| `PUT` | `/subscriptions/{id}` | Editar |
| `DELETE` | `/subscriptions/{id}` | Eliminar |
| `GET` | `/subscriptions/product/{product_id}` | Buscar por product_id |

```bash
# Crear subscription
curl -X POST https://api.trytoku.com/subscriptions \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_id": "cust_123",
    "product_id": "plan-mensual-premium",
    "name": "Plan Premium",
    "description": "Acceso completo al servicio"
  }'
```

---

## Invoices (Deudas)

| Método | Ruta | Descripción |
|--------|------|-------------|
| `POST` | `/invoices` | Crear invoice |
| `GET` | `/invoices` | Listar |
| `GET` | `/invoices/{id}` | Obtener |
| `GET` | `/invoices/external/{external_id}` | Buscar por external_id |
| `PUT` | `/invoices/{id}` | Editar |
| `POST` | `/invoices/{id}/void` | Anular invoice |
| `DELETE` | `/invoices/{id}` | Eliminar |

```bash
# Crear invoice (deuda)
curl -X POST https://api.trytoku.com/invoices \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_id": "cust_123",
    "subscription_id": "sub_456",
    "amount": 50000,
    "currency": "CLP",
    "due_date": "2024-12-31",
    "description": "Cuota diciembre 2024",
    "external_id": "ACC-2024-12-001"
  }'
```

---

## Payment Methods

| Método | Ruta | Descripción |
|--------|------|-------------|
| `GET` | `/payment_methods` | Listar todos |
| `GET` | `/payment_methods/{id}` | Obtener uno |
| `DELETE` | `/payment_methods/{id}` | Eliminar |
| `POST` | `/payment_methods/{id}/subscriptions` | Asociar a subscription |
| `POST` | `/payment_methods/bulk_associate` | Asociar múltiples (bulk) |

---

## Transactions (Cobros)

| Método | Ruta | Descripción |
|--------|------|-------------|
| `GET` | `/transactions` | Listar todas |
| `GET` | `/transactions/{id}` | Obtener una |
| `GET` | `/transactions/{id}/fee` | Detalle del fee cobrado |
| `POST` | `/charge` | Cobro individual |
| `POST` | `/charge/bulk` | Cobro masivo (varios customers) |
| `POST` | `/transactions/{id}/refund` | Devolver transacción |
| `POST` | `/invoices/{id}/refund` | Devolver por invoice |

```bash
# Ejecutar cobro
curl -X POST https://api.trytoku.com/charge \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "invoice_id": "inv_001",
    "payment_method_id": "pm_789"
  }'
```

---

## Entities

| Método | Ruta | Descripción |
|--------|------|-------------|
| `POST` | `/entities` | Crear entidad |

---

## Payment Object

| Método | Ruta | Descripción |
|--------|------|-------------|
| `GET` | `/payments` | Listar payments |
| `GET` | `/payments/{id}` | Obtener payment |
| `POST` | `/payments` | Crear o modificar |
| `PUT` | `/payments/{id}` | Modificar |
| `DELETE` | `/payments` | Eliminar |

---

## Webhooks (Configuración)

| Método | Ruta | Descripción |
|--------|------|-------------|
| `POST` | `/webhooks` | Crear webhook endpoint |
| `GET` | `/webhooks` | Listar webhooks |
| `GET` | `/webhooks/{id}` | Obtener webhook |
| `PUT` | `/webhooks/{id}` | Editar webhook |
| `POST` | `/webhooks/test` | Disparar prueba |

---

## Misc

```
POST /payment_portal/session   → Crear sesión de Payment Portal
GET  /events                   → Listar eventos
```

---

## Códigos de Respuesta HTTP

| Código | Significado |
|--------|-------------|
| `200` | OK |
| `201` | Creado exitosamente |
| `400` | Bad Request — datos inválidos |
| `401` | Unauthorized — API key incorrecta |
| `404` | Not Found — recurso no existe |
| `409` | Conflict — recurso duplicado |
| `422` | Unprocessable Entity — validación fallida |
| `429` | Too Many Requests — rate limit |
| `500` | Error interno Toku |
