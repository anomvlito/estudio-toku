# Guía de Integración Backend — Toku

---

## Métodos de Integración

| Método | Complejidad | Tiempo setup | Para qué |
|--------|------------|-------------|---------|
| **API REST** | Alta | 1-2 semanas | Tiempo real, control total, alta frecuencia |
| **SFTP** | Media | 3-5 días | Cargas batch diarias/semanales, sistemas legacy |
| **Formulario Manual** | Nula | Inmediato | Pruebas, correcciones puntuales, bajo volumen |

---

## Cuándo usar cada método

### API REST
**Usar cuando:**
- Tus datos cambian frecuentemente o en tiempo real (nuevas deudas, clientes, pagos)
- Necesitas control total sobre el flujo de creación/modificación
- Requieres webhooks para sincronización inmediata
- Tienes un equipo técnico que puede mantener la integración

**No usar cuando:**
- Solo necesitas cargar datos históricos una vez
- Tu sistema no soporta llamadas HTTP externas

### SFTP
**Usar cuando:**
- Tienes cargas batch (ej: "todas las deudas del mes" el día 1)
- Tu sistema ya exporta CSVs o JSONs
- Tienes sistemas legacy que no pueden hacer HTTP
- Quieres bajo acoplamiento con Toku

**No usar cuando:**
- Necesitas que los cambios se reflejen en minutos
- Tus volúmenes son bajos y variables

### Formulario Manual
**Usar cuando:**
- Estás en fase de prueba/POC
- Necesitas cargar un cliente o deuda esporádicamente
- Tu equipo de operaciones hace correcciones puntuales

---

## Flujo Típico Completo (API REST)

```
1. Autenticarse
   Authorization: Bearer YOUR_API_KEY

2. Crear Customer
   POST /customers
   → { customer_id: "cust_123" }

3. Crear Subscription (producto/plan)
   POST /subscriptions
   → { subscription_id: "sub_456", product_id: "plan-mensual" }

4. Crear Invoice (deuda mensual)
   POST /invoices
   → { invoice_id: "inv_001" }

5. Cliente inscribe medio de pago
   → Checkout Session (Chile) o Iframe/WebLink

6. Webhook payment_method.created
   → Tu sistema recibe confirmación de inscripción

7. Ejecutar cobro
   POST /charge { invoice_id, payment_method_id }

8. Webhook invoice.paid o invoice.failed
   → Tu sistema actualiza el estado del pago

9. Webhook settlement.created
   → Confirmar liquidación de fondos recibida
```

---

## SFTP

```
Host:   sftp.toku.com
Puerto: 22
Ruta:   /incoming/
```

**Formato de archivos:**
```csv
# customers_20241201.csv
customer_id,email,name,phone
1,cliente@mail.com,Juan Pérez,+56912345678

# invoices_20241201.csv
invoice_id,customer_id,amount,due_date,description
INV-001,1,50000,2024-12-31,Cuota diciembre
```

**Frecuencia recomendada:**
- Diaria → operaciones normales
- Semanal → volúmenes reducidos
- Mensual → solo datos maestros (clientes/productos)

---

## Buenas Prácticas de Integración

### 1. IDs únicos y permanentes
```python
# ✅ Correcto: ID del cliente en tu sistema
customer = create_customer(external_id="tu-sistema-user-789")

# ❌ Incorrecto: ID generado dinámicamente o temporal
customer = create_customer(external_id=str(uuid4()))
```

### 2. Idempotencia en webhooks
```python
def handle_webhook(event):
    # Verificar si ya procesamos este evento
    if already_processed(event['id']):
        return 200  # OK sin reprocesar

    save_event(event['id'])
    process(event)
```

### 3. Procesamiento async de webhooks
```python
# ❌ Bloqueante — Toku puede hacer timeout
def webhook():
    slow_db_operation()   # 10 segundos
    return 200

# ✅ Async — retornar 200 inmediatamente
def webhook():
    queue.push(request.json)
    return 200  # Toku recibe 200 en < 1 segundo

@worker
def process_queued_event(event):
    slow_db_operation()
```

### 4. Validación antes de enviar datos

```python
def prepare_invoice(data: dict) -> dict:
    assert data.get('customer_id'), "customer_id requerido"
    assert data.get('amount') > 0, "amount debe ser positivo"
    assert data.get('due_date'), "due_date requerido"
    assert data.get('currency') in ['CLP', 'MXN', 'BRL'], "currency inválida"
    return data
```

### 5. Múltiples métodos de pago
Habilitar todos los medios disponibles por región aumenta la conversión.
- Chile: PAC + PAT + Transferencia
- México: Card + CLABE + Cash
- Brasil: Pix + Boleto + Débito automático

### 6. Reintentos de cobro
Toku puede configurar reglas de reintento automático cuando un cobro falla.
Algunas empresas pasan de 2 intentos/mes a 15, logrando +5% de recaudación adicional.

---

## Seguridad

- API Keys en variables de entorno (`TOKU_API_KEY=...`)
- IP Whitelist: configurar IPs autorizadas en Toku
- HTTPS obligatorio para webhooks e iframes
- Nunca loguear el body completo de webhooks (puede tener datos sensibles)
- Rotar API Keys periódicamente
