# Postback

Modern, intuitive affiliate marketing postback URLs app.

## Tech

- Next.js 14 (App Router) + TypeScript
- Prisma ORM
- Tailwind CSS
- SQLite (dev) / Postgres (prod ready)

## Getting started

1. Clone and install:

```bash
git clone https://github.com/xMartinezYT/postback.git
cd postback
npm i
```

2. Configure env:

```bash
cp .env.example .env
# For local dev keep SQLite:
# DATABASE_URL="file:./dev.db"
```

3. Generate Prisma client and migrate:

```bash
npx prisma generate
npx prisma migrate dev --name init
```

4. Run dev:

```bash
npm run dev
# open http://localhost:3000
```

## Core concepts

- Templates: define a slug per partner/network. Your inbound postback path is:
  - GET/POST /api/postback/{slug}
  - Example:
    - GET http://localhost:3000/api/postback/network-a?click_id=abc123&payout=1.23&currency=USD&status=approved&txid=tx-001

- Security:
  - IP allowlist: add rows in `IPRule` to explicitly allow or deny incoming IPs (UI coming later).
  - HMAC verification (optional): enable on a Template and set `hmacSecret`. The handler expects hex digest in the `x-signature` header (override with `hmacHeader`). The signature is computed over the raw query string (GET) or raw body (POST with `application/x-www-form-urlencoded`).

- Dedupe:
  - Uses a unique key based on `txid` if present, otherwise falls back to template + clickId + status + payout.

- Forwarding:
  - Configure `WebhookEndpoint` rows per template to forward conversion events to traffic sources or internal systems.

## API summary

- GET /api/templates -> list templates
- POST /api/templates -> create template
  - Body (JSON): `{ "name": "Network A", "slug": "network-a", "description": "..." }`
- GET/POST /api/postback/{slug} -> receive postback
  - Params: `click_id`, `payout`, `currency`, `status`, `txid` (flexible mappings supported)

## Next steps / roadmap

- UI pages to edit template security (HMAC on/off, secret, IP strict), manage webhook endpoints
- Better status mapping, custom param mapping per template
- Retry & backoff for forwarding
- Auth (optional, multi-tenant)
- Export CSV and analytics breakdowns (by offer/network/source)
- Tests

## Deploy

- Docker:
  - `docker build -t postback .`
  - `docker run -p 3000:3000 --env-file .env postback`
- Postgres:
  - Set `DATABASE_URL` in `.env` to your Postgres URL, run `npx prisma migrate deploy`.