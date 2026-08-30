# 🔧 KEPENG Universal Interface Builder

Super mudah setup transfer interface apapun **tanpa perlu coding!**

Kamu bisa setup:
- ✅ **Barcode** - Static barcode untuk POS / retail
- ✅ **QR Code** - Dynamic QR untuk setiap transfer
- ✅ **Payment Link** - Shareable link via SMS/email
- ✅ **Webhook** - Auto-notify merchant setiap transaksi
- ✅ **Embed Form** - Embed payment form di website

## Quick Example

### 1️⃣ Setup Interface (One Time)

```bash
curl -X POST https://kepeng.vercel.app/api/v1/interfaces \
  -H "Authorization: Bearer <apiKey>:<secret>" \
  -H "Content-Type: application/json" \
  -d '{
    "interfaceID": "my-qr-payment",
    "type": "qr",
    "config": {
      "size": "medium",
      "includeAmount": true,
      "includeReference": true
    }
  }'
```

Response:
```json
{
  "success": true,
  "data": {
    "id": "my-qr-payment",
    "type": "qr",
    "created": "2026-08-29T...",
    "active": true
  }
}
```

### 2️⃣ Create Transfer

```bash
curl -X POST https://kepeng.vercel.app/api/v1/transfer \
  -H "Authorization: Bearer <apiKey>:<secret>" \
  -H "Content-Type: application/json" \
  -d '{
    "fromAccount": "882233445566",
    "fromBank": "BCA",
    "toAccount": "445566778899",
    "toBank": "GoPay",
    "amount": 500000,
    "currencyFrom": "IDR",
    "currencyTo": "IDR",
    "reference": "Invoice #123"
  }'
```

Response: `{ "transferID": "TXN-2026-001-abc" }`

### 3️⃣ Generate Interface Output

```bash
curl -X POST https://kepeng.vercel.app/api/v1/transfer/TXN-2026-001-abc/interface/my-qr-payment \
  -H "Authorization: Bearer <apiKey>:<secret>"
```

Response:
```json
{
  "success": true,
  "data": {
    "transferID": "TXN-2026-001-abc",
    "interfaceID": "my-qr-payment",
    "interfaceType": "qr",
    "output": {
      "type": "qr",
      "qrURL": "https://api.qrserver.com/v1/create-qr-code/?...",
      "instructions": "Scan QR code untuk bayar"
    }
  }
}
```

### 4️⃣ Embed QR di Website

```html
<img src="[qrURL dari response di atas]" alt="Scan untuk bayar">
```

**Selesai!** 🎉 Customer scan & transfer!

---

## Interface Types Detailed

### 1. Barcode Interface

**Best for**: Retail POS, static invoices, receipts

```bash
curl -X POST /api/v1/interfaces \
  -H "Authorization: Bearer <key>:<secret>" \
  -d '{
    "interfaceID": "retail-pos",
    "type": "barcode",
    "config": {
      "format": "CODE128",        # CODE128, QR, EAN13
      "includeAmount": true,
      "includeReference": true,
      "size": "medium"            # small, medium, large
    }
  }'
```

**Generate output**:
```bash
POST /api/v1/transfer/:id/interface/retail-pos
```

**Response**:
```json
{
  "output": {
    "type": "barcode",
    "format": "CODE128",
    "data": "TXN-2026-001|500000|IDR|Invoice #123|1693324800",
    "renderURL": "https://barcode.tec-it.com/?data=...",
    "instructions": "Scan barcode untuk initiate transfer"
  }
}
```

**Use in HTML**:
```html
<img src="[renderURL]" alt="Barcode untuk scan">
```

---

### 2. QR Code Interface

**Best for**: Mobile payments, restaurant, e-commerce

```bash
curl -X POST /api/v1/interfaces \
  -H "Authorization: Bearer <key>:<secret>" \
  -d '{
    "interfaceID": "mobile-qr",
    "type": "qr",
    "config": {
      "size": "medium",
      "includeAmount": true,
      "includeReference": true,
      "logoUrl": null  # Optional: embed logo di center QR
    }
  }'
```

**Generate output**:
```bash
POST /api/v1/transfer/:id/interface/mobile-qr
```

**Response**:
```json
{
  "output": {
    "type": "qr",
    "qrURL": "https://api.qrserver.com/v1/create-qr-code/?...",
    "instructions": "Scan QR code untuk bayar",
    "metadata": {
      "transferID": "TXN-...",
      "amount": 500000,
      "currency": "IDR"
    }
  }
}
```

