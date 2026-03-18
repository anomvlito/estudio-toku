# API Reference - Introducción

¡Bienvenido a Toku! 🤖 🚀

## Información General

**Base URL**: `https://api.trytoku.com`

La API de Toku entrega respuestas en formato JSON con códigos HTTP estándar para indicar el éxito de la operación. 

### Requisitos
- **Content-Type**: `application/json`
- **Body**: JSON válido

---

## 📋 Secciones de la API

### 🔐 Autenticación
- **API Key**
- **OAuth M2M**
- **Cuentas de Cobro**
- **Headers & Body**
- **IP & Puertos de conexión**
- **Códigos de respuesta**
- **Versionamiento**

### 👥 Customers (Clientes)
- Objeto Customer
- Crear Customer (POST)
- Editar Customer (PUT)
- Obtener Customer (GET)
- Listar Customers (GET)
- Obtener Customer por external_id (GET)
- Obtener Invoices de un customer (GET)
- Eliminar Customer (DEL)
- Webhooks

### 📅 Subscriptions (Suscripciones)
- Objeto Subscription
- Crear Subscription (POST)
- Editar Subscription (PUT)
- Obtener Subscription (GET)
- Listar Subscriptions (GET)
- Obtener Subscription por product_id (GET)
- Eliminar Subscription (DEL)
- Obtener Subscriptions de un Customer (GET)
- Webhooks

### 📄 Invoices (Deudas)
- Objeto Invoice
- Crear Invoice (POST)
- Obtener Invoice (GET)
- Obtener Invoice por invoice_external_id (GET)
- Listar invoices (GET)
- Anular Invoice (POST)
- Eliminar Invoice (DEL)
- Editar Invoice (PUT)
- Webhooks

### 🏠 Entities (Entidades)
- Crear Entity (POST)

### 🇨🇱 Chile
#### Customer Collection Account
- Objeto Customer Collection Account
- Obtener Customer Collection Account (GET)
- Crear Customer Collection Account (POST)
- Eliminar Customer Collection Account (DEL)

#### Checkout Session
- Objeto Checkout Session
- Create a Checkout Session (POST)
- Retrieve Checkout Session (GET)
- Expire Checkout Session (POST)
- Update Checkout Session (PATCH)

#### Checkout Elements
- Create Enrollment Checkout Form
- Create Payment Checkout Form

#### Redirections
- Objeto Redirection
- Crear Redirection (POST)
- Editar Redirection (PUT)

### 💳 Transacciones & Pagos
- Objeto Transaction
- Obtener todas las Transactions (GET)
- Obtener una Transaction (GET)
- Obtener el detalle del Fee por Transaction (GET)

#### Cobro
- Cobro (POST)
- Cobro Bulk (POST)

#### Cards (Solo Organizaciones PCI Compliant)
- Encriptación de data
- Generar llave pública de encriptación (POST)
- Crear Transaction con medio de pago Card (POST)
- Inscribir tarjeta como medio de pago (asíncrono) (POST)
- Inscribir tarjeta como medio de pago (síncrono) (POST)

#### Payment Method
- Objeto Payment Method
- Eliminar Payment Method (DEL)
- Obtener todos los Payment Methods (GET)
- Obtener un Payment Method (GET)
- Obtener los Payment Methods de un Customer (GET)
- Asociar un Payment Method a una Subscription (POST)
- Asociar múltiples Payment Methods a Subscriptions (POST)

### 🇲🇽 Mexico
#### Cards
- El objeto PaymentMethod de tipo Card
- Encriptación de data
- Generar llave pública de encriptación
- Crear un PaymentMethod de tipo Card (POST)

#### Bank Accounts
- El objeto PaymentMethod de tipo BankAccount
- Crear un PaymentMethod de tipo BankAccount (POST)
- Verificar un BankAccount (POST)
- Crear PaymentMethod de tipo BankAccount en Batch (POST)

#### Cash
- Crear una Referencia (POST)

#### Transfer
- Obtener la BankAccount (GET)

### 🇧🇷 Brasil
#### Pix Automático
- Cadastro de Pix Automático via notificación push (POST)

#### Configuración
- Obter Configuração (GET)
- Criar Configuração (POST)
- Atualizar Configuração (PATCH)
- Eliminar Configuração (DEL)

#### Boleto
- Obter PDF do boleto (POST)
- Inicialización do Boleto (POST)

#### Pix
- Inicialización do Pix (POST)

#### Débito Automático
- Cadastro conta débito automático (POST)

### 💰 Payment Object
- Objeto Payment
- Eliminar Payments (DEL)
- Modificar Payment (PUT)
- Obtener Payments (GET)
- Obtener Payment (GET)
- Crear o Modificar Payments (POST)

### 🏦 Settlement
- El objeto Settlement
- Webhooks

### 🪝 Webhooks
- Objeto Webhook Endpoint
- Crear Webhook Endpoint (POST)
- Probar Webhook (POST)
- Editar Webhook Endpoint (PUT)
- Obtener Webhook Endpoint (GET)
- Obtener Webhook Endpoints (GET)

### 📢 Eventos
- Objeto Evento
- Listado de eventos

### 🇲🇽 Facturación
- Obtener Información de Facturación del Customer (GET)
- Crear Información de Facturación del Customer (POST)
- Actualizar Información de Facturación del Customer (PATCH)
- Obtener Información de Facturación de la Deuda (GET)
- Crear Información de Facturación de la Deuda (POST)
- Actualizar Información de Facturación de la Deuda (PATCH)
- Crear Información de facturación público general (POST)

### 📋 Facturas México
- Obtener Facturas (GET)
- PPD (POST)
- Complemento PPD (POST)
- PUE (POST)
- Cancelación (POST)
- Nota de Crédito (POST)

### 💳 Devoluciones
- Devolución de Transacción (POST)
- Devolución de Invoice (POST)

### 🛠️ Misc
- Payment Portal Session (POST)
- Catálogo de errores (Core)
- Catálogo de errores (Pagos y Medios de Pagos)

---

**Documentación oficial**: https://toku.readme.io/reference
