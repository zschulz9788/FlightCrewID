# Flight Crew ID - Payment Integration Guide

## Overview
This guide explains how to implement secure payment processing for your Flight Crew ID application system.

## Payment Options

### 1. STRIPE (Recommended) ⭐
**Why choose Stripe:**
- Industry standard for online payments
- PCI compliant (handles all security)
- Accepts all major cards (Visa, Mastercard, Amex, Discover)
- Built-in fraud detection
- Easy integration
- Great documentation
- Supports recurring billing (for renewals)

**Setup Steps:**
1. Create account at https://stripe.com
2. Get your API keys (Publishable & Secret)
3. Replace `pk_test_YOUR_PUBLISHABLE_KEY_HERE` in the HTML with your actual key
4. Set up webhook endpoint for payment confirmations

**Pricing:**
- 2.9% + $0.30 per successful transaction
- No monthly fees
- No setup fees

### 2. PayPal
**Why choose PayPal:**
- Widely recognized and trusted
- Customers can pay with PayPal balance or card
- Good for international payments
- Guest checkout available

**Pricing:**
- 2.99% + $0.49 per transaction
- Slightly higher than Stripe

### 3. Square
**Why choose Square:**
- Simple setup
- Good for small businesses
- All-in-one payment solution

**Pricing:**
- 2.9% + $0.30 per transaction

## Implementation Steps

### FRONTEND (What's already in the HTML file)

```javascript
// 1. Initialize Stripe with your publishable key
const stripe = Stripe('pk_live_YOUR_PUBLISHABLE_KEY');

// 2. Create card element
const elements = stripe.elements();
const cardElement = elements.create('card');
cardElement.mount('#card-element');

// 3. Create payment method
const {paymentMethod, error} = await stripe.createPaymentMethod({
    type: 'card',
    card: cardElement,
    billing_details: {
        name: customerName,
        email: customerEmail,
        address: billingAddress
    }
});

// 4. Send payment method ID to your server
const response = await fetch('/process-payment', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({
        paymentMethodId: paymentMethod.id,
        amount: totalAmount,
        customerData: formData
    })
});
```

### BACKEND (Server-side - Required for production)

You'll need a backend server to securely process payments. Here's an example using Node.js:

#### Option A: Node.js/Express Backend

```javascript
// server.js
const express = require('express');
const stripe = require('stripe')('sk_live_YOUR_SECRET_KEY');
const app = express();

app.use(express.json());

app.post('/process-payment', async (req, res) => {
    try {
        const { paymentMethodId, amount, customerData } = req.body;
        
        // Create a PaymentIntent
        const paymentIntent = await stripe.paymentIntents.create({
            amount: amount * 100, // Convert to cents
            currency: 'usd',
            payment_method: paymentMethodId,
            confirm: true,
            description: `Flight Crew ID - ${customerData.idDuration}`,
            receipt_email: customerData.email,
            metadata: {
                customer_name: customerData.name,
                id_type: customerData.idDuration
            }
        });

        if (paymentIntent.status === 'succeeded') {
            // 1. Save order to database
            await saveOrderToDatabase(customerData, paymentIntent.id);
            
            // 2. Send confirmation email
            await sendConfirmationEmail(customerData.email);
            
            // 3. Return success
            res.json({ 
                success: true, 
                orderId: paymentIntent.id 
            });
        }
    } catch (error) {
        console.error('Payment error:', error);
        res.status(400).json({ 
            success: false, 
            error: error.message 
        });
    }
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

#### Option B: PHP Backend

```php
<?php
require_once('vendor/autoload.php');

\Stripe\Stripe::setApiKey('sk_live_YOUR_SECRET_KEY');

$input = json_decode(file_get_contents('php://input'), true);

