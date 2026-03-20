# Troubleshooting — Toku API

---

## Errores de Autenticación

| Error | Causa | Solución |
|-------|-------|---------|
| `401 Unauthorized` | API key incorrecta o expirada | Verificar `Authorization: Bearer KEY` |
| `401 Unauthorized` | IP no en whitelist | Agregar IP al whitelist en dashboard |
| `403 Forbidden` | Key sin permisos para ese recurso | Verificar permisos de la API key |

```bash
# Verificar que el header es correcto
curl -H "Authorization: Bearer $TOKU_API_KEY" \
     https://api.trytoku.com/customers
```

---

## Errores de Validación

| Error | Causa | Solución |
|-------|-------|---------|
| `400 Bad Request` | JSON malformado | Validar JSON antes de enviar |
| `422 Unprocessable Entity` | Campo requerido faltante | Revisar campos obligatorios |
| `409 Conflict` | `external_id` ya existe | Usar ID único o actualizar recurso existente |

---

## Webhooks no Llegan

**Checklist:**
1. ¿Tu endpoint retorna `HTTP 200`? (no 201, no 204, no 302 — debe ser exactamente 200)
2. ¿Tu endpoint es accesible desde internet? (no `localhost`)
3. ¿Tiene HTTPS con certificado válido?
4. ¿Tu firewall bloquea las IPs de Toku?
5. ¿El webhook está marcado como `active: true`?

```bash
# Probar endpoint manualmente
curl -X POST https://tu-dominio.com/webhooks/toku \
  -H "Content-Type: application/json" \
  -d '{"id": "test", "type": "invoice.paid", "data": {}}'
```

**Para desarrollo local — usar ngrok:**
```bash
ngrok http 5000
# Obtiene: https://abc123.ngrok.io → registrar en Toku
```

---

## Firma de Webhook Inválida

El error más común: usar el body JSON parseado en lugar del body raw.

```python
# ❌ INCORRECTO — altera el body
body = json.dumps(request.json)

# ✅ CORRECTO — bytes originales tal como los envió Toku
body = request.get_data()

# Validación completa
signature = request.headers.get('X-Toku-Signature')
expected = hmac.new(secret.encode(), body, hashlib.sha256).hexdigest()
is_valid = hmac.compare_digest(signature, expected)
```

**Otras causas de firma inválida:**
- Usar el secret equivocado (verificar en dashboard)
- Middleware que altera el body antes de llegar a tu handler (ej: gzip)

---

## Eventos Duplicados

Toku puede reenviar el mismo evento si tu servidor no respondió 200 a tiempo.

```python
# Solución: deduplicar por event_id
def handle_webhook(event):
    event_id = event['id']

    # Verificar antes de procesar
    if WebhookEvent.objects.filter(event_id=event_id).exists():
        return 200  # Ya procesado — ignorar

    # Guardar y procesar
    WebhookEvent.objects.create(event_id=event_id)
    process(event)
    return 200
```

---

## Invoice no se Cobra

**Causas posibles:**
1. No hay `payment_method` activo asociado al customer/subscription
2. El `payment_method` está en estado `inactive` o `deleted`
3. La invoice ya fue pagada o anulada

```bash
# Verificar payment methods del customer
GET /customers/{id}/payment_methods

# Verificar estado de la invoice
GET /invoices/{id}
```

---

## Cobro Falla Constantemente

| Código de Error | Causa | Acción |
|----------------|-------|--------|
| `insufficient_funds` | Sin saldo | Notificar al cliente, reintentar en días de pago |
| `card_declined` | Tarjeta rechazada | Pedir al cliente actualizar medio de pago |
| `account_closed` | Cuenta cerrada | Solicitar nuevo medio de pago |
| `invalid_account` | Cuenta inválida | Verificar número de cuenta (CLABE, CBU) |
| `expired_card` | Tarjeta vencida | Pedir actualización |

---

## Customer No Encontrado

```bash
# Buscar por external_id (el ID de tu sistema)
GET /customers/external/{tu_external_id}

# Si no existe, crearlo primero
POST /customers
```

---

## Rate Limiting (429 Too Many Requests)

- Implementar exponential backoff
- Cachear respuestas GET que no cambian frecuentemente
- Usar SFTP para cargas masivas en lugar de N requests individuales

```python
import time

def request_with_retry(fn, max_retries=3):
    for attempt in range(max_retries):
        response = fn()
        if response.status_code == 429:
            time.sleep(2 ** attempt)  # 1s, 2s, 4s
            continue
        return response
    raise Exception("Rate limit persistente")
```

---

## Debugging Local con Logs

```python
import logging

logger = logging.getLogger(__name__)

@app.route('/webhooks/toku', methods=['POST'])
def handle_webhook():
    logger.info("Webhook recibido", extra={
        'event_id': request.json.get('id'),
        'event_type': request.json.get('type'),
        'customer_id': request.json.get('data', {}).get('customer_id'),
    })
    # ...
```

---

## Escalación

Si el problema persiste después de revisar esta guía:
1. Revisar logs completos del webhook en el dashboard de Toku
2. Usar `POST /webhooks/test` para verificar conectividad
3. Contactar soporte técnico Toku con el `event_id` del evento fallido
