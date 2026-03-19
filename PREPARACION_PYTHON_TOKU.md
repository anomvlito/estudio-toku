# Preparación Python para Entrevista Toku
> Foco: Python/FastAPI, integraciones, pagos recurrentes
> Basado en las ofertas de 2026 (SE, Senior SE, Integrations Engineer)

---

## Prioridad de temas (ordenados por probabilidad que pregunten)

1. **FastAPI** — es su stack actual, preguntarán seguro
2. **async/await** — FastAPI lo requiere, lo evaluarán
3. **Idempotencia y retries** — core del negocio de pagos
4. **pytest + mocking** — mencionado explícitamente en ofertas
5. **Pydantic** — viene con FastAPI, infaltable
6. **Flask vs FastAPI trade-offs** — su pregunta clásica de trade-offs
7. **JWT / OAuth 2.0** — mencionado en Integrations Engineer
8. **PostgreSQL + SQLAlchemy** — su DB principal

---

## 1. FastAPI — Lo que más importa

### Estructura básica de un endpoint

```python
from fastapi import FastAPI, Depends, HTTPException, status
from pydantic import BaseModel

app = FastAPI()

class PaymentCreate(BaseModel):
    amount: int          # en centavos, nunca float para plata
    currency: str = "CLP"
    mandate_id: str
    idempotency_key: str  # crítico en pagos

class PaymentResponse(BaseModel):
    id: str
    status: str
    amount: int

@app.post("/payments", response_model=PaymentResponse, status_code=status.HTTP_201_CREATED)
async def create_payment(payment: PaymentCreate, db: Session = Depends(get_db)):
    # validar idempotency_key primero
    existing = await db.get_payment_by_idempotency_key(payment.idempotency_key)
    if existing:
        return existing  # retornar el mismo resultado, no crear duplicado

    result = await process_payment(payment)
    return result
```

**Por qué FastAPI sobre Flask:**
- Validación automática con Pydantic (no escribes `if not data.get('amount'): raise...`)
- Documentación OpenAPI automática en `/docs`
- Async nativo (Flask es sync, necesitas plugins adicionales)
- Type hints = menos bugs en runtime
- Inyección de dependencias built-in

---

## 2. async/await en Python — Conceptos clave

### ¿Qué es async y cuándo importa?

```python
import asyncio
import httpx

# SYNC — bloquea el thread mientras espera respuesta del banco
def get_bank_status_sync(account_id: str):
    response = requests.get(f"https://bank-api.com/accounts/{account_id}")
    return response.json()

# ASYNC — libera el thread mientras espera, puede manejar otras requests
async def get_bank_status_async(account_id: str):
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://bank-api.com/accounts/{account_id}")
        return response.json()

# Ejecutar múltiples llamadas en paralelo (clave para integraciones)
async def check_multiple_accounts(account_ids: list[str]):
    async with httpx.AsyncClient() as client:
        tasks = [client.get(f"https://bank-api.com/accounts/{aid}") for aid in account_ids]
        results = await asyncio.gather(*tasks)  # ejecuta todos en paralelo
    return [r.json() for r in results]
```

### Errores comunes que preguntan en entrevistas

```python
# MAL: mezclar sync I/O en código async — bloquea el event loop
async def bad_example():
    time.sleep(5)           # bloquea todo
    requests.get(url)       # bloquea todo

# BIEN: usar alternativas async
async def good_example():
    await asyncio.sleep(5)  # cede control al event loop
    async with httpx.AsyncClient() as c:
        await c.get(url)    # no bloquea

# MAL: olvidar await
async def forget_await():
    result = get_bank_status_async("123")  # retorna coroutine, no el resultado!

# BIEN:
async def correct():
    result = await get_bank_status_async("123")
```

### Pregunta trampa: ¿async siempre es mejor?

**No.** Async es mejor cuando el cuello de botella es I/O (red, disco, BD). Para CPU-intensive (cálculos pesados, procesamiento de datos), async no ayuda — necesitas multiprocessing.

---

## 3. Idempotencia y Retries — Corazón del negocio de pagos

Este es el tema MÁS importante para Toku. Un cobro duplicado es un desastre.

