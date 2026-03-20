# Toku — Arquitectura y Modelo de Datos

## Modelo de Entidades

```
Organization (empresa que usa Toku)
    ├── Customer (cliente que paga)
    │   ├── Subscription / product_id (contenedor lógico de deudas)
    │   │   └── Invoice (deuda individual)
    │   │       ├── Payment Intent (intento de cobro a 1 invoice)
    │   │       └── Transaction (puede agrupar múltiples invoices)
    │   └── Payment Method (medio de pago inscrito)
    │       ├── PAC / PAT          (Chile)
    │       ├── VCC / Card / BankAccount (México)
    │       ├── Card / BankAccount (Internacional)
    │       └── Pix / Boleto       (Brasil)
    └── Settlement (liquidación de fondos a cuenta de la org)
```

## Glosario

| Término | Definición |
|---------|-----------|
| **Organization** | La empresa que contrata Toku. Entidad raíz del sistema. |
| **Customer** | Usuario final que paga. Tiene email, nombre, teléfono, `external_id`. |
| **Subscription** | Contenedor lógico de deudas. **No es membresía recurrente**. Se identifica con `product_id`. Un customer puede tener múltiples subscriptions. |
| **Invoice** | Deuda individual de un customer. ID: `external_id` propio o par `product_id + due_date`. |
| **Payment Method** | Medio de pago automático inscrito al customer para cobros recurrentes. |
| **Payment Intent** | Intento de cobro asociado a exactamente **1 invoice**. |
| **Transaction** | Intento de pago de nivel alto. Puede agrupar **múltiples invoices**. |
| **Settlement** | Liquidación: depósito de los fondos cobrados a la cuenta bancaria de la org. Incluye fees y neteo. |
| **Checkout Session** | Sesión de pago temporal (Chile) que genera una URL única para que el cliente pague. |
| **PAC** | Pago Automático de Cuenta — débito bancario automático (solo Chile). |
| **PAT** | Pago Automático Tarjeta — cobro a tarjeta de débito/crédito/prepago (solo Chile). |
| **VCC** | Validación de Cuenta CLABE — validación de cuenta bancaria en México (estándar CLABE). |

## Relaciones Clave

- Un **Customer** puede tener N **Subscriptions**
- Una **Subscription** puede tener N **Invoices**
- Un **Invoice** puede tener N **Payment Intents** (reintentos)
- Un **Customer** puede tener N **Payment Methods**
- Un **Payment Method** puede estar asociado a N **Subscriptions**
- Un **Transaction** puede pagar N **Invoices** a la vez

## Estados de Invoice

| Estado | Significado |
|--------|-------------|
| `pending` | Pendiente de pago |
| `paid` | Pagada completamente |
| `partially_paid` | Pago parcial recibido |
| `failed` | Cobro fallido |
| `void` | Anulada |
| `overdue` | Vencida sin pago |

## Estados de Transaction / Payment Intent

| Estado | Significado |
|--------|-------------|
| `completed` | Exitoso |
| `failed` | Fallido |
| `pending` | En proceso / esperando banco |
| `initialized` | Iniciado pero no procesado |

## Autenticación

```bash
# Todas las requests requieren:
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

- API Keys obtenidas en el dashboard de Toku
- También disponible: **OAuth M2M** para server-to-server
- Nunca hardcodear keys — usar variables de entorno
- IP Whitelist disponible para mayor seguridad
