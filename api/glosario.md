# Glosario de Toku

Estas son las palabras clave que te ayudarán a entender mejor la documentación de Toku.

---

## Conceptos Principales

### Organization | Organización
Es el comercio/empresa que contrata Toku. Es la entidad primaria en el sistema que configura y utiliza todos los servicios de Toku.

### Customer | Cliente
Hace referencia al usuario que pagará a la organización. Es la persona o entidad que debe realizar pagos por productos/servicios.

### Subscription | Suscripción | Producto
Generalmente se utiliza para identificar un plan de pagos que está asociado a varias deudas o invoices. 
Se identifica con el ID producto.

**Nota**: Es importante no confundir con una suscripción en el sentido de "membresía recurrente". En Toku, una Subscription es un contenedor lógico de deudas.

### Invoice | Deudas
Son las deudas/cobros individuales de cada cliente/customer. 
Se identifican con un ID que puede ser:
- Entregado por la organización (external_id), o
- Generado automáticamente pelo par: `product_id + fecha de vencimiento`

---

## Medios de Pago

### Payment Method | Medio de Pago
Medio de pago automático vinculado a un cliente para realizar cobros recurrentes. Puede ser:
- **PAC** (Chile): Pago automático de cuenta
- **PAT** (Chile): Pago automático tarjeta (débito, crédito o prepago)
- **VCC** (México): Validación de Cuenta Clabe
- **Tarjeta de Crédito/Débito** (Internacional)
- **Cuenta Bancaria** (Internacional)
- **Pix** (Brasil)
- **Boleto** (Brasil)

---

## Intento de Pago

### Payment Intent | Intento de Pago
Intento de un cliente/customer a realizar un pago en una invoice/deuda específica.

**Estados posibles:**
- ✅ Exitoso
- ❌ Fallido
- ⏳ Procesando cobro (estado intermedio)

**Características:**
- Está relacionado exclusivamente a una invoice/deuda
- Se genera cuando se intenta cobrar un medio de pago específico

### Transaction | Transacción
También es un intento de un cliente/customer a realizar un pago.

**Estados posibles:**
- ✅ Exitoso
- ❌ Fallido
- ⏳ Inicializado

**Diferencia con Payment Intent:**
- Una Transaction puede agrupar múltiples invoices/deudas
- Es el nivel más alto de intentos de pago
- Se utiliza para operaciones que transcienden una single invoice

---

## Conceptos Tecnológicos

### PAC | Pago Automático de Cuenta 
**(Solo Chile)**
Sistema de débito automático desde una cuenta bancaria del cliente para realizar pagos recurrentes.

### PAT | Pago Automático Tarjeta 
**(Solo Chile)**
Sistema de cobro automático a través de una tarjeta (débito, crédito o prepago).

### VCC | Validación de Cuenta Clabe 
**(Solo México)**
Validación de la cuenta bancaria utilizando el estándar CLABE (Clave Bancaria Estandarizada) en México.

---

## Flujos de Pago

### Settlement | Liquidación
Proceso en el cual los fondos cobrados son depositados en la cuenta bancaria de la organización. Incluye detalles de:
- Monto total depositado
- Comisiones aplicadas
- Transacciones incluidas
- Fecha de deposito

---

## Resumen de Relaciones

```
Organization
    ├── Customer
    │   ├── Subscription (product_id)
    │   │   └── Invoice/Deuda
    │   │       ├── Payment Intent
    │   │       └── Transaction
    │   └── Payment Method
    │       ├── PAC / PAT (Chile)
    │       ├── VCC (México)
    │       ├── Card (Internacional)
    │       ├── Bank Account (Internacional)
    │       ├── Pix (Brasil)
    │       └── Boleto (Brasil)
    └── Settlement
        └── Transacciones liquidadas
```

---

**Documentación oficial**: https://toku.readme.io/docs/glosario
