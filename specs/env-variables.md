---
title: Environment Variables
tags:
  - spec
  - technical
  - infrastructure
status: approved
last-updated: 2026-05-05
---

# Environment Variables

**Summary**: All environment variables required across the three YachtBay apps. Use this as the reference when creating `.env.example` files during scaffolding and when configuring Railway services.

---

## Backend (`apps/backend`)

### Database & Cache
```env
# PostgreSQL connection string (Railway provides this automatically)
DATABASE_URL="postgresql://user:password@host:5432/yachtbay?schema=public"

# Redis connection string (Railway provides this automatically)
REDIS_URL="redis://localhost:6379"
```

### Auth & Security
```env
# JWT secrets — generate with: openssl rand -hex 64
JWT_ACCESS_SECRET="your-access-token-secret-min-64-chars"
JWT_REFRESH_SECRET="your-refresh-token-secret-min-64-chars"

# Token expiry
JWT_ACCESS_EXPIRES_IN="15m"
JWT_REFRESH_EXPIRES_IN="7d"

# Cookie domain — use ".yachtbay.com" in production, "localhost" in development
# The leading dot enables sharing across subdomains
COOKIE_DOMAIN=".yachtbay.com"
```

### Stripe
```env
# Secret key — use sk_test_... in development, sk_live_... in production
STRIPE_SECRET_KEY="sk_test_..."

# Webhook signing secret — from Stripe Dashboard → Webhooks
# Different secret for test vs live mode
STRIPE_WEBHOOK_SECRET="whsec_..."
```

### Resend (Email)
```env
# API key from resend.com dashboard
RESEND_API_KEY="re_..."

# From address (must be a verified domain in Resend)
EMAIL_FROM="noreply@yachtbay.com"
EMAIL_FROM_NAME="YachtBay"
```

### Cloudflare R2 (Image Storage)
```env
# Found in Cloudflare dashboard → R2 → Manage R2 API Tokens
CLOUDFLARE_R2_ACCOUNT_ID="your-account-id"
CLOUDFLARE_R2_ACCESS_KEY_ID="your-access-key-id"
CLOUDFLARE_R2_SECRET_ACCESS_KEY="your-secret-access-key"

# Bucket name (create in Cloudflare R2 dashboard)
CLOUDFLARE_R2_BUCKET_NAME="yachtbay-photos"

# Public CDN URL for the bucket (configure custom domain in R2 or use r2.dev URL)
CLOUDFLARE_R2_PUBLIC_URL="https://photos.yachtbay.com"
```

### Mapbox (Server-side geocoding)
```env
# Secret token for server-side Mapbox Geocoding API calls
# Use a token with geocoding scope only — not the same as the public frontend token
MAPBOX_SECRET_TOKEN="sk.eyJ1..."
```

### CORS & URLs
```env
# Allowed origins for CORS — comma-separated in production
STOREFRONT_URL="https://yachtbay.com"
OWNER_PANEL_URL="https://owners.yachtbay.com"
ADMIN_PANEL_URL="https://admin.yachtbay.com"

# Development:
# STOREFRONT_URL="http://localhost:3001"
# OWNER_PANEL_URL="http://localhost:3002"
# ADMIN_PANEL_URL="http://localhost:3003"
```

### App Config
```env
PORT=3000
NODE_ENV="development"   # development | production

# Platform default commission rate (percentage, e.g. 13 = 13%)
# Can be overridden per owner in the database
PLATFORM_COMMISSION_RATE=13
```

### BullMQ (Queue names — optional overrides)
```env
QUEUE_EMAIL="email-queue"
QUEUE_PAYMENT="payment-queue"
QUEUE_HOST="host-queue"
QUEUE_AUDIT="audit-queue"
```

---

## Storefront (`apps/storefront`)

React Router 7 (SSR). All variables are server-side by default. Variables prefixed with `VITE_` are exposed to the browser bundle.

```env
# Backend API base URL
API_URL="https://api.yachtbay.com/api/v1"
# Development: API_URL="http://localhost:3000/api/v1"

# Mapbox public token — safe to expose to browser
# Create a separate token with only map display scopes (no geocoding)
VITE_MAPBOX_PUBLIC_TOKEN="pk.eyJ1..."

# Stripe publishable key — safe to expose to browser
VITE_STRIPE_PUBLISHABLE_KEY="pk_test_..."

# Owner panel URL — used for "Switch to Owner Panel" link
VITE_OWNER_PANEL_URL="https://owners.yachtbay.com"

NODE_ENV="development"
```

---

## Owner Panel (`apps/owner-panel`)

React Router 7 (SPA mode). Same conventions.

```env
# Backend API base URL
API_URL="https://api.yachtbay.com/api/v1"
# Development: API_URL="http://localhost:3000/api/v1"

# Mapbox public token (same token as storefront is fine)
VITE_MAPBOX_PUBLIC_TOKEN="pk.eyJ1..."

# Stripe publishable key
VITE_STRIPE_PUBLISHABLE_KEY="pk_test_..."

# Storefront URL — used for "Switch to Renter View" link
VITE_STOREFRONT_URL="https://yachtbay.com"

NODE_ENV="development"
```

---

## Admin Panel (`apps/admin`)

React Router 7 (SPA mode). Minimal — no Stripe Elements or maps needed.

```env
# Backend API base URL
API_URL="https://api.yachtbay.com/api/v1"
# Development: API_URL="http://localhost:3000/api/v1"

NODE_ENV="development"
```

---

## Railway Configuration Notes

Railway injects `DATABASE_URL` and `REDIS_URL` automatically when you provision a PostgreSQL or Redis service and link it to your backend service. You don't need to set these manually — Railway handles the connection string.

All other variables must be added manually in Railway → Service → Variables.

**Per-environment strategy:**
- Railway project `yachtbay-staging` → use Stripe test keys (`sk_test_`, `pk_test_`)
- Railway project `yachtbay-production` → use Stripe live keys (`sk_live_`, `pk_live_`)

---

## Secrets Checklist (before going live)

- [ ] JWT secrets are unique, minimum 64 characters, never reused
- [ ] Stripe webhook secret matches the live webhook endpoint registered in Stripe dashboard
- [ ] Stripe keys switched from test (`sk_test_`, `pk_test_`) to live (`sk_live_`, `pk_live_`)
- [ ] R2 bucket CORS configured to allow uploads from storefront and owner panel origins
- [ ] Mapbox tokens scoped correctly (server token has geocoding, client token has maps only)
- [ ] `NODE_ENV=production` in all Railway production services
- [ ] `COOKIE_DOMAIN=.yachtbay.com` in production (not localhost)
- [ ] `EMAIL_FROM` domain verified in Resend

---

## Related Pages

- [[roadmap/phase-1-development]]
- [[specs/backend-architecture]]
- [[wiki/concepts/tech-stack]]
- [[decisions/2026-05-01-stripe-connect-accounts-v2]]
