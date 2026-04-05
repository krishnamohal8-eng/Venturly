# Venturly — AI-Powered P2P Micro-Lending Platform

> **"The Intelligent Bridge for Small Business Capital"**
> Team: Venture Coachers · NIT Jalandhar · Theme: Open Innovation

---

## Quick Start (Local)

```bash
# 1. Install dependencies
cd venturly/backend  && npm install
cd ../frontend       && npm install

# 2. Seed demo data
cd ../backend && npm run seed

# 3. Start backend (terminal 1)
npm run dev   # → http://localhost:3001

# 4. Start frontend (terminal 2)
cd ../frontend && npm run dev   # → http://localhost:3000
```

---

## Demo Accounts (password: `password123`)

| Role     | Email               | Name           |
|----------|---------------------|----------------|
| Borrower | maya@demo.com       | Maya Patel     |
| Borrower | carlos@demo.com     | Carlos Rivera  |
| Borrower | priya@demo.com      | Priya Singh    |
| Investor | alex@demo.com       | Alex Chen      |
| Investor | sarah@demo.com      | Sarah Johnson  |

---

## URLs

| Service         | URL                                    |
|-----------------|----------------------------------------|
| Frontend        | http://localhost:3000                  |
| API             | http://localhost:3001                  |
| Swagger UI      | http://localhost:3001/api              |
| OpenAPI JSON    | http://localhost:3001/api-json         |
| Health check    | http://localhost:3001/health           |

---

## Frontend Pages (14 routes)

| Route                              | Description                                      |
|------------------------------------|--------------------------------------------------|
| `/`                                | Homepage with live platform stats                |
| `/marketplace`                     | Browse, search, filter & fund loans              |
| `/calculator`                      | Loan & investment calculator                     |
| `/loans/[id]`                      | Loan detail: repayment schedule + audit trail    |
| `/register`                        | Register as borrower or investor                 |
| `/login`                           | Login (passkey or password)                      |
| `/dashboard/borrower`              | Apply for loans, live score preview, risk check  |
| `/dashboard/investor`              | Portfolio with inline repayment schedules        |
| `/dashboard/investor/analytics`    | Portfolio analytics & charts                     |
| `/stats`                           | Platform impact dashboard                        |
| `/profile`                         | Account settings + passkey management            |
| `/admin`                           | Admin: users, stats, audit trail                 |
| `/api-docs`                        | API reference with cURL examples                 |
| `/api-docs` → Swagger              | http://localhost:3001/api                        |

---

## API Reference (37 endpoints)

### Auth
| Method | Path | Description |
|--------|------|-------------|
| POST | /auth/register | Register (role: borrower/investor) |
| POST | /auth/login | Login, returns JWT (7 day expiry) |

### Passkeys (WebAuthn — passwordless login)
| Method | Path | Description |
|--------|------|-------------|
| GET | /passkeys/register/options | Get WebAuthn challenge to create passkey |
| POST | /passkeys/register/verify | Save passkey after biometric confirmation |
| POST | /passkeys/authenticate/options | Get challenge for passwordless login |
| POST | /passkeys/authenticate/verify | Verify biometric → returns JWT |
| GET | /passkeys | List my registered passkeys |
| DELETE | /passkeys/:id | Remove a passkey |
| POST | /passkeys/payment/challenge | Get biometric challenge before payment |
| POST | /passkeys/payment/verify | Verify biometric → returns paymentToken |

### Users
| Method | Path | Description |
|--------|------|-------------|
| GET | /users/me | Get my profile |
| PATCH | /users/me | Update name/password (passkey required if registered) |
| GET | /users/admin/users | Admin: all users |

### Loans
| Method | Path | Description |
|--------|------|-------------|
| GET | /loans | Paginated marketplace (?page, limit, grade, esg, search, sort) |
| GET | /loans/:id | Loan detail |
| POST | /loans/apply | Apply — Quantum Score engine grades instantly |
| GET | /loans/my/loans | My loans (borrower) |

### Investments
| Method | Path | Description |
|--------|------|-------------|
| POST | /investments | Fund a loan (tokens × $25) |
| GET | /investments/my | My investments |
| GET | /investments/my/stats | Portfolio stats |

### Payments (6 methods, all passkey-secured)
| Method | Path | Description |
|--------|------|-------------|
| POST | /payments/stripe/intent | Google Pay / Apple Pay / Card intent |
| POST | /payments/stripe/confirm | Confirm Stripe payment |
| POST | /payments/paypal/order | Create PayPal order |
| POST | /payments/paypal/capture | Capture PayPal payment |
| POST | /payments/razorpay/order | Create Razorpay order (UPI/India) |
| POST | /payments/razorpay/verify | Verify Razorpay payment |
| POST | /payments/bank-transfer/initiate | ACH bank transfer instructions |
| POST | /payments/webhook | Stripe webhook |