**Contoh use case - Restaurant**:
```html
<!-- Taruh di receipt -->
<div class="payment-qr">
  <h3>Scan untuk bayar</h3>
  <img src="[qrURL]" alt="QR Code" width="300">
  <p>Rp 500,000</p>
</div>
```

---

### 3. Payment Link Interface

**Best for**: E-commerce checkout, invoices via email/SMS

```bash
curl -X POST /api/v1/interfaces \
  -H "Authorization: Bearer <key>:<secret>" \
  -d '{
    "interfaceID": "ecom-link",
    "type": "link",
    "config": {
      "baseUrl": "https://yoursite.com",  # Required
      "format": "short",                   # short, full, custom
      "expiresIn": 86400000,               # 24 jam (in ms)
      "includeAmount": true,
      "includeReference": true,
      "customParams": {
        "source": "email",
        "campaign": "summer2026"
      }
    }
  }'
```

**Generate output**:
```bash
POST /api/v1/transfer/:id/interface/ecom-link
```

**Response**:
```json
{
  "output": {
    "type": "link",
    "url": "https://yoursite.com/pay/ABC123XYZ?amount=500000&ref=Invoice%20%23123&source=email&campaign=summer2026",
    "shortURL": "https://kpg.io/abc123",
    "qrFromLink": "https://api.qrserver.com/v1/create-qr-code/?data=https://kpg.io/abc123",
    "expiresAt": "2026-08-30T12:00:00Z",
    "instructions": "Share link atau scan QR untuk bayar"
  }
}
```

**Use in Email**:
```html
<a href="[url]">Bayar sekarang</a>

<!-- Atau kasih dua opsi -->
<p>Link: [shortURL]</p>
<img src="[qrFromLink]" alt="Scan untuk bayar">
```

**Use in SMS**:
```
Bayar Invoice #123 di sini: https://kpg.io/abc123
```

---

### 4. Webhook Interface

**Best for**: Auto-notify merchant, trigger workflow, update database

```bash
curl -X POST /api/v1/interfaces \
  -H "Authorization: Bearer <key>:<secret>" \
  -d '{
    "interfaceID": "webhook-notify",
    "type": "webhook",
    "config": {
      "url": "https://yourapi.com/webhooks/kepeng",  # Required
      "method": "POST",
      "headers": {
        "X-API-Key": "your-secret-key"
      },
      "retryAttempts": 3,
      "retryDelayMs": 1000,
      "events": ["CREATED", "EXECUTING", "COMPLETED", "FAILED"]
    }
  }'
```

**Generate output**:
```bash
POST /api/v1/transfer/:id/interface/webhook-notify
```

**Response**:
```json
{
  "output": {
    "type": "webhook",
    "endpoint": "https://yourapi.com/webhooks/kepeng",
    "method": "POST",
    "headers": {
      "Content-Type": "application/json",
      "X-KEPENG-Signature": "abc123..."
    },
    "payload": {
      "event": "transfer.created",
      "timestamp": "2026-08-29T12:00:00Z",
      "transferID": "TXN-2026-001-abc",
      "amount": 500000,
      "from": { "account": "...", "bank": "BCA" },
      "to": { "account": "...", "bank": "GoPay" }
    },
    "events": ["CREATED", "EXECUTING", "COMPLETED", "FAILED"]
  }
}
```

**KEPENG akan POST ke endpoint kamu dengan:**
- `CREATED` - Transfer baru dibuat
- `EXECUTING` - Sedang diproses
- `COMPLETED` - Berhasil
- `FAILED` - Gagal

**Contoh handler di Node.js**:
```javascript
app.post('/webhooks/kepeng', (req, res) => {
  const { event, transferID, amount } = req.body;

  if (event === 'CREATED') {
    console.log(`Transfer ${transferID} baru dibuat`);
    // Catat di database
  } else if (event === 'COMPLETED') {
    console.log(`Transfer ${transferID} berhasil!`);
    // Update order status
    // Send email ke customer
    // Trigger fulfillment
  } else if (event === 'FAILED') {
    console.log(`Transfer ${transferID} gagal!`);
    // Retry atau notify customer
  }

  res.json({ success: true });
});
```

---

### 5. Embed Form Interface

**Best for**: Website checkout form, embedded payment

