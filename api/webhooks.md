# Webhooks | Notificaciones en Tiempo Real

Guía completa del sistema de webhooks de Toku para recibir notificaciones de eventos en tiempo real.

---

## 📌 ¿Qué son los Webhooks?

Los **webhooks** son notificaciones automáticas que Toku envía a tu servidor cuando ocurren eventos importantes. Funcionan como un "sistema de llamadas" donde Toku te avisa inmediatamente cuando algo ocurre.

### Analogía
```
Sin Webhooks:
Tu app ----> "¿Pasó algo?" ----> Toku server ----> "No"
Tu app ----> "¿Pasó algo?" ----> Toku server ----> "No"
Tu app ----> "¿Pasó algo?" ----> Toku server ----> "¡SÍ! Pago completado"

Con Webhooks:
Toku server ----> "¡Evento!" ----> Tu app (automático)
Tu app recibe y procesa inmediatamente
```

---

## 🔔 Eventos Disponibles

### Cliente (Customer)

#### `customer.created`
Se dispara cuando se **crea un nuevo cliente**.

```json
{
  "id": "evt_1234567890",
  "type": "customer.created",
  "data": {
    "customer_id": "cust_123",
    "email": "cliente@example.com",
    "name": "Juan Pérez",
    "phone": "+56912345678",
    "created_at": "2024-12-20T10:00:00Z"
  },
  "timestamp": "2024-12-20T10:00:00Z"
}
```

**Campos importantes:**
- `customer_id`: ID único del cliente
- `email`: Email del cliente
- `name`: Nombre completo
- `phone`: Teléfono de contacto

---

#### `customer.updated`
Se dispara cuando se **actualiza un cliente existente**.

```json
{
  "id": "evt_1234567891",
  "type": "customer.updated",
  "data": {
    "customer_id": "cust_123",
    "changes": {
      "email": {
        "old": "anterior@example.com",
        "new": "nuevo@example.com"
      },
      "phone": {
        "old": "+56912345678",
        "new": "+56987654321"
      }
    },
    "updated_at": "2024-12-20T10:15:00Z"
  },
  "timestamp": "2024-12-20T10:15:00Z"
}
```

---

### Suscripción (Subscription)

#### `subscription.created`
Se dispara cuando se **crea una nueva suscripción** (producto/plan).

```json
{
  "id": "evt_1234567892",
  "type": "subscription.created",
  "data": {
    "subscription_id": "sub_456",
    "product_id": "prod_789",
    "customer_id": "cust_123",
    "name": "Plan Premium",
    "description": "Acceso a todos los servicios",
    "created_at": "2024-12-20T10:30:00Z"
  },
  "timestamp": "2024-12-20T10:30:00Z"
}
```

---

#### `subscription.updated`
Se dispara cuando se **actualiza una suscripción**.

```json
{
  "id": "evt_1234567893",
  "type": "subscription.updated",
  "data": {
    "subscription_id": "sub_456",
    "changes": {
      "status": {
        "old": "active",
        "new": "suspended"
      }
    }
  }
}
```

---

### Factura/Deuda (Invoice)

#### `invoice.created`
Se dispara cuando se **crea una nueva factura/deuda**.

```json
{
  "id": "evt_1234567894",
  "type": "invoice.created",
  "data": {
    "invoice_id": "inv_001",
    "customer_id": "cust_123",
    "subscription_id": "sub_456",
    "amount": 50000,
    "currency": "CLP",
    "due_date": "2024-12-31T23:59:59Z",
    "description": "Pago de servicio diciembre",
    "external_reference": "ACC-2024-12-001",
    "created_at": "2024-12-20T11:00:00Z"
  },
  "timestamp": "2024-12-20T11:00:00Z"
}
```

---

