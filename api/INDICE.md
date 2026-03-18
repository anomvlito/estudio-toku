# Índice Completo de Documentación Toku

Índice navegable de toda la documentación disponible sobre la API de Toku.

---

## 📖 Documentación Disponible

### 🚀 Guías de Inicio (Para Principiantes)

#### 1. **[Getting Started](./getting-started.md)** - Introducción a Toku
Punto de partida ideal si eres nuevo en Toku.
- ✓ Estructura de la documentación
- ✓ Métodos de integración (Backend/Frontend)
- ✓ Primera integración paso a paso
- ✓ FAQ - Preguntas frecuentes

**Leer si**: Es tu primer contacto con Toku

---

#### 2. **[Casos de Uso](./casos-de-uso.md)** - Aplicaciones Prácticas
Entiende qué puede hacer Toku por tu negocio.
- ✓ 4 Pilares de Toku (recaudación, costos, experiencia, seguridad)
- ✓ 9 Casos de uso específicos
- ✓ 7 Industrias contempladas
- ✓ Propuesta de valor por sector

**Leer si**: Quieres entender el valor de negocio

---

#### 3. **[Glosario](./glosario.md)** - Términos y Definiciones
Referencia rápida de términos clave.
- ✓ Conceptos principales (Organization, Customer, Subscription, etc.)
- ✓ Medios de pago por región (PAC, PAT, VCC, etc.)
- ✓ Relaciones entre entidades
- ✓ Términos técnicos

**Leer si**: No sabes qué significa un término

---

### 🔧 Guías Técnicas (Para Desarrolladores)

#### 4. **[Backend Integration](./backend-integration.md)** - Métodos de Integración Backend
Guía completa para integrar Toku en tu sistema.
- ✓ **API REST** - Integración en tiempo real (1-2 semanas)
- ✓ **SFTP** - Cargas batch de datos (3-5 días)
- ✓ **Formularios** - Entrada manual o autogestión (inmediato)
- ✓ Comparativa de métodos, pros/contras, ejemplos de código

**Leer si**: Necesitas integrar Toku en tu backend

---

#### 5. **[API Reference](./api-reference.md)** - Referencia Técnica Completa
Documentación exhaustiva de todos los endpoints.
- ✓ 60+ endpoints documentados
- ✓ Recursos: Customers, Subscriptions, Invoices, Payment Methods, Transactions, Settlements
- ✓ Operaciones por región (Chile, México, Brasil)
- ✓ Autenticación y seguridad
- ✓ Códigos de error

**Leer si**: Necesitas documentación técnica de endpoints

---

#### 6. **[Frontend Channels](./frontend-channels.md)** - Canales de Pago Cliente
Opciones disponibles para que tus clientes realicen pagos.
- ✓ **Web Link** - Enlace directo al checkout
- ✓ **Iframe** - Incrustado en tu sitio web
- ✓ **QR Code** - Escaneo móvil
- ✓ **Mensajería** - WhatsApp, SMS, Telegram
- ✓ **Mobile WebView** - App nativa iOS/Android
- ✓ Ejemplos de código para cada canal

**Leer si**: Necesitas mostrar opciones de pago a tus clientes

---

#### 7. **[Webhooks](./webhooks.md)** - Sistema de Notificaciones
Sistema de eventos para sincronización en tiempo real.
- ✓ 15+ eventos disponibles
- ✓ Configuración y validación de webhooks
- ✓ Manejo de reintentos
- ✓ Best practices y testing
- ✓ Ejemplos en Python, Node.js
- ✓ Troubleshooting

**Leer si**: Necesitas sincronizar eventos en tiempo real

---

#### 8. **[Buenas Prácticas](./buenas-practicas.md)** - Recomendaciones
Recomendaciones para una integración exitosa.
- ✓ 5 prácticas clave
- ✓ Validación de datos
- ✓ Uso de webhooks
- ✓ IDs únicos
- ✓ Múltiples medios de pago

**Leer si**: Quieres integración robusta y confiable

---

## 🗺️ Mapa Mental de Navegación