### Idempotencia

```python
from fastapi import FastAPI, Header
from pydantic import BaseModel

class ChargeRequest(BaseModel):
    amount: int
    mandate_id: str

@app.post("/charges")
async def create_charge(
    request: ChargeRequest,
    idempotency_key: str = Header(...),  # requerir header explícito
    db: Session = Depends(get_db)
):
    # 1. Buscar si ya procesamos esta request
    existing = await db.query(Charge).filter(
        Charge.idempotency_key == idempotency_key
    ).first()

    if existing:
        # Retornar EXACTAMENTE el mismo resultado de antes
        return existing.to_response()

    # 2. Crear registro en "pending" ANTES de llamar al banco
    #    Si el proceso muere, se puede recuperar por reconciliación
    charge = await db.create_charge(request, idempotency_key, status="pending")

    # 3. Llamar al banco
    try:
        bank_response = await bank_client.debit(request.amount, request.mandate_id)
        await db.update_charge(charge.id, status="succeeded")
        return charge.to_response()
    except BankError as e:
        status = "failed" if not e.retryable else "failed_retryable"
        await db.update_charge(charge.id, status=status)
        raise
```

### Retries con backoff exponencial

```python
import asyncio
import random

async def retry_with_backoff(func, max_retries: int = 3, base_delay: float = 1.0):
    """
    Delay: 1s, 2s, 4s... con jitter para evitar thundering herd
    """
    for attempt in range(max_retries):
        try:
            return await func()
        except TransientError:
            if attempt == max_retries - 1:
                raise  # último intento, propagar

            delay = base_delay * (2 ** attempt)
            jitter = delay * 0.1 * (random.random() * 2 - 1)
            await asyncio.sleep(delay + jitter)
```

### Qué se reintenta vs qué no

```python
# Errores que NO se reintentan (definitivos):
NON_RETRYABLE = {"MANDATE_CANCELLED", "INVALID_ACCOUNT", "UNAUTHORIZED"}

# Errores que SÍ se reintentan (transitorios):
RETRYABLE = {"TIMEOUT", "BANK_UNAVAILABLE", "RATE_LIMITED"}
```

---

## 4. pytest — Testing de integraciones

### conftest.py típico de FastAPI

```python
import pytest
import pytest_asyncio
from httpx import AsyncClient

@pytest_asyncio.fixture
async def async_client():
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client
```

### Mocking de APIs bancarias (lo que más preguntarán)

```python
import pytest
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_charge_success():
    with patch("app.clients.bank.BankClient.debit", new_callable=AsyncMock) as mock_debit:
        mock_debit.return_value = {"transaction_id": "TXN-123", "status": "approved"}

        result = await payment_service.create_charge(
            amount=10000,
            mandate_id="MANDATE-456",
            idempotency_key="IDEM-789"
        )

    assert result.status == "succeeded"
    mock_debit.assert_called_once()

@pytest.mark.asyncio
async def test_charge_idempotency():
    """El mismo idempotency_key no debe crear dos cobros"""
    with patch("app.clients.bank.BankClient.debit", new_callable=AsyncMock) as mock_debit:
        mock_debit.return_value = {"transaction_id": "TXN-1", "status": "approved"}

        result1 = await payment_service.create_charge(amount=10000, idempotency_key="KEY-001")
        result2 = await payment_service.create_charge(amount=10000, idempotency_key="KEY-001")

    assert mock_debit.call_count == 1  # solo se llamó al banco una vez
    assert result1.id == result2.id

@pytest.mark.asyncio
async def test_charge_retries_on_timeout():
    with patch("app.clients.bank.BankClient.debit", new_callable=AsyncMock) as mock_debit:
        # Primeros 2 intentos fallan, tercero ok
        mock_debit.side_effect = [
            BankTimeoutError(),
            BankTimeoutError(),
            {"transaction_id": "TXN-1", "status": "approved"}
        ]

        result = await payment_service.create_charge(amount=10000)

    assert result.status == "succeeded"
    assert mock_debit.call_count == 3
```

---

## 5. Pydantic — Validación de datos

