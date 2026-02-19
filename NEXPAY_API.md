# Card Payment API - Telegram Stars Purchase

Documentation for purchasing Telegram Stars using EUR or USD card payments.

**Supported Currencies:** EUR, USD
**Minimum Payment:** 5 EUR / 5 USD
**Maximum Stars per Transaction:** 10,000 stars

## Base URL
`https://nexpay.pro/api`

---

## Endpoints

### 1. Get Stars Prices

Get the current price for a specific amount of stars in both EUR and USD.

```
GET /price?amount={stars_amount}
```

**Query Parameters:**
- `amount` (optional) - Number of stars (default: 1, min: 50, max: 10,000)

**Response:**
```json
{
  "eur": 15.50,
  "usd": 17.00,
  "valid": true
}
```

**Response Fields:**
- `eur` - Price in EUR for the requested amount of stars
- `usd` - Price in USD for the requested amount of stars
- `valid` - Whether the requested amount is within acceptable range (50-10,000)

---

### 2. Create Deposit

Create a new stars purchase deposit. Returns a URL where the user can enter card details.

```
POST /deposit
```

**Request Body:**
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
- `username` (required) - Telegram username (without @)
- `user_id` (optional) - Telegram user ID if known
- `amount` (required) - Payment amount in selected currency
- `stars_amount` (required) - Number of stars to purchase (50-10,000)
- `currency` (required) - Payment currency: `"EUR"` or `"USD"`

**Response:**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response Fields:**
- `orderId` - Unique identifier for this deposit (used to check status and process payment)

**Error Responses:**
- `400` - Invalid amount, stars_amount, or currency
- `403` - Card payments not available in your region
- `429` - Rate limit exceeded or access denied

---

### 3. Process Card Payment

Submit card details to process the payment. Called after user enters card information on the payment page.

```
POST /card/process-payment
```

**Request Body:**
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

**Card Details Fields:**
- `card_number` (required) - Card number (16 digits, spaces allowed)
- `card_expiry` (required) - Expiry date in MM/YY format
- `card_cvv` (required) - CVV code (3-4 digits)
- `card_holder_name` (required) - Name on card
- `card_type` (optional) - Card type: `"visa"`, `"mastercard"`, `"amex"` (auto-detected if not provided)

**Customer Details Fields:**
- All fields are required
- `phone` - International format (e.g., +1234567890)
- `country` - 2-letter ISO country code (e.g., US, GB, DE)
- `state` - State/province code
- `zip` - Postal/ZIP code

**Response:**
```json
{
  "status": "success",
  "message": "Payment submitted successfully",
  "paymentId": "qnt_xyz123",
  "acs_url": "https://3ds-verification-url.com"
}
```

**Response Fields:**
- `status` - Payment status: `"success"`, `"pending"`, or `"failed"`
- `message` - Human-readable status message
- `paymentId` - Payment provider's transaction ID
- `acs_url` (optional) - URL for additional payment verification (redirect user here if present)

**Error Responses:**
- `400` - Invalid card details or customer information
- `403` - Access denied (IP blacklisted or region not supported)
- `404` - Deposit not found or already processed
- `500` - Payment processing failed

---

### 4. Get Payment Status

Check the current status of a payment.

```
GET /card/payment/:orderId
```

**Response:**
```json
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "success",
  "amount": 15.50,
  "stars_amount": 1000,
  "username": "john_doe",
  "timestamp": 1234567890,
  "currency": "EUR"
}
```

**Response Fields:**
- `orderId` - Unique identifier for this deposit
- `status` - Current payment status (see values below)
- `amount` - Payment amount in the selected currency
- `stars_amount` - Number of stars purchased
- `username` - Telegram username
- `timestamp` - Unix timestamp when deposit was created
- `currency` - Payment currency (EUR or USD)

**Status Values:**
- `null` - Awaiting payment verification (initial state and while processing card payment)
- `"in_progress"` - Payment verified, processing stars delivery
- `"success"` - Payment complete, stars delivered
- `"error"` - Payment failed, cancelled, or refunded

---

## Payment Flow

1. **Get Price** - `GET /price?amount=1000` to show user the current price
2. **Create Deposit** - `POST /deposit` with selected currency and amount
3. **Show Payment Form** - Direct user to the returned URL
4. **Submit Card Details** - `POST /card/process-payment` with card information
5. **Handle Verification** - If `acs_url` is present in response, redirect user to that URL
6. **Poll Status** - Check `GET /card/payment/:orderId` until status is `"success"` or `"error"`
7. **Complete** - Stars are automatically delivered to the user's Telegram account

---