```
¿NUEVO EN TOKU?
    ↓
    ├─→ [Getting Started](./getting-started.md)
    ├─→ [Casos de Uso](./casos-de-uso.md)
    └─→ [Glosario](./glosario.md)

QUIERO INTEGRAR BACKEND
    ↓
    ├─→ [Backend Integration](./backend-integration.md)
    ├─→ [API Reference](./api-reference.md)
    ├─→ [Webhooks](./webhooks.md)
    └─→ [Buenas Prácticas](./buenas-practicas.md)

QUIERO MOSTRAR PAGO A CLIENTES
    ↓
    ├─→ [Frontend Channels](./frontend-channels.md)
    ├─→ [Backend Integration](./backend-integration.md)
    └─→ [Buenas Prácticas](./buenas-practicas.md)

NECESITO ENDPOINT ESPECÍFICO
    ↓
    └─→ [API Reference](./api-reference.md)

NECESITO CONFIGURAR EVENTOS
    ↓
    ├─→ [Webhooks](./webhooks.md)
    └─→ [Backend Integration](./backend-integration.md)

NO ENTIENDO UN TÉRMINO
    ↓
    └─→ [Glosario](./glosario.md)
```

---

## 📊 Matriz de Documentación

| Documento | Nivel | Público | Temas |
|-----------|-------|---------|-------|
| [Getting Started](./getting-started.md) | Principiante | Todos | Introducción, FAQ |
| [Casos de Uso](./casos-de-uso.md) | Principiante | Product, Sales | Valor negocio |
| [Glosario](./glosario.md) | Básico | Técnico | Terminología |
| [Backend Integration](./backend-integration.md) | Intermedio | Desarrolladores | Integración backend |
| [API Reference](./api-reference.md) | Avanzado | Desarrolladores | Endpoints técnicos |
| [Frontend Channels](./frontend-channels.md) | Intermedio | Desarrolladores | Experiencia cliente |
| [Webhooks](./webhooks.md) | Avanzado | Desarrolladores | Notificaciones |
| [Buenas Prácticas](./buenas-practicas.md) | Intermedio | Desarrolladores | Mejores prácticas |

---

## 🎯 Rutas de Aprendizaje Recomendadas

### 🆕 Si nunca usaste Toku
1. [Getting Started](./getting-started.md) (15 min)
2. [Casos de Uso](./casos-de-uso.md) (10 min)
3. [Glosario](./glosario.md) (10 min)
4. [Overview Backend Integration](./backend-integration.md) (20 min)
5. **Total**: ~55 minutos

### 🔌 Si necesitas integración técnica
1. [Glosario](./glosario.md) (10 min) - si necesitas
2. [Backend Integration](./backend-integration.md) (30 min)
3. [API Reference](./api-reference.md) (20 min)
4. [Webhooks](./webhooks.md) (25 min)
5. [Buenas Prácticas](./buenas-practicas.md) (15 min)
6. **Total**: ~100 minutos

### 🎨 Si necesitas UX de pago
1. [Frontend Channels](./frontend-channels.md) (25 min)
2. [API Reference](./api-reference.md) - específico (15 min)
3. [Buenas Prácticas](./buenas-practicas.md) (15 min)
4. **Total**: ~55 minutos

### 🔧 Para troubleshooting
1. [Webhooks](./webhooks.md) - Sección Troubleshooting
2. [Backend Integration](./backend-integration.md) - Sección específica
3. [Buenas Prácticas](./buenas-practicas.md) - Recomendaciones

---

## 🔍 Búsqueda Rápida por Tema

