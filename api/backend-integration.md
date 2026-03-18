# Backend Integration | Integración Backend

Guía completa de las diferentes formas para integrar Toku en tu sistema backend.

---

## 📊 Comparativa de Métodos de Integración

| Método | Complejidad | Tiempo Implementación | Esfuerzo | Recomendado Para |
|--------|------------|----------------------|---------|-----------------|
| **API** | Alta | 1-2 semanas | Alto | Integración con datos en tiempo real |
| **SFTP** | Media | 3-5 días | Medio | Cargas batch diarias/semanales |
| **Formulario Manual** | Baja | Inmediato | Bajo | Entrada ocasional o pruebas |

---

## 🔌 Integración vía API

### Descripción
La integración API permite una conexión directa entre tu sistema y Toku en **tiempo real**. Es la opción más potente y flexible.

### Ventajas
✅ Sincronización en **tiempo real**  
✅ Control total sobre datos y flujos  
✅ Manejo de errores automático  
✅ Webhooks para notificaciones  
✅ Ideal para aplicaciones transaccionales  

### Desventajas
❌ Requiere desarrollo técnico  
❌ Mayor complejidad inicial  
❌ Requiere autenticación y seguridad  
❌ Necesita mantenimiento continuo  

### Métodos API Disponibles

#### Clientes (Customers)
- `POST /customers` - Crear cliente
- `GET /customers/{id}` - Obtener detalles
- `PUT /customers/{id}` - Actualizar cliente
- `GET /customers` - Listar clientes

#### Suscripciones (Subscriptions)
- `POST /subscriptions` - Crear suscripción
- `GET /subscriptions/{id}` - Obtener detalles
- `PUT /subscriptions/{id}` - Actualizar
- `GET /subscriptions` - Listar suscripciones

#### Deudas (Invoices)
- `POST /invoices` - Crear factura/deuda
- `GET /invoices/{id}` - Obtener detalles
- `GET /invoices` - Listar deudas

#### Medios de Pago (Payment Methods)
- `POST /payment_methods` - Crear medio de pago
- `GET /payment_methods/{id}` - Obtener detalles
- `GET /payment_methods` - Listar medios

#### Transacciones
- `POST /transactions` - Crear transacción
- `GET /transactions/{id}` - Obtener detalles
- `GET /transactions` - Listar transacciones

#### Webhooks
- `POST /webhooks` - Crear webhook
- `GET /webhooks/{id}` - Obtener detalles
- `PUT /webhooks/{id}` - Actualizar webhook
- `DELETE /webhooks/{id}` - Eliminar webhook

### Autenticación API

```
Header: Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

---

## 📁 Integración vía SFTP

### Descripción
Carga de datos mediante archivos batch a través de protocolo SFTP. Perfecta para sincronizaciones periódicas.

### Ventajas
✅ Bajo acoplamiento con tu sistema  
✅ Ideal para cargas batch  
✅ No requiere API keys en tiempo real  
✅ Compatible con sistemas legacy  
✅ Procesamiento estandarizado  

### Desventajas
❌ No es tiempo real  
❌ Requiere procesamiento batch  
❌ Sintonización más lenta  
❌ Menos flexible para ajustes  

### Proceso SFTP

#### 1. Preparación de Datos
Estructura de archivos CSV/JSON:

```csv
# Clientes
customer_id,email,name,phone
1,cliente@example.com,Juan Perez,+56912345678

