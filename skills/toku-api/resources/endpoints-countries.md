# Endpoints por País — Toku API

Cada país tiene endpoints y medios de pago específicos.

---

## 🇨🇱 Chile

### Medios de Pago Disponibles
- **PAC** — Pago Automático de Cuenta (débito bancario recurrente)
- **PAT** — Pago Automático Tarjeta (débito/crédito/prepago recurrente)
- Transferencia bancaria
- Moneda: **CLP**

### Customer Collection Account
Cuenta de cobro asociada a un customer (para recibir transferencias).

```
GET    /chile/customer_collection_accounts/{id}
POST   /chile/customer_collection_accounts
DELETE /chile/customer_collection_accounts/{id}
```

### Checkout Session
Sesión de pago temporal con URL única para redirigir al cliente.

```
POST   /checkout/sessions              → Crear sesión
GET    /checkout/sessions/{id}         → Obtener sesión
POST   /checkout/sessions/{id}/expire  → Expirar manualmente
PATCH  /checkout/sessions/{id}         → Actualizar sesión
```

```bash
# Crear checkout session
curl -X POST https://api.trytoku.com/checkout/sessions \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_id": "cust_123",
    "invoice_ids": ["inv_001"],
    "success_url": "https://miempresa.cl/pago-exitoso",
    "cancel_url": "https://miempresa.cl/pago-cancelado"
  }'
```

### Checkout Elements
Formularios embebibles para inscripción y pago.

```
POST /checkout/enrollment   → Form inscripción de medio de pago
POST /checkout/payment      → Form de pago de deuda
```

### Redirections
Redirecciones post-pago configurables.

```
POST /redirections      → Crear redirection
PUT  /redirections/{id} → Editar redirection
```

### Cards (Requiere PCI Compliance)
Solo organizaciones con certificación PCI pueden usar estos endpoints directamente.

```
POST /chile/encryption_keys                   → Obtener llave pública RSA
POST /chile/transactions/card                 → Cobrar con tarjeta
POST /chile/payment_methods/card/async        → Inscribir tarjeta (async)
POST /chile/payment_methods/card/sync         → Inscribir tarjeta (sync)
```

**Flujo de encriptación de tarjeta:**
1. `GET /chile/encryption_keys` → obtener llave pública
2. Encriptar datos de tarjeta en cliente con RSA
3. Enviar datos encriptados al endpoint de inscripción

---

## 🇲🇽 México

### Medios de Pago Disponibles
- **Tarjeta** de crédito/débito
- **Cuenta CLABE** (VCC — Validación de Cuenta CLABE)
- **Cash** (OXXO u otras referencias de pago en efectivo)
- **Transferencia SPEI**
- Moneda: **MXN**

### Cards

```
POST /mexico/encryption_keys           → Obtener llave pública RSA
POST /mexico/payment_methods/card      → Crear PaymentMethod tipo Card
```

### Bank Accounts (CLABE)

```
POST /mexico/payment_methods/bank_account         → Crear cuenta CLABE
POST /mexico/payment_methods/bank_account/verify  → Verificar cuenta CLABE
POST /mexico/payment_methods/bank_account/batch   → Carga batch de cuentas
```

**Flujo VCC (Validación CLABE):**
1. Customer proporciona número CLABE
2. `POST /mexico/payment_methods/bank_account` → crear
3. Toku envía micro-depósito de verificación
4. `POST /mexico/payment_methods/bank_account/verify` → confirmar monto
5. Cuenta queda activa para cobros

### Cash (Efectivo)

```
POST /mexico/cash/reference   → Crear referencia de pago en efectivo
```
Genera un código/referencia para que el cliente pague en OXXO u otros corresponsales.

### Transfer

```
GET /mexico/bank_accounts     → Obtener cuenta destino para SPEI
```

### Facturación (SAT México)

```
GET    /mexico/customers/{id}/billing    → Info facturación del customer
POST   /mexico/customers/{id}/billing    → Crear datos de facturación
PATCH  /mexico/customers/{id}/billing    → Actualizar
GET    /mexico/invoices/{id}/billing     → Info facturación de la deuda
POST   /mexico/invoices/{id}/billing     → Crear
PATCH  /mexico/invoices/{id}/billing     → Actualizar
POST   /mexico/billing/public            → Datos Público en General (sin RFC)
```

### Facturas México (CFDI)

```
GET  /mexico/facturas               → Listar facturas emitidas
POST /mexico/facturas/ppd           → Emitir PPD (Pago en Parcialidades o Diferido)
POST /mexico/facturas/ppd/complement → Complemento de pago PPD
POST /mexico/facturas/pue           → Emitir PUE (Pago en Una Exhibición)
POST /mexico/facturas/cancel        → Cancelar factura
POST /mexico/facturas/credit_note   → Emitir Nota de Crédito
```

---

## 🇧🇷 Brasil

### Medios de Pago Disponibles
- **Pix** (estándar e automático)
- **Boleto bancário**
- **Débito Automático**
- Moneda: **BRL**

### Pix Automático

```
POST /brasil/pix/automatic   → Cadastro via notificación push
```
Permite inscribir Pix automático cuando el banco del cliente envía una notificación.

### Configuração Brasil

```
GET    /brasil/config    → Obter configuração da organização
POST   /brasil/config    → Criar configuração
PATCH  /brasil/config    → Atualizar configuração
DELETE /brasil/config    → Eliminar configuração
```

### Boleto

```
POST /brasil/boleto/pdf    → Obtener PDF del boleto para imprimir/enviar
POST /brasil/boleto/init   → Inicializar proceso de pago por boleto
```

### Pix (Estándar)

```
POST /brasil/pix/init    → Inicializar pago Pix (genera QR code / copia e cola)
```

### Débito Automático

```
POST /brasil/debit/register   → Cadastrar conta para débito automático
```

---

## Resumen de Medios de Pago por País

| Medio de Pago | Chile | México | Brasil |
|--------------|-------|--------|--------|
| Débito bancario automático | PAC | — | Débito Automático |
| Tarjeta automática | PAT | Card | — |
| Cuenta bancaria validada | — | CLABE (VCC) | — |
| Pago instantáneo | — | SPEI | Pix |
| Efectivo/corresponsal | — | OXXO/Cash | Boleto |
| Facturación fiscal | — | CFDI/SAT | — |