### Autenticación
- [Backend Integration - Autenticación API](./backend-integration.md#autenticación)
- [API Reference - Autenticación](./api-reference.md)

### Clientes (Customers)
- [API Reference - Customers](./api-reference.md)
- [Glosario - Customer](./glosario.md)
- [Webhooks - customer.created/updated](./webhooks.md)

### Deudas (Invoices)
- [API Reference - Invoices](./api-reference.md)
- [Glosario - Invoice](./glosario.md)
- [Webhooks - invoice.created/paid/failed](./webhooks.md)

### Suscripciones
- [API Reference - Subscriptions](./api-reference.md)
- [Glosario - Subscription](./glosario.md)
- [Webhooks - subscription.created/updated](./webhooks.md)

### Medios de Pago
- [API Reference - Payment Methods](./api-reference.md)
- [Frontend Channels](./frontend-channels.md)
- [Glosario - Payment Methods](./glosario.md)

### Transacciones
- [API Reference - Transactions](./api-reference.md)
- [Glosario - Transacción](./glosario.md)
- [Webhooks - transaction.completed/failed](./webhooks.md)

### Pagos por País

**Chile:**
- PAC (Pago Automático de Cuenta) - [Glosario](./glosario.md)
- PAT (Pago Automático Tarjeta) - [Glosario](./glosario.md)
- [API Reference - Regional Chile](./api-reference.md)

**México:**
- VCC (Validación de Cuenta CLABE) - [Glosario](./glosario.md)
- Facturación - [API Reference](./api-reference.md)

**Brasil:**
- Pix - [Glosario](./glosario.md)
- Boleto - [Glosario](./glosario.md)

### Integración por Método

- **API REST**: [Backend Integration - API Section](./backend-integration.md#integración-vía-api)
- **SFTP**: [Backend Integration - SFTP Section](./backend-integration.md#integración-vía-sftp)
- **Manual/Formularios**: [Backend Integration - Forms Section](./backend-integration.md#form-based-entry)

### UX de Pago

- **Web Link**: [Frontend Channels - Web Link](./frontend-channels.md#canal-1-web-link)
- **Iframe**: [Frontend Channels - Iframe](./frontend-channels.md#canal-2-iframe)
- **QR Code**: [Frontend Channels - QR](./frontend-channels.md#canal-3-qr-code)
- **WhatsApp/SMS**: [Frontend Channels - Mensajería](./frontend-channels.md#canal-4-mensajería)
- **App Móvil**: [Frontend Channels - WebView](./frontend-channels.md#canal-5-mobile-webview)

---

## 📈 Estadísticas de Documentación

```
Total de Archivos:           8 documentos
Total de Endpoints:          60+ endpoints
Total de Eventos Webhook:    15+ eventos
Total de Ejemplos Código:    20+ ejemplos
Países Soportados:           3 (Chile, México, Brasil)
Medios de Pago:              8+ por región
Lenguajes Programación:      Python, JavaScript, Swift, Kotlin, Rust
Palabras Totales:            20,000+
```

---

## 🔗 Enlaces Útiles Directos

| Recurso | URL |
|---------|-----|
| Sitio Oficial Toku | https://trytoku.com |
| Documentación Original | https://toku.readme.io |
| API Base URL | https://api.trytoku.com |
| Getting Started Oficial | https://toku.readme.io/docs/getting-started |
| API Reference Oficial | https://toku.readme.io/reference |

---

## 💡 Consejos de Navegación

1. **Busca rápido**: Usa Ctrl+F (Cmd+F en Mac) dentro de cualquier documento
2. **Si no sabes un término**: Ve a [Glosario](./glosario.md)
3. **Si es tu primer vez**: Comienza por [Getting Started](./getting-started.md)
4. **Si necesitas código**: Consulta [Backend Integration](./backend-integration.md) o [Frontend Channels](./frontend-channels.md)
5. **Si tienes dudas técnicas**: [API Reference](./api-reference.md) es tu amigo
6. **Si necesitas síncronia en tiempo real**: [Webhooks](./webhooks.md) te cuenta cómo

---

## ✅ Checklist de Integración

- [ ] Leí [Getting Started](./getting-started.md)
- [ ] Entiendo los términos ([Glosario](./glosario.md))
- [ ] Seleccioné método de integración backend ([Backend Integration](./backend-integration.md))
- [ ] Seleccioné canales de pago frontend ([Frontend Channels](./frontend-channels.md))
- [ ] Revisé endpoints necesarios ([API Reference](./api-reference.md))
- [ ] Configuré webhooks ([Webhooks](./webhooks.md))
- [ ] Seguí buenas prácticas ([Buenas Prácticas](./buenas-practicas.md))

---

**Documentación compilada de**: https://toku.readme.io/  
**Última actualización**: 2024  
**Estado**: ✅ Documentación Completa