```bash
curl -X POST /api/v1/interfaces \
  -H "Authorization: Bearer <key>:<secret>" \
  -d '{
    "interfaceID": "website-embed",
    "type": "embed",
    "config": {
      "theme": "light",                    # light, dark
      "layout": "vertical",                # vertical, horizontal, modal
      "includeHistory": false,
      "redirectOnSuccess": "/thank-you",
      "customCSS": "body { background: #f5f7fa; }"
    }
  }'
```

**Generate output**:
```bash
POST /api/v1/transfer/:id/interface/website-embed
```

**Response**:
```json
{
  "output": {
    "type": "embed",
    "embedID": "kepeng-embed-TXN-2026-001-abc",
    "embedCode": "<div id=\"kepeng-embed-...\"></div>\n<script>...</script>"
  }
}
```

**Use in HTML**:
```html
<!DOCTYPE html>
<html>
<head>
  <title>Checkout - My Store</title>
</head>
<body>
  <h1>Checkout</h1>
  
  <!-- KEPENG Payment Embed -->
  <div id="kepeng-embed-TXN-2026-001-abc"></div>
  <script>
    (function() {
      const container = document.getElementById('kepeng-embed-TXN-2026-001-abc');
      const paymentData = { /* transfer data */ };
      
      const script = document.createElement('script');
      script.src = 'https://kepeng.io/embed/v1.js';
      script.onload = function() {
        window.KepengEmbed.render(container, paymentData, {
          theme: 'light',
          layout: 'vertical',
          redirectOnSuccess: '/thank-you',
          onComplete: function(result) {
            console.log('Payment done!', result);
          }
        });
      };
      document.head.appendChild(script);
    })();
  </script>
</body>
</html>
```

**Form akan show:**
- From account & bank selector
- To account & bank selector
- Amount input
- Currency selector
- Reference input
- Send button

**After complete**: Auto-redirect ke `/thank-you` atau custom URL

---

## Real-World Examples

### Example 1: E-Commerce Checkout

```javascript
// 1. Setup interface (one time)
await fetch('/api/v1/interfaces', {
  method: 'POST',
  headers: { 'Authorization': 'Bearer ...' },
  body: JSON.stringify({
    interfaceID: 'ecom-checkout',
    type: 'link',
    config: { baseUrl: 'https://mystore.com' }
  })
});

// 2. User checkout
const transfer = await fetch('/api/v1/transfer', {
  method: 'POST',
  body: JSON.stringify({
    fromAccount: cartData.buyerPhone,
    fromBank: 'GoPay',
    toAccount: '081234567890',
    toBank: 'Dana',
    amount: cartData.total,
    currencyFrom: 'IDR',
    currencyTo: 'IDR',
    reference: `Order #${cartData.orderId}`
  })
});

// 3. Generate payment link
const paymentLink = await fetch(
  `/api/v1/transfer/${transfer.transferID}/interface/ecom-checkout`,
  { method: 'POST' }
);

// 4. Show to customer
console.log('Pay here:', paymentLink.output.url);
console.log('Or scan:', paymentLink.output.qrFromLink);
```

### Example 2: Restaurant POS

```javascript
// Setup barcode once
createInterface('pos-barcode', 'barcode', { format: 'CODE128' });

// Every bill:
const transfer = createTransfer({ amount: billAmount, ... });
const barcode = generateInterface('pos-barcode', transfer);

// Print di receipt
printReceipt(`
Invoice #${billAmount}
[BARCODE_IMAGE]
Scan untuk bayar
`);
```

### Example 3: Marketplace Auto-Payout

```javascript
// Setup webhook once
createInterface('seller-notify', 'webhook', {
  url: 'https://marketplace.com/webhooks/payout',
  events: ['COMPLETED']
});

// Batch transfer to sellers
for (seller of sellers) {
  const transfer = createTransfer({...seller.data});
  executeTransfer(transfer.id);
  // Webhook auto-notify kamu
}
```

---

## API Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/v1/interfaces` | POST | Create interface |
| `/api/v1/interfaces` | GET | List interfaces |
| `/api/v1/transfer/:id/interface/:interfaceID` | POST | Generate output |

---

## Best Practices

✅ **Setup interfaces one time**, reuse forever
✅ **Use short links** untuk SMS/social media
✅ **Use QR codes** untuk mobile
✅ **Use webhooks** untuk auto-workflow
✅ **Use embed** untuk checkout flow
✅ **Test locally** sebelum production

---

**KEPENG: Super Flexible, Super Simple! 🔌**