### Repayments
| Method | Path | Description |
|--------|------|-------------|
| GET | /repayments/:loanId/schedule | Amortization schedule |
| POST | /repayments/:loanId/generate | Generate schedule (post-funding) |
| POST | /repayments/:loanId/pay/:n | Pay installment #n |
| POST | /repayments/:loanId/risk-check | AI risk trigger simulation |

### Notifications
| Method | Path | Description |
|--------|------|-------------|
| GET | /notifications | My notifications (last 50) |
| GET | /notifications/unread/count | Unread badge count |
| PATCH | /notifications/read/all | Mark all read |
| PATCH | /notifications/:id/read | Mark one read |

### Audit & Stats
| Method | Path | Description |
|--------|------|-------------|
| GET | /audit | Hash-chained ledger (?loanId filter) |
| GET | /stats | Platform-wide impact metrics |

---

## Quantum Score Engine

| Signal | Points |
|--------|--------|
| Revenue-to-loan ratio | 0–40 |
| Months in business | 0–20 |
| Bank account connected (Plaid) | +10 |
| Sales platform connected | +10 |
| Base score | 20 |
| ESG category bonus | +10 |

| Score | Grade | APR |
|-------|-------|-----|
| 80–100 | A | 8% |
| 65–79 | B | 10% |
| 50–64 | C | 13% |
| 40–49 | D | 16% |
| < 40 | — | Rejected |

---

## Payment Methods

| Method | Provider | Header Required |
|--------|----------|-----------------|
| Google Pay | Stripe | `x-payment-token` |
| Apple Pay | Stripe | `x-payment-token` |
| Credit/Debit Card | Stripe | `x-payment-token` |
| PayPal | PayPal REST API | `x-payment-token` |
| UPI / NetBanking | Razorpay | `x-payment-token` |
| Bank Transfer | ACH | `x-payment-token` |

All payment endpoints require a `x-payment-token` header obtained from `POST /passkeys/payment/verify` (biometric confirmation). Users without passkeys receive a bypass token automatically.

---

## Environment Variables

### Backend (`backend/.env`)
```
DATABASE_URL=venturly.sqlite          # SQLite locally, PostgreSQL URL in production
JWT_SECRET=your_secret_here
PORT=3001
NODE_ENV=development

# Passkeys
RP_ID=localhost
ORIGIN=http://localhost:3000

# Stripe (Google Pay / Apple Pay / Card)
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Google Pay
GOOGLE_PAY_ENV=TEST
GOOGLE_PAY_MERCHANT_ID=BCR2DN4TR4XXXXXXXXX

# PayPal
PAYPAL_CLIENT_ID=...
PAYPAL_CLIENT_SECRET=...
PAYPAL_BASE_URL=https://api-m.sandbox.paypal.com

# Razorpay (UPI)
RAZORPAY_KEY_ID=rzp_test_...
RAZORPAY_KEY_SECRET=...

# Twilio (SMS)
TWILIO_ACCOUNT_SID=...
TWILIO_AUTH_TOKEN=...
TWILIO_FROM_NUMBER=+1...
```

### Frontend (`frontend/.env.local`)
```
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
NEXT_PUBLIC_GOOGLE_PAY_ENV=TEST
```

---

## Deploy to Production

### Backend → Railway
1. Push to GitHub
2. railway.app → New Project → Deploy from GitHub → set root to `backend`
3. Add env vars (see above) + Railway auto-adds `DATABASE_URL`
4. Set `NODE_ENV=production`

### Frontend → Vercel
1. vercel.com → Import repo → set root to `frontend`
2. Add `NEXT_PUBLIC_API_URL=https://your-railway-url.up.railway.app`
3. Deploy → get `https://venturly.vercel.app`

---

## Architecture

```
Frontend  Next.js 16 (Turbopack) — 14 pages
          Passkey login · Payment modal · Notification bell
          Calculator · Analytics · Admin dashboard

Backend   NestJS 10 + TypeORM — 37 API endpoints
          ├── auth          JWT register/login
          ├── passkeys      WebAuthn biometric login + payment auth
          ├── users         Profile CRUD + admin
          ├── loans         Quantum Score engine, paginated marketplace
          ├── investments   Token funding, portfolio stats
          ├── payments      6 methods: Google Pay, Apple Pay, Card, PayPal, Razorpay, ACH
          ├── repayments    Amortization, payments, risk triggers, dunning cron
          ├── notifications In-app alerts + Twilio SMS
          ├── audit         SHA-256 hash-chained immutable ledger
          └── stats         Platform aggregates

Database  SQLite (local) / PostgreSQL (production)
Security  JWT + WebAuthn passkeys + payment token binding
API Docs  Swagger UI (OpenAPI 3.0) + rate limiting (100 req/min)
```