#### `invoice.paid`
Se dispara cuando una **factura es pagada completamente**.

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
    "payment_method_id": "pm_789",
    "transaction_id": "txn_123",
    "reference": "INV-001-PAY-20241225"
  },
  "timestamp": "2024-12-25T14:30:00Z"
}
```

---

#### `invoice.partially_paid`
Se dispara cuando se **realiza un pago parcial** a una factura.

```json
{
  "id": "evt_1234567896",
  "type": "invoice.partially_paid",
  "data": {
    "invoice_id": "inv_001",
    "customer_id": "cust_123",
    "total_amount": 50000,
    "paid_amount": 25000,
    "remaining_amount": 25000,
    "paid_at": "2024-12-22T10:00:00Z"
  }
}
```

---

#### `invoice.failed`
Se dispara cuando **falla el cobro** de una factura.

```json
{
  "id": "evt_1234567897",
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

---

### Medio de Pago (Payment Method)

#### `payment_method.created`
Se dispara cuando se **agrega un nuevo medio de pago** a un cliente.

```json
{
  "id": "evt_1234567898",
  "type": "payment_method.created",
  "data": {
    "payment_method_id": "pm_789",
    "customer_id": "cust_123",
    "type": "bank_account",
    "bank_code": "001",
    "account_number": "****1234",
    "holder_name": "Juan Pérez",
    "created_at": "2024-12-20T12:00:00Z"
  }
}
```

---

#### `payment_method.updated`
Se dispara cuando se **actualiza un medio de pago**.

```json
{
  "id": "evt_1234567899",
  "type": "payment_method.updated",
  "data": {
    "payment_method_id": "pm_789",
    "customer_id": "cust_123",
    "changes": {
      "status": {
        "old": "active",
        "new": "inactive"
      }
    }
  }
}
```

---

#### `payment_method.deleted`
Se dispara cuando se **elimina un medio de pago**.

```json
{
  "id": "evt_1234567900",
  "type": "payment_method.deleted",
  "data": {
    "payment_method_id": "pm_789",
    "customer_id": "cust_123",
    "deleted_at": "2024-12-20T13:00:00Z"
  }
}
```

---

### Transacción (Transaction)

#### `transaction.completed`
Se dispara cuando una **transacción se completa exitosamente**.

```json
{
  "id": "evt_1234567901",
  "type": "transaction.completed",
  "data": {
    "transaction_id": "txn_123",
    "customer_id": "cust_123",
    "amount": 50000,
    "currency": "CLP",
    "status": "completed",
    "payment_method_id": "pm_789",
    "invoice_ids": ["inv_001"],
    "completed_at": "2024-12-25T14:30:00Z",
    "reference": "TXN-2024-12-25-001"
  }
}
```

---

#### `transaction.failed`
Se dispara cuando una **transacción falla**.

```json
{
  "id": "evt_1234567902",
  "type": "transaction.failed",
  "data": {
    "transaction_id": "txn_123",
    "customer_id": "cust_123",
    "amount": 50000,
    "currency": "CLP",
    "status": "failed",
    "failure_reason": "Tarjeta rechazada",
    "failure_code": "card_declined",
    "failed_at": "2024-12-25T14:35:00Z"
  }
}
```

---

#### `transaction.pending`
Se dispara cuando una **transacción queda en estado pendiente**.

```json
{
  "id": "evt_1234567903",
  "type": "transaction.pending",
  "data": {
    "transaction_id": "txn_123",
    "customer_id": "cust_123",
    "amount": 50000,
    "status": "pending",
    "pending_reason": "Esperando confirmación del banco"
  }
}
```

---

### Liquidación (Settlement)

#### `settlement.created`
Se dispara cuando se **genera una nueva liquidación**.

```json
{
  "id": "evt_1234567904",
  "type": "settlement.created",
  "data": {
    "settlement_id": "sett_001",
    "settlement_date": "2024-12-27",
    "total_amount": 450000,
    "currency": "CLP",
    "transaction_count": 9,
    "fees": 4500,
    "net_amount": 445500,
    "created_at": "2024-12-27T09:00:00Z"
  }
}
```

---

## 🔧 Configuración de Webhooks

### Paso 1: Crear Endpoint en tu Servidor

```python
# Flask Example
from flask import Flask, request, jsonify
import hmac
import hashlib

app = Flask(__name__)
WEBHOOK_SECRET = 'tu_secreto_webhook'

@app.route('/webhooks/toku', methods=['POST'])
def handle_webhook():
    # Validar firma
    signature = request.headers.get('X-Toku-Signature')
    body = request.get_data()
    
    expected_sig = hmac.new(
        WEBHOOK_SECRET.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    
    if not hmac.compare_digest(signature, expected_sig):
        return {'error': 'Invalid signature'}, 401
    
    # Procesar evento
    event = request.json
    event_type = event['type']
    
    if event_type == 'invoice.paid':
        handle_invoice_paid(event['data'])
    elif event_type == 'transaction.completed':
        handle_transaction_completed(event['data'])
    
    # Retornar 200 OK
    return {'status': 'received'}, 200

def handle_invoice_paid(data):
    print(f"Factura {data['invoice_id']} pagada")
    # Actualizar tu base de datos, enviar confirmación, etc.

def handle_transaction_completed(data):
    print(f"Transacción {data['transaction_id']} completada")
    # Procesar pago, registrar en tu sistema, etc.

if __name__ == '__main__':
    app.run(port=5000)
```

---

### Paso 2: Registrar Webhook en Toku

```bash
curl -X POST https://api.trytoku.com/webhooks \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://tu-dominio.com/webhooks/toku",
    "events": [
      "invoice.created",
      "invoice.paid",
      "invoice.failed",
      "transaction.completed",
      "transaction.failed",
      "settlement.created"
    ],
    "active": true,
    "secret": "tu_secreto_webhook"
  }'
```

---

### Paso 3: Validar Firma del Webhook

Todos los webhooks incluyen una firma `X-Toku-Signature` para validar autenticidad:

```python
import hmac
import hashlib

def verify_webhook(request, secret):
    signature = request.headers.get('X-Toku-Signature')
    body = request.get_data()
    
    expected = hmac.new(
        secret.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(signature, expected)
```

---

## 🔄 Manejo de Reintentos

Toku reintentos automáticos si tu endpoint retorna error:

| Intento | Retardo |
|---------|---------|
| 1       | Inmediato |
| 2       | 5 minutos |
| 3       | 30 minutos |
| 4       | 2 horas |
| 5       | 5 horas |

**Máximo**: 5 intentos en 24 horas

---

## ✅ Best Practices Webhooks

### 1. Validar Firma Siempre
```python
if not verify_webhook(request, webhook_secret):
    return 401  # Rechazar si no es válido
```

### 2. Procesar de Forma Asíncrona
```python
# ❌ NO hacer esto
def handle_webhook():
    resultado = procesar_base_datos_lenta()  # 10 segundos
    return 200

# ✅ Hacer esto
from celery import shared_task

@shared_task
def procesar_evento(data):
    # Procesamiento en background
    pass

def handle_webhook():
    procesar_evento.delay(request.json)
    return 200  # Retorna inmediatamente
```

### 3. Guardar ID del Evento para Evitar Duplicados
```python
def handle_webhook():
    event_id = request.json['id']
    
    # Verificar si ya fue procesado
    if Evento.objects.filter(event_id=event_id).exists():
        return 200  # Ya procesado
    
    # Guardar y procesar
    Evento.objects.create(
        event_id=event_id,
        type=request.json['type'],
        data=request.json['data']
    )
    return 200
```

### 4. Usar HTTPS Obligatoriamente
```
❌ http://mi-servidor.com/webhooks/toku
✅ https://mi-servidor.com/webhooks/toku
```

### 5. Registrar Todos los Eventos
```python
def handle_webhook():
    logger.info(f"Webhook recibido: {event_type}", extra={
        'event_id': event['id'],
        'timestamp': event['timestamp'],
        'customer_id': event['data'].get('customer_id')
    })
```

### 6. Alertas para Fallos Críticos
```python
if event_type == 'invoice.failed':
    # Alertar al equipo
    send_alert(f"Fallo de pago: {data['invoice_id']}")
```

---

## 🧪 Testing de Webhooks

### Herramientas Disponibles

#### 1. ngrok (Local Development)
```bash
ngrok http 5000
# Obtendrás: https://randomstring.ngrok.io

# Registrar como webhook:
# https://randomstring.ngrok.io/webhooks/toku
```

#### 2. Webhook.cool
```
Crear URL temporal: https://webhook.cool
Registrar en Toku
Ver requests en tiempo real
```

#### 3. API de Testing Toku
```bash
curl -X POST https://api.trytoku.com/webhooks/test \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "event_type": "invoice.paid",
    "webhook_id": "hook_123"
  }'
```

---

## 📊 Monitoreo de Webhooks

### Métricas Importantes
- ✅ Webhooks exitosos (200)
- ❌ Webhooks fallidos (timeout/error)
- ⏱️ Latencia promedio
- 🔄 Tasa de reintentos
- 📉 Errores por tipo de evento

### Dashboard de Monitoreo
```
Total enviados: 1,523
Exitosos: 1,510 (99.1%)
Fallidos: 8 (0.5%)
Pendientes: 5 (0.3%)
Latencia promedio: 245ms
```

---

## 🚨 Troubleshooting

### Webhook no se Recibe
1. ✓ Verificar status HTTP (debe retornar 200)
2. ✓ Verificar HTTPS válido
3. ✓ Revisar logs de servidor
4. ✓ Comprobar firewall/IP whitelist
5. ✓ Usar ngrok para debugging local

### Signature Inválida
```python
# Verificar que estés usando el secret correcto
secret_esperado = "xyzabc123"  # ¿Es el correcto?

# Verificar que uses el body raw, no parsed JSON
body = request.get_data()  # ✓ Correcto
body = json.dumps(request.json)  # ❌ Incorrecto
```

### Eventos Duplicados
```python
# Implementar deduplicación por event_id
@unique
class WebhookEvent(models.Model):
    event_id = models.CharField(unique=True)
    processed = models.BooleanField(default=False)
```

---

## 📚 Documentación de Referencia

- [API Reference Completa](./api-reference.md)
- [Backend Integration](./backend-integration.md)
- [Frontend Channels](./frontend-channels.md)
- [Glosario](./glosario.md)
- [Documentación Oficial Toku](https://toku.readme.io/reference)

---

**Última actualización**: Documentación 2024  
**Documentación oficial**: https://toku.readme.io/reference
