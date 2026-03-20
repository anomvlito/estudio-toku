# Webhooks — Sistema de Eventos Toku

Los webhooks son el mecanismo principal para sincronizar Toku con tu sistema en tiempo real.
Toku hace POST a tu endpoint cuando ocurre un evento. Tu servidor debe responder **HTTP 200**.

---

## Todos los Eventos Disponibles

| Evento | Cuándo se dispara |
|--------|------------------|
| `customer.created` | Nuevo customer creado |
| `customer.updated` | Customer modificado |
| `subscription.created` | Nueva subscription creada |
| `subscription.updated` | Subscription modificada |
| `invoice.created` | Nueva deuda creada |
| `invoice.paid` | Deuda pagada completamente |
| `invoice.partially_paid` | Pago parcial registrado |
| `invoice.failed` | Cobro de deuda falló |
| `payment_method.created` | Nuevo medio de pago inscrito |
| `payment_method.updated` | Medio de pago modificado |
| `payment_method.deleted` | Medio de pago eliminado |
| `transaction.completed` | Transacción exitosa |
| `transaction.failed` | Transacción fallida |
| `transaction.pending` | Transacción en espera del banco |
| `settlement.created` | Nueva liquidación generada |

---

## Estructura General de Evento

```json
{
  "id": "evt_1234567890",
  "type": "invoice.paid",
  "data": { ... },
  "timestamp": "2024-12-25T14:30:00Z"
}
```

### Ejemplos por Evento

**invoice.paid**
```json
{
  "id": "evt_abc",
  "type": "invoice.paid",
  "data": {
    "invoice_id": "inv_001",
    "customer_id": "cust_123",
    "amount": 50000,
    "currency": "CLP",
    "paid_at": "2024-12-25T14:30:00Z",
    "payment_method_id": "pm_789",
    "transaction_id": "txn_123"
  }
}
```

**invoice.failed**
```json
{
  "type": "invoice.failed",
  "data": {
    "invoice_id": "inv_001",
    "customer_id": "cust_123",
    "amount": 50000,
    "failure_reason": "Fondos insuficientes",
    "failure_code": "insufficient_funds",
    "failed_at": "2024-12-26T09:15:00Z",
    "retry_date": "2024-12-27T09:00:00Z"
  }
}
```

**settlement.created**
```json
{
  "type": "settlement.created",
  "data": {
    "settlement_id": "sett_001",
    "settlement_date": "2024-12-27",
    "total_amount": 450000,
    "currency": "CLP",
    "transaction_count": 9,
    "fees": 4500,
    "net_amount": 445500
  }
}
```

---

## Registrar un Webhook

```bash
curl -X POST https://api.trytoku.com/webhooks \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://tu-dominio.com/webhooks/toku",
    "events": [
      "invoice.created",
      "invoice.paid",
      "invoice.failed",
      "transaction.completed",
      "settlement.created"
    ],
    "active": true,
    "secret": "tu_secreto_webhook"
  }'
```

---

## Validar Firma (Seguridad)

Cada webhook incluye el header `X-Toku-Signature` con HMAC-SHA256.

```python
import hmac
import hashlib

def verify_webhook(request, secret: str) -> bool:
    signature = request.headers.get('X-Toku-Signature')
    body = request.get_data()  # RAW bytes — nunca usar json.dumps(request.json)
    expected = hmac.new(
        secret.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(signature, expected)
```

---

## Handler Completo (Flask + Celery)

```python
from flask import Flask, request
from celery import shared_task

app = Flask(__name__)
WEBHOOK_SECRET = 'tu_secreto_webhook'

@app.route('/webhooks/toku', methods=['POST'])
def handle_webhook():
    # 1. Validar firma
    if not verify_webhook(request, WEBHOOK_SECRET):
        return {'error': 'Invalid signature'}, 401

    event = request.json
    event_id = event['id']

    # 2. Deduplicar (idempotencia)
    if WebhookEvent.objects.filter(event_id=event_id).exists():
        return {'status': 'already_processed'}, 200

    # 3. Persistir y procesar en background
    WebhookEvent.objects.create(event_id=event_id, type=event['type'])
    procesar_evento.delay(event)

    return {'status': 'received'}, 200  # Retornar 200 inmediatamente


@shared_task
def procesar_evento(event):
    handlers = {
        'invoice.paid':           actualizar_pago_en_sistema,
        'invoice.failed':         notificar_fallo_y_reintentar,
        'transaction.completed':  registrar_transaccion,
        'payment_method.created': activar_cobro_automatico,
        'settlement.created':     registrar_liquidacion,
    }
    handler = handlers.get(event['type'])
    if handler:
        handler(event['data'])
```

---

## Política de Reintentos

Si tu endpoint no responde 200, Toku reintenta automáticamente:

| Intento | Retardo acumulado |
|---------|-------------------|
| 1 | Inmediato |
| 2 | +5 minutos |
| 3 | +30 minutos |
| 4 | +2 horas |
| 5 | +5 horas |

**Máximo**: 5 intentos en 24 horas. Después se marca como fallido.

---

## Testing de Webhooks

```bash
# Disparar evento de prueba desde Toku
curl -X POST https://api.trytoku.com/webhooks/test \
  -H "Authorization: Bearer $API_KEY" \
  -d '{"event_type": "invoice.paid", "webhook_id": "hook_123"}'
```

**Herramientas para desarrollo local:**
- `ngrok http 5000` → túnel HTTPS a tu local
- https://webhook.cool → captura y visualiza requests
