# Card Payment API — Telegram Stars Purchase

Documentation for purchasing Telegram Stars using a bank card in EUR or USD.

**Supported currencies:** EUR, USD  
**Minimum payment amount:** 5 EUR / 5 USD  
**Max Stars per transaction:** 10,000

## Base URL
`https://api.nexpay.pro`

---

## Endpoints

### 1) Get Stars price

Returns the current price for the requested amount of Stars in EUR and USD.

```
GET /price?amount={stars_amount}
```

**Query parameters:**
- `amount` (optional) — number of Stars (default: 1; min: 50; max: 10,000)

**Response:**
```json
{
  "eur": 15.50,
  "usd": 17.00,
  "valid": true
}
```

**Response fields:**
- `eur` — price in EUR for the requested Stars amount  
- `usd` — price in USD for the requested Stars amount  
- `valid` — whether the amount is within the allowed range (50–10,000)

---

### 2) Create a deposit

Creates a new order to purchase Stars and returns a deposit identifier (used to check status and proceed with payment).

```
POST /deposit
```

**Request body:**
```json
{
  "username": "john_doe",
  "user_id": "123456789",
  "amount": 15.50,
  "stars_amount": 1000,
  "currency": "EUR"
}
```

**Fields:**
- `username` (required) — Telegram username (without @)
- `user_id` (optional) — Telegram user ID, if available
- `amount` (required) — payment amount in the selected currency
- `stars_amount` (required) — Stars amount (50–10,000)
- `currency` (required) — `"EUR"` or `"USD"`

**Response:**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response fields:**
- `orderId` — unique deposit identifier

**Errors:**
- `400` — invalid `amount`, `stars_amount`, or `currency`
- `403` — card payments are not available in your region
- `429` — rate limit exceeded or access temporarily restricted

---

### 3) Process card payment

Sends card details to perform the payment. Typically called after the user completes the payment form.

```
POST /card/process-payment
```

**Request body:**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "cardDetails": {
    "card_number": "4111111111111111",
    "card_expiry": "12/25",
    "card_cvv": "123",
    "card_holder_name": "John Doe",
    "card_type": "visa"
  },
  "customerDetails": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "john@example.com",
    "phone": "+1234567890",
    "address": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zip": "10001",
    "country": "US"
  }
}
```

**cardDetails fields:**
- `card_number` (required) — card number (16 digits; spaces allowed)
- `card_expiry` (required) — expiry date in `MM/YY` format
- `card_cvv` (required) — CVV/CVC (3–4 digits)
- `card_holder_name` (required) — cardholder name
- `card_type` (optional) — `"visa"`, `"mastercard"`, `"amex"` (may be detected automatically)

**customerDetails fields:**
- all fields are required
- `phone` — international format (e.g., +1234567890)
- `country` — 2-letter ISO country code (e.g., US, GB, DE)
- `state` — state/region code
- `zip` — postal/ZIP code

**Response:**
```json
{
  "status": "success",
  "paymentId": "pay_1234567890",
  "redirectUrl": "https://example.com/3ds/redirect"
}
```

**Response fields:**
- `status` — operation status (e.g., `success` or `pending`)
- `paymentId` — payment identifier
- `redirectUrl` — 3DS verification URL (if required)

---

### 4) Check deposit status

Lets you retrieve the current order state (created, paid, declined, etc.).

```
GET /deposit/{orderId}
```

**Path parameters:**
- `orderId` (required) — deposit identifier

**Response:**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "paid",
  "stars_amount": 1000,
  "currency": "EUR",
  "amount": 15.50
}
```

**Response fields:**
- `status` — current deposit status (e.g., `created`, `pending`, `paid`, `failed`)
- the remaining fields mirror the order parameters