```python
from pydantic import BaseModel, Field, EmailStr, field_validator
from enum import Enum

class Currency(str, Enum):
    CLP = "CLP"
    MXN = "MXN"
    BRL = "BRL"

class MandateCreate(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)
    account_number: str
    currency: Currency = Currency.CLP
    max_amount: int = Field(..., gt=0, description="Monto máximo en centavos")

    @field_validator('account_number')
    @classmethod
    def validate_account(cls, v):
        if not v.isdigit():
            raise ValueError("account_number debe contener solo dígitos")
        return v
```

### Nunca uses float para dinero

```python
# MAL — floating point errors:
price = 0.1 + 0.2  # = 0.30000000000000004

# BIEN — guardar en centavos (int):
price_in_cents = 1000  # = $10.00

# O usar Decimal para cálculos:
from decimal import Decimal
price = Decimal("0.10") + Decimal("0.20")  # = 0.30 exacto
```

---

## 6. Flask vs FastAPI — La pregunta de trade-offs

| Dimensión | Flask | FastAPI |
|-----------|-------|---------|
| Validación de datos | Manual (marshmallow) | Automática (Pydantic) |
| Async | Con esfuerzo (Flask 2.0+) | Nativo |
| Documentación API | Plugin externo | Automática `/docs` |
| Performance | Menor (WSGI) | Mayor (ASGI) |
| Ecosistema | Más maduro | Creciendo rápido |
| Cuándo usarlo | Legacy, prototipos simples | APIs nuevas, alta concurrencia |

**Cómo responder en entrevista:**
> "Flask es excelente donde el equipo ya lo conoce. Para un servicio de pagos elegiría FastAPI: la validación con Pydantic reduce bugs en producción, el async nativo maneja mejor llamadas concurrentes a APIs bancarias, y la documentación automática facilita integraciones con clientes."

---

## 7. JWT y OAuth 2.0

### JWT — implementación básica

```python
import jwt
from datetime import datetime, timedelta
from fastapi import HTTPException, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

SECRET_KEY = "your-secret"  # en producción: variable de entorno
ALGORITHM = "HS256"

def create_access_token(user_id: str, expires_in_minutes: int = 30) -> str:
    payload = {
        "sub": user_id,
        "iat": datetime.utcnow(),
        "exp": datetime.utcnow() + timedelta(minutes=expires_in_minutes),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expirado")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Token inválido")

security = HTTPBearer()

async def get_current_user(credentials: HTTPAuthorizationCredentials = Security(security)):
    payload = verify_token(credentials.credentials)
    return payload["sub"]
```

### OAuth 2.0 Client Credentials (para integraciones B2B con bancos)

```
Toku ──── POST /oauth/token {client_id, client_secret} ────> Banco
Toku <─── {access_token, expires_in} ─────────────────────── Banco
Toku ──── GET /debit  Authorization: Bearer <token> ────────> Banco
Toku <─── {transaction_id, status} ─────────────────────────  Banco
```

```python
import httpx
from datetime import datetime, timedelta

class BankOAuthClient:
    def __init__(self, client_id: str, client_secret: str, token_url: str):
        self.client_id = client_id
        self.client_secret = client_secret
        self.token_url = token_url
        self._token: str | None = None
        self._token_expires_at: datetime | None = None

    async def get_token(self) -> str:
        # Reusar token si sigue vigente
        if self._token and self._token_expires_at > datetime.utcnow():
            return self._token

        async with httpx.AsyncClient() as client:
            response = await client.post(
                self.token_url,
                data={"grant_type": "client_credentials",
                      "client_id": self.client_id,
                      "client_secret": self.client_secret}
            )
            response.raise_for_status()
            data = response.json()

        self._token = data["access_token"]
        self._token_expires_at = datetime.utcnow() + timedelta(seconds=data["expires_in"] - 30)
        return self._token
```

---

## 8. PostgreSQL + SQLAlchemy async

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from sqlalchemy import String, Integer, DateTime, func
import uuid

