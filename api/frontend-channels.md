# Frontend Channels | Canales de Acceso Frontend

Guía completa de los diferentes canales disponibles para que tus clientes accedan a Toku y realicen pagos.

---

## 📱 Resumen de Canales

| Canal | Complejidad | Tiempo Setup | Control UI | Recomendado Para |
|-------|------------|--------------|-----------|-----------------|
| **Web Link** | Muy Baja | < 1 día | Ninguno | Email, SMS, notificaciones |
| **Iframe** | Media | 2-3 días | Total | Sitio web propio |
| **QR Code** | Baja | < 1 día | Ninguno | Escaneo, papelería |
| **Mensajería** | Media | 2-3 días | Mínimo | WhatsApp, SMS directo |
| **Mobile WebView** | Alta | 3-5 días | Total | App móvil nativa |

---

## 🔗 Canal 1: Web Link | Enlace Web

### Descripción
Proporciona un **enlace directo** a Toku donde el cliente puede pagar. El más simple y rápido de implementar.

### Ventajas
✅ Implementación inmediata  
✅ No requiere desarrollo  
✅ Compatible con cualquier dispositivo  
✅ Perfecto para iniciar  
✅ Totalmente responsive  

### Desventajas
❌ Sin branding personalizado  
❌ Cambio de contexto (sale de tu sitio)  
❌ Menos control sobre UX  
❌ Requiere redirección  

### Implementación

#### Generar Enlace
```
https://pay.toku.com/checkout/[SESSION_ID]
```

#### Ejemplo en Email
```html
<a href="https://pay.toku.com/checkout/sess_123456" class="btn">
  Pagar Ahora
</a>
```

#### Ejemplo en SMS
```
Realiza tu pago aquí: https://pay.toku.com/checkout/sess_123456
```

### Características
- 🔒 URL única por sesión
- ⏱️ Validez configurable (15 min - 7 días)
- 💳 Múltiples medios de pago automáticos
- 🌐 Soporte multiidioma
- 📊 Analytics integrado

---

## 🪟 Canal 2: Iframe | Incrustación en Página

### Descripción
Integra formulario de pago **completamente dentro de tu sitio**. Máximo control sobre UX.

### Ventajas
✅ Branding completamente personalizado  
✅ Usuario continúa en tu sitio  
✅ Control total sobre flujo  
✅ Mejor conversión (menos cambios de contexto)  
✅ Integración con tu diseño  

### Desventajas
❌ Requiere desarrollo (HTML/CSS/JS)  
❌ Más complejo de implementar  
❌ Mayor configuración inicial  
❌ Necesita HTTPS obligatorio  
❌ Control de borders y sizing limitado  

### Implementación

#### HTML Base
```html
<div id="toku-payment-container"></div>

<script src="https://sdk.toku.com/js/checkout.js"></script>
<script>
  TokuCheckout.createCheckout({
    sessionId: 'sess_123456',
    container: '#toku-payment-container',
    theme: {
      primaryColor: '#007bff',
      fontFamily: 'Segoe UI, sans-serif'
    }
  });
</script>
```

#### Opciones de Personalización
```javascript
{
  // Sesión
  sessionId: 'sess_123456',
  
  // Contenedor
  container: '#checkout-div',
  
  // Tema
  theme: {
    primaryColor: '#007bff',
    successColor: '#28a745',
    dangerColor: '#dc3545',
    fontFamily: 'Arial, sans-serif'
  },
  
  // Comportamiento
  autoFocus: true,
  fullHeight: true,
  
  // Callbacks
  onSuccess: (result) => {
    console.log('Pago exitoso:', result);
  },
  onError: (error) => {
    console.error('Error:', error);
  }
}
```

### Eventos Disponibles
```javascript
// Pago completado exitosamente
TokuCheckout.on('success', (data) => {
  // data.transactionId, data.customerId, etc.
});

// Error en pago
TokuCheckout.on('error', (error) => {
  // error.code, error.message
});

// Usuario abandonó el formulario
TokuCheckout.on('cancel', () => {
  // Redirigir o actualizar UI
});

// Cambio en estado del formulario
TokuCheckout.on('ready', () => {
  // Iframe listo
});
```

---

## 📲 Canal 3: QR Code | Código QR

### Descripción
Genera un código QR que los clientes pueden **escanear con su teléfono** para realizar pagos.

### Ventajas
✅ Sin digitación manual  
✅ Perfecto para papelería/físico  
✅ Implementación muy simple  
✅ Rápido escaneo con cualquier teléfono  
✅ Ideal para boletas/recibos  

### Desventajas
❌ Requiere teléfono del cliente  
❌ No funciona offline  
❌ Problema si código dañado  
❌ Menos datos que otros canales  

### Implementación

#### Generar QR
```bash
GET /qr-code?session_id=sess_123456&size=300
```

#### Ejemplo en HTML
```html
<img src="https://api.toku.com/qr?sessionId=sess_123456&size=300" 
     alt="Código de Pago Toku" />
```

#### Ejemplo en PDF (Boleta)
```python
from reportlab.graphics import renderPDF
from qrcode import QRCode

qr = QRCode(version=1, box_size=10, border=5)
qr.add_data('https://pay.toku.com/checkout/sess_123456')
qr.make(fit=True)
img = qr.make_image()
img.save('boleta_qr.png')
```