# Deudas
invoice_id,customer_id,amount,due_date,description
INV-001,1,50000,2024-12-31,Pago de servicio
```

#### 2. Conexión SFTP
```
Host: sftp.toku.com
Usuario: tu_usuario_sftp
Puerto: 22
Ruta: /incoming/
```

#### 3. Carga de Archivos
- Colocar archivos en directorio `/incoming/`
- Nombrar con formato: `customers_YYYYMMDD.csv`
- Toku procesa automáticamente
- Confirmación enviada por email

#### 4. Frecuencia de Cargas
- **Diaria**: Para operaciones normales
- **Semanal**: Para volúmenes reducidos
- **Mensual**: Para datos maestros

### Formato de Archivos Soportados

**CSV:**
```
customer_id,email,name,subscription_id,amount,due_date
101,cliente@mail.com,Cliente 1,SUB-001,25000,2024-12-31
102,otro@mail.com,Cliente 2,SUB-002,30000,2024-12-31
```

**JSON:**
```json
{
  "customers": [
    {
      "customer_id": "101",
      "email": "cliente@mail.com",
      "name": "Cliente 1"
    }
  ],
  "invoices": [
    {
      "invoice_id": "INV-001",
      "customer_id": "101",
      "amount": 25000,
      "due_date": "2024-12-31"
    }
  ]
}
```

---

## 📝 Form-Based Entry | Entrada Manual mediante Formulario

### Descripción
Portal web para entrada manual de datos. Dos modalidades disponibles.

### Opción 1: Ejecutivos (Portal de Toku)
- Acceso directo en dashboard de Toku
- Interfaz visual e intuitiva
- Ideal para operadores/ejecutivos
- Soporte en tiempo real

### Opción 2: Autogestionado (Portal del Cliente)
- Los clientes pueden ingresar sus datos
- Reducción de carga para tu equipo
- Transparencia en el proceso
- Notificaciones automáticas

### Casos de Uso
- ✅ Pruebas iniciales
- ✅ Datos ocasionales
- ✅ Correcciones puntuales
- ✅ Clientes nuevos
- ❌ NO recomendado para alto volumen

---

## 🔔 Webhooks | Notificaciones en Tiempo Real

### Descripción
Sistema de notificaciones para eventos importantes en tu cuenta Toku.

### Eventos Disponibles
- `customer.created` - Nuevo cliente creado
- `customer.updated` - Cliente actualizado
- `subscription.created` - Nueva suscripción
- `invoice.created` - Nueva deuda/factura
- `invoice.paid` - Deuda pagada
- `payment_method.created` - Nuevo medio de pago
- `transaction.completed` - Transacción completada
- `transaction.failed` - Transacción fallida
- `settlement.created` - Liquidación creada

### Configuración de Webhook

#### Paso 1: Crear Endpoint
```bash
POST https://api.example.com/webhooks/toku
```

#### Paso 2: Registrar en Toku
```json
{
  "url": "https://api.example.com/webhooks/toku",
  "events": [
    "invoice.created",
    "invoice.paid",
    "transaction.completed"
  ],
  "active": true
}
```

#### Paso 3: Procesar Evento
```json
{
  "id": "evt_123456",
  "type": "invoice.paid",
  "data": {
    "invoice_id": "INV-001",
    "customer_id": "101",
    "amount": 25000,
    "paid_at": "2024-12-20T10:15:00Z"
  },
  "timestamp": "2024-12-20T10:15:00Z"
}
```

### Best Practices Webhooks
✅ Guardar el `id` del evento para evitar duplicados  
✅ Procesar eventos de forma asíncrona  
✅ Implementar reintentos en caso de fallo  
✅ Usar HTTPS con certificado válido  
✅ Registrar todos los eventos recibidos  

---

## 🔐 Seguridad en Backend

### Autenticación
- Usar **API Keys** con rotación periódica
- Nunca guardar keys en código fuente
- Usar variables de entorno
- Implementar RBAC (Role-Based Access Control)

### Validación de Datos
- Validar totales antes de enviar
- Verificar IDs únicos de clientes
- Comprobar montos y fechas
- Desduplicar antes de cargar

### Monitoreo
- Registrar todos los eventos
- Alertas para fallos de API
- Audit logs de cambios
- Monitoreo de cuotas API

---

## 📚 Documentación de Referencia

- [API Reference Completa](./api-reference.md)
- [Glosario de Términos](./glosario.md)
- [Buenas Prácticas](./buenas-practicas.md)
- [Documentación Oficial Toku](https://toku.readme.io/docs/backend)

---

**Última actualización**: Documentación 2024  
**Documentación oficial**: https://toku.readme.io/docs/backend