class Charge(Base):
    __tablename__ = "charges"

    id: Mapped[str] = mapped_column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    mandate_id: Mapped[str] = mapped_column(String, nullable=False)
    amount: Mapped[int] = mapped_column(Integer, nullable=False)
    status: Mapped[str] = mapped_column(String, default="pending")
    idempotency_key: Mapped[str] = mapped_column(String, unique=True, nullable=False)
    created_at: Mapped[datetime] = mapped_column(DateTime, server_default=func.now())

# Dependency para FastAPI
async def get_db():
    async with AsyncSession(engine) as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

### SELECT FOR UPDATE SKIP LOCKED — patrón de worker

```python
from sqlalchemy import select, and_

async def get_pending_charges(db: AsyncSession, limit: int = 100):
    """Patrón para workers que procesan colas: cada worker agarra registros distintos"""
    stmt = (
        select(Charge)
        .where(Charge.status == "pending")
        .order_by(Charge.created_at.asc())
        .limit(limit)
        .with_for_update(skip_locked=True)  # evita que dos workers agarren el mismo registro
    )
    result = await db.execute(stmt)
    return result.scalars().all()
```

---

## 9. Webhooks — Diseño resiliente

```python
from fastapi import BackgroundTasks
import httpx, hmac, hashlib, json

@app.post("/internal/charges/{charge_id}/notify")
async def trigger_webhook(charge_id: str, background_tasks: BackgroundTasks):
    charge = await get_charge(charge_id)
    background_tasks.add_task(deliver_webhook, charge)
    return {"status": "queued"}

async def deliver_webhook(charge, max_retries: int = 5):
    payload = {
        "event": "charge.succeeded",
        "data": {"id": charge.id, "amount": charge.amount, "status": charge.status},
        "timestamp": datetime.utcnow().isoformat(),
    }

    for attempt in range(max_retries):
        try:
            async with httpx.AsyncClient(timeout=10.0) as client:
                response = await client.post(
                    charge.webhook_url,
                    json=payload,
                    headers={
                        "X-Toku-Signature": compute_hmac_signature(payload),
                        "Content-Type": "application/json",
                    }
                )
                if response.status_code < 500:  # 2xx o 4xx = definitivo, no reintentar
                    return
        except httpx.TimeoutException:
            pass

        await asyncio.sleep(2 ** attempt)  # backoff exponencial

def compute_hmac_signature(payload: dict) -> str:
    """HMAC para que el cliente verifique autenticidad del webhook"""
    body = json.dumps(payload, sort_keys=True).encode()
    return hmac.new(WEBHOOK_SECRET.encode(), body, hashlib.sha256).hexdigest()
```

---

## Checklist de repaso rápido (para el día antes)

- [ ] ¿Qué es async y cuándo usarlo? (I/O vs CPU)
- [ ] ¿Por qué FastAPI > Flask para APIs modernas?
- [ ] ¿Qué es idempotencia? ¿Cómo la implementarías?
- [ ] ¿Por qué crear el registro en BD *antes* de llamar al banco?
- [ ] ¿Cómo harías retry con backoff exponencial?
- [ ] ¿Cómo mockeas una API externa en pytest?
- [ ] ¿Por qué nunca float para dinero?
- [ ] ¿Cómo funciona JWT? ¿Cuándo usarías OAuth 2.0?
- [ ] ¿Qué es `SELECT FOR UPDATE SKIP LOCKED`?
- [ ] ¿Cuándo MongoDB > PostgreSQL? (esquema variable por regulación de cada país)

---

## Las 3 preguntas más probables en la entrevista

**1. "¿Cómo diseñarías un sistema de cobros recurrentes que garantice que no se cobra dos veces?"**
→ idempotency_key + crear registro en pending antes del cobro + reconciliación de pendientes viejos

**2. "¿Cómo estructurarías un webhook resiliente?"**
→ background task, retries con backoff, firma HMAC para verificar autenticidad, registrar intentos en BD

**3. "¿Cuándo usarías PostgreSQL vs MongoDB?"**
→ PG para datos estructurados y transacciones ACID (cobros, mandatos). Mongo para esquema variable — en Toku: formatos regulatorios distintos por país (PIX Brasil, SPEI México, PAC Chile)

---

*Preparación para entrevista Toku — marzo 2026*