try {
    $paymentIntent = \Stripe\PaymentIntent::create([
        'amount' => $input['amount'] * 100,
        'currency' => 'usd',
        'payment_method' => $input['paymentMethodId'],
        'confirm' => true,
        'description' => 'Flight Crew ID - ' . $input['customerData']['idDuration'],
        'receipt_email' => $input['customerData']['email']
    ]);

    if ($paymentIntent->status === 'succeeded') {
        // Save to database
        saveOrder($input['customerData'], $paymentIntent->id);
        
        // Send email
        sendConfirmationEmail($input['customerData']['email']);
        
        echo json_encode(['success' => true, 'orderId' => $paymentIntent->id]);
    }
} catch (\Stripe\Exception\CardException $e) {
    http_response_code(400);
    echo json_encode(['success' => false, 'error' => $e->getMessage()]);
}
?>
```

#### Option C: Python/Flask Backend

```python
import stripe
from flask import Flask, request, jsonify

app = Flask(__name__)
stripe.api_key = 'sk_live_YOUR_SECRET_KEY'

@app.route('/process-payment', methods=['POST'])
def process_payment():
    try:
        data = request.get_json()
        
        payment_intent = stripe.PaymentIntent.create(
            amount=int(data['amount'] * 100),
            currency='usd',
            payment_method=data['paymentMethodId'],
            confirm=True,
            description=f"Flight Crew ID - {data['customerData']['idDuration']}",
            receipt_email=data['customerData']['email']
        )
        
        if payment_intent.status == 'succeeded':
            # Save order to database
            save_order(data['customerData'], payment_intent.id)
            
            # Send confirmation email
            send_confirmation_email(data['customerData']['email'])
            
            return jsonify({'success': True, 'orderId': payment_intent.id})
            
    except stripe.error.CardError as e:
        return jsonify({'success': False, 'error': str(e)}), 400

if __name__ == '__main__':
    app.run(port=3000)
```

## Database Schema

### Orders Table
```sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_number VARCHAR(50) UNIQUE,
    stripe_payment_id VARCHAR(100),
    
    -- Customer Info
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(255),
    phone VARCHAR(20),
    
    -- Order Details
    id_duration VARCHAR(20), -- 'one-year' or 'two-year'
    base_price DECIMAL(10,2),
    addon_lanyard INT DEFAULT 0,
    addon_protector INT DEFAULT 0,
    addon_luggage INT DEFAULT 0,
    addon_backup INT DEFAULT 0,
    shipping_cost DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    
    -- Documents
    document_type VARCHAR(50),
    document_files TEXT, -- JSON array of file paths
    
    -- Status
    status VARCHAR(20) DEFAULT 'pending', -- pending, processing, completed, shipped
    payment_status VARCHAR(20) DEFAULT 'pending',
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Users Table
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    password_hash VARCHAR(255),
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## File Upload Handling

### Backend File Upload (Node.js with Multer)

```javascript
const multer = require('multer');
const path = require('path');

// Configure storage
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/documents/');
    },
    filename: (req, file, cb) => {
        const uniqueName = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, uniqueName + path.extname(file.originalname));
    }
});

// File filter
const fileFilter = (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|pdf/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (extname && mimetype) {
        cb(null, true);
    } else {
        cb(new Error('Only .png, .jpg, .jpeg, and .pdf files are allowed'));
    }
};

const upload = multer({
    storage: storage,
    limits: { fileSize: 5 * 1024 * 1024 }, // 5MB limit
    fileFilter: fileFilter
});

// Upload endpoint
app.post('/upload-documents', upload.array('documents', 5), (req, res) => {
    const filePaths = req.files.map(file => file.path);
    res.json({ success: true, files: filePaths });
});
```

## Email Notifications

### Using SendGrid (Recommended)

```javascript
const sgMail = require('@sendgrid/mail');
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

async function sendConfirmationEmail(customerEmail, orderData) {
    const msg = {
        to: customerEmail,
        from: 'orders@flightcrewid.com',
        subject: 'Flight Crew ID Order Confirmation',
        html: `
            <h2>Thank you for your order!</h2>
            <p>Your order #${orderData.orderNumber} has been received.</p>
            <h3>Order Details:</h3>
            <ul>
                <li>ID Type: ${orderData.idDuration}</li>
                <li>Total: $${orderData.total}</li>
            </ul>
            <p>Your ID will be processed and shipped within 5-7 business days.</p>
        `
    };
    
    await sgMail.send(msg);
}
```

