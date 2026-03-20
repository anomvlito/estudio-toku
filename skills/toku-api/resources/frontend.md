# Canales Frontend — Experiencia de Pago del Cliente

Opciones para mostrar el checkout de Toku a tus clientes.

---

## Comparativa Rápida

| Canal | Complejidad | Control UI | Conversión aprox. | Mejor para |
|-------|------------|-----------|-------------------|-----------|
| **Web Link** | Muy baja | Ninguno | ~80% | Emails, notificaciones push |
| **Iframe** | Media | Total | ~95% | Sitio web propio |
| **QR Code** | Baja | Ninguno | ~70% | Papelería, facturas impresas, puntos físicos |
| **WhatsApp** | Media | Mínimo | ~98% | Contacto directo, recuperación de deuda |
| **SMS** | Media | Mínimo | ~90% | Recordatorios masivos |
| **Mobile WebView** | Alta | Total | ~96% | App nativa iOS/Android |

---

## Canal 1: Web Link

La URL de checkout generada por una Checkout Session (Chile) o directamente por Toku.

```
https://pay.toku.com/checkout/[SESSION_ID]
```

**Cuándo usarlo:**
- Envío en emails transaccionales o de recordatorio
- Notificaciones push en apps
- Links en SMS sin necesidad de integración
- Primera integración rápida (no requiere dev frontend)

**Cuándo NO usarlo:**
- Cuando quieres que el usuario permanezca en tu sitio
- Cuando necesitas personalización de marca (branding)

```html
<!-- En un email HTML -->
<a href="https://pay.toku.com/checkout/sess_123456">
  Pagar $50.000
</a>
```

---

## Canal 2: Iframe (JavaScript SDK)

Embebe el formulario de pago dentro de tu sitio sin redirección.

**Cuándo usarlo:**
- Tienes un sitio web y quieres que el cliente no salga
- Necesitas personalizar colores y tipografía
- Quieres mejor conversión que el web link
- Tienes control sobre el HTML de la página

**Cuándo NO usarlo:**
- No tienes HTTPS (obligatorio para iframe)
- Tu sitio es solo para mobile (usar WebView en su lugar)

```html
<div id="toku-container"></div>
<script src="https://sdk.toku.com/js/checkout.js"></script>
<script>
TokuCheckout.createCheckout({
  sessionId: 'sess_123456',
  container: '#toku-container',
  theme: {
    primaryColor: '#007bff',
    fontFamily: 'Segoe UI, sans-serif'
  },
  onSuccess: (result) => {
    // result.transactionId, result.customerId
    window.location.href = '/gracias';
  },
  onError: (error) => console.error(error.code, error.message),
  onCancel: () => window.location.href = '/pago-cancelado',
});
</script>
```

---

## Canal 3: QR Code

Imagen QR que apunta a la URL de checkout.

**Cuándo usarlo:**
- Facturas o boletas impresas
- Puntos de venta físicos
- Material de marketing (volantes, afiches)
- Entornos donde el usuario tiene teléfono pero no puede tipear

**Cuándo NO usarlo:**
- Cuando no puedes garantizar que el cliente tenga teléfono con cámara
- Entornos 100% digitales (hay opciones con mejor UX)

```html
<!-- Imagen QR generada por Toku -->
<img src="https://api.trytoku.com/qr?sessionId=sess_123456&size=300"
     alt="Código QR de pago" />
```

```python
# Generar QR en PDF (Python)
from qrcode import QRCode

qr = QRCode(version=1, box_size=10, border=5)
qr.add_data('https://pay.toku.com/checkout/sess_123456')
qr.make(fit=True)
img = qr.make_image()
img.save('boleta_qr.png')
```

---

## Canal 4: Mensajería (WhatsApp / SMS)

Envío directo del link de pago al teléfono del cliente.

**Cuándo usarlo:**
- Recuperación de deuda vencida (mayor urgencia)
- Clientes que prefieren WhatsApp sobre email
- Recordatorios automatizados de vencimiento
- Cuando el email tiene baja tasa de apertura

**Cuándo NO usarlo:**
- Sin consentimiento previo del cliente (riesgo legal)
- Si no tienes el teléfono verificado del cliente
- Para envíos masivos sin segmentación (riesgo de spam)

```bash
# Enviar por WhatsApp
curl -X POST https://api.trytoku.com/messages/whatsapp \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "+56912345678",
    "body": "Hola Juan, tienes un pago pendiente de $50.000 venc. 31 dic. Pagar aquí: https://pay.toku.com/checkout/sess_123456",
    "type": "payment_link"
  }'

# Enviar por SMS
curl -X POST https://api.trytoku.com/messages/sms \
  -H "Authorization: Bearer $API_KEY" \
  -d '{"to": "+56912345678", "message": "Pago pendiente $50.000: https://pay.toku.com/checkout/sess_123456"}'
```

**Reglas de mensajería:**
- Horario: 8:00 AM – 8:00 PM
- Máximo 3 recordatorios por deuda
- Incluir siempre monto y fecha de vencimiento
- Obtener consentimiento previo del cliente

---

## Canal 5: Mobile WebView

Toku embebido en app nativa iOS/Android via WebView.

**Cuándo usarlo:**
- Tienes una app nativa y quieres que el cliente pague sin salir de ella
- Necesitas control total de la experiencia en móvil
- Quieres capturar eventos de éxito/error desde la app nativa

**Cuándo NO usarlo:**
- No tienes una app nativa (usar Iframe en web)
- No tienes recursos de desarrollo móvil

```swift
// iOS (Swift/WKWebView)
import WebKit

class TokuCheckoutViewController: UIViewController {
    let webView = WKWebView()

    override func viewDidLoad() {
        super.viewDidLoad()
        let url = URL(string: "https://pay.toku.com/checkout/sess_123456")!
        webView.load(URLRequest(url: url))
        webView.navigationDelegate = self
    }
}
```

```kotlin
// Android (Kotlin/WebView)
val webView: WebView = findViewById(R.id.webview)
webView.webViewClient = WebViewClient()
webView.loadUrl("https://pay.toku.com/checkout/sess_123456")
```

```javascript
// React Native
import { WebView } from 'react-native-webview';

const TokuCheckout = ({ sessionId }) => (
  <WebView
    source={{ uri: `https://pay.toku.com/checkout/${sessionId}` }}
    onMessage={(e) => {
      if (e.nativeEvent.data === 'success') {
        // pago completado — navegar a pantalla de confirmación
      }
    }}
  />
);
```

---

## Guía de Selección Rápida

```
¿El cliente está en email/notificación?
  └→ Web Link o QR Code

¿Estás en un sitio web y quieres máxima conversión?
  └→ Iframe

¿Necesitas recuperar una deuda urgente?
  └→ WhatsApp (mayor engagement)

¿El cliente está en tu app móvil?
  └→ WebView

¿Va en documentos físicos (facturas, boletas)?
  └→ QR Code
```