### Casos de Uso
- 📄 Facturas/Boletas impresas
- 🛒 Puntos de venta físicos
- 📧 Emails marketing
- 🎫 Entradas/Tickets
- 📋 Documentos diversos

---

## 💬 Canal 4: Mensajería | WhatsApp, SMS, Telegram

### Descripción
Envía enlace de pago directamente a través de **WhatsApp, SMS o Telegram**. Máxima comodidad.

### Ventajas
✅ Contacto directo medio preferido  
✅ Alto engagement (abierto inmediatamente)  
✅ Mejor conversión que email  
✅ Reducción abandono de carro  
✅ Recordatorios automáticos  

### Desventajas
❌ Costo por mensaje  
❌ Regulación y consentimiento  
❌ Límites de frecuencia  
❌ Riesgo de spam  

### Implementación WhatsApp

#### API Integración
```bash
curl -X POST https://api.toku.com/messages/whatsapp \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "+56912345678",
    "body": "Hola, tu pago está listo. Accede aquí: https://pay.toku.com/checkout/sess_123456",
    "type": "payment_link"
  }'
```

#### Best Practices WhatsApp
```
✅ Mensajes cortos y claros
✅ Incluir monto a pagar
✅ Incluir fecha de vencimiento
✅ Call-to-action claro ("Pagar Ahora")
✅ Obtener consentimiento previo
✅ Horarios razonables (8 AM - 8 PM)
✅ No spam (máx 3 recordatorios)
```

#### Mensaje de Ejemplo
```
Hola Juan,

Tienes un pago pendiente de $25.000 vencimiento 31 de Diciembre.

Pagar Ahora: https://pay.toku.com/checkout/sess_123456

¿Preguntas? Contacta a soporte@empresa.cl
```

### Implementación SMS

```bash
curl -X POST https://api.toku.com/messages/sms \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "to": "+56912345678",
    "message": "Paga ahora: https://pay.toku.com/checkout/sess_123456"
  }'
```

---

## 🔧 Canal 5: Mobile WebView | Aplicación Móvil Nativa

### Descripción
Integra Toku **directamente en tu aplicación móvil** nativa (iOS/Android). Máximo control técnico.

### Ventajas
✅ Control total de experiencia  
✅ Integración nativa perfecta  
✅ Retención en tu app  
✅ Mejor performance  
✅ Offline support posible  

### Desventajas
❌ Requiere desarrollo móvil  
❌ Mantenimiento en 2 plataformas  
❌ Mayor complejidad  
❌ Actualizaciones requeridas  

### Implementación iOS (WebView)

```swift
import WebKit

class TokuCheckoutViewController: UIViewController {
    @IBOutlet weak var webView: WKWebView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let sessionId = "sess_123456"
        let url = URL(string: "https://pay.toku.com/checkout/\(sessionId)")!
        
        webView.load(URLRequest(url: url))
        webView.navigationDelegate = self
    }
}

extension TokuCheckoutViewController: WKNavigationDelegate {
    func webView(_ webView: WKWebView, 
                 didFinish navigation: WKNavigation!) {
        // Pago completado
        print("Pago procesado")
    }
}
```

### Implementación Android (WebView)

```kotlin
import android.webkit.WebView
import android.webkit.WebViewClient

class TokuCheckoutActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val webView: WebView = findViewById(R.id.webview)
        webView.webViewClient = WebViewClient()
        
        val sessionId = "sess_123456"
        webView.loadUrl("https://pay.toku.com/checkout/$sessionId")
    }
}
```

### React Native

```javascript
import { WebView } from 'react-native-webview';

export const TokuCheckout = ({ sessionId }) => {
  return (
    <WebView
      source={{ uri: `https://pay.toku.com/checkout/${sessionId}` }}
      onMessage={(event) => {
        if (event.nativeEvent.data === 'success') {
          // Pago completado
        }
      }}
    />
  );
};
```

---

## 🎯 Seleccionar el Canal Correcto

### Para Email Marketing
→ **Web Link** o **QR Code**
- Fácil de insertar
- Compatible con cualquier email

### Para Sitio Web Propio
→ **Iframe** o **Web Link**
- Iframe para mejor UX
- Web Link para setup rápido

### Para Contacto Directo
→ **Mensajería** (WhatsApp/SMS)
- Mayor engagement
- Respuesta inmediata

### Para Papelería
→ **QR Code**
- Boletas/Facturas
- Publicidad/Volantes

### Para App Móvil
→ **WebView** o **Iframe**
- Integración nativa
- Control total

---

## 📊 Comparativa de Conversión

```
Web Link:          ████████░ 80%
Iframe:            ██████████ 95%
QR Code:           ███████░░ 70%
Mensajería (SMS):  █████████░ 90%
Mensajería (WA):   ██████████ 98%
Mobile WebView:    ██████████ 96%
```

---

## 🔒 Consideraciones de Seguridad

✅ Siempre usar HTTPS  
✅ Validar origen de requests  
✅ No guardar datos sensibles en cliente  
✅ Implementar CSRF protection  
✅ Registrar acceso a webhooks  
✅ Monitorear patrones sospechosos  

---

## 📚 Documentación de Referencia

- [API Reference Completa](./api-reference.md)
- [Backend Integration](./backend-integration.md)
- [Glosario](./glosario.md)
- [Documentación Oficial Toku](https://toku.readme.io/docs/frontend-1)

---

**Última actualización**: Documentación 2024  
**Documentación oficial**: https://toku.readme.io/docs/frontend-1