## Security Best Practices

### 1. Never expose secret keys in frontend
```javascript
// ❌ WRONG - Never do this
const stripe = Stripe('sk_live_SECRET_KEY'); // Secret key exposed!

// ✅ CORRECT - Only use publishable key
const stripe = Stripe('pk_live_PUBLISHABLE_KEY');
```

### 2. Validate on server-side
```javascript
// Always verify amounts on server
app.post('/process-payment', async (req, res) => {
    const { cartItems } = req.body;
    
    // Calculate amount server-side (don't trust client)
    const actualAmount = calculateTotal(cartItems);
    
    // Use actualAmount, not req.body.amount
    const paymentIntent = await stripe.paymentIntents.create({
        amount: actualAmount * 100,
        // ...
    });
});
```

### 3. Use HTTPS
- Always use SSL certificate
- Redirect HTTP to HTTPS

### 4. Implement CSRF protection
```javascript
const csrf = require('csurf');
app.use(csrf({ cookie: true }));
```

### 5. Rate limiting
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5 // 5 payment attempts per 15 minutes
});

app.post('/process-payment', limiter, async (req, res) => {
    // Process payment
});
```

## Testing

### Stripe Test Cards

```
Successful payment:
4242 4242 4242 4242

Requires authentication (3D Secure):
4000 0025 0000 3155

Declined card:
4000 0000 0000 9995

Insufficient funds:
4000 0000 0000 9995

Expiration: Any future date
CVV: Any 3 digits
ZIP: Any 5 digits
```

## Production Checklist

- [ ] Switch from test keys to live keys
- [ ] Set up SSL certificate (HTTPS)
- [ ] Configure webhook endpoints
- [ ] Set up email notifications
- [ ] Implement database backups
- [ ] Add error logging (Sentry, LogRocket)
- [ ] Test full payment flow
- [ ] Set up refund policy
- [ ] Configure tax settings (if applicable)
- [ ] Add terms and conditions
- [ ] Privacy policy
- [ ] PCI compliance verification

## Cost Calculator

### Example Order: 2-Year ID + 2 Lanyards + Badge Protector
```
2-Year ID:          $98.00
Lanyards (2):       $20.00
Badge Protector:    $10.00
Shipping:           $20.00
--------------------------
Subtotal:          $148.00

Stripe Fee (2.9% + $0.30):
$148.00 × 0.029 = $4.29
+ $0.30 = $4.59

You receive: $143.41
Stripe keeps: $4.59
```

## Recommended Hosting

### For Small-Medium Traffic:
- **Heroku** - Easy deployment, free tier available
- **DigitalOcean** - $5-10/month VPS
- **AWS Lightsail** - $3.50-5/month

### For High Traffic:
- **AWS** - Full suite, scalable
- **Google Cloud** - Similar to AWS
- **Vercel/Netlify** - Great for frontend + serverless functions

## Need Help?

- Stripe Documentation: https://stripe.com/docs
- Stripe Support: support@stripe.com
- Stack Overflow: Tag 'stripe-payments'

---

## Quick Start Commands

### Install Dependencies (Node.js)
```bash
npm init -y
npm install express stripe multer @sendgrid/mail dotenv cors
```

### Run Development Server
```bash
node server.js
```

### Environment Variables (.env file)
```
STRIPE_SECRET_KEY=sk_live_YOUR_SECRET_KEY
STRIPE_PUBLISHABLE_KEY=pk_live_YOUR_PUBLISHABLE_KEY
SENDGRID_API_KEY=YOUR_SENDGRID_KEY
DATABASE_URL=your_database_connection_string
PORT=3000
```

---

**Remember:** Always start with Stripe's test mode, then switch to live mode only after thorough testing!
