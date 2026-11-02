# Secure Personal Banking Dashboard — final-stage starter

This package keeps the existing dashboard design, mobile sidebar drawer, and dark/light switch, and adds a PostgreSQL-backed API and secure registration/login flow.

## Included
- Registration and login with bcrypt password hashes, rate limiting, signed session tokens in HttpOnly/SameSite cookies, and logout.
- PostgreSQL schema for users, USD accounts, transaction history, and incoming international payments.
- User-scoped dashboard endpoint; account balances and records are loaded from the database, not hard-coded sample balances.
- Incoming-payment records include sender name, sender country, sender bank, amount, provider reference, timestamps, and status.
- Privileged server-to-server import/status endpoints. A payment credits the account only when its status is confirmed as `received`, and duplicate provider references are rejected.
- Transfer endpoint fails closed until an authorized provider is configured. No UI action falsely reports that funds moved.
- Responsive dashboard, sidebar toggle, and persistent dark/light theme.

## Run locally
Requires Node.js 20+ and PostgreSQL.

1. Create a PostgreSQL database, e.g. `mt_dashboard`.
2. Copy `.env.example` to `.env`; set `DATABASE_URL`, `JWT_SECRET` (a long random value), and `ADMIN_API_KEY`.
3. Apply the schema: `psql "$DATABASE_URL" -f schema.sql`
4. Install dependencies: `npm install`
5. Start: `npm start`
6. Open `http://localhost:3000`, create an account, and sign in.

Use `npm run check` for JavaScript syntax checks. Do not expose the admin API key in frontend code or commit `.env`.

## Importing an incoming payment (trusted server only)
The `/api/admin/incoming-payments` endpoint requires the `x-admin-api-key` header. It accepts `accountId`, `providerReference`, `senderName`, `senderCountry`, `senderBank`, `amount`, `status`, and optional `providerPayload`. Only call it from a trusted integration service after validating the payment with your provider. To find a user's account ID, use a secured internal/admin database workflow; do not expose other users' account IDs in the public client.

Example request body:

```json
{
  "accountId": "ACCOUNT_UUID",
  "providerReference": "PROVIDER_REFERENCE",
  "senderName": "Sender Name",
  "senderCountry": "United Kingdom",
  "senderBank": "Sender Bank",
  "amount": "250.00",
  "status": "pending"
}
```

Use `PATCH /api/admin/incoming-payments/:id/status` with `{ "status": "received" }` only after the provider confirms settlement. That operation posts the ledger credit once. Do not manually mark payments received based on screenshots, emails, or unverified claims.

## Important: real money movement
The app does **not** yet perform real international transfers. That requires selecting and contracting with a licensed bank/payment institution or authorized platform, completing KYB/KYC/AML and jurisdiction checks, implementing that provider's API and signed webhooks, reconciliation, fraud controls, and production security review. `PAYMENT_PROVIDER=none` is the safe default; the transfer endpoint returns HTTP 503. Merely changing the environment variable does not enable money movement; a provider adapter still needs to be implemented and tested against the chosen provider's official API.

## Production checklist
- Deploy behind HTTPS and set `NODE_ENV=production` and `COOKIE_SECURE=true`.
- Use a managed PostgreSQL service, backups, migrations, monitoring, and least-privilege DB credentials.
- Configure strong secrets in the host's secret manager, not source control.
- Add email verification, password reset, MFA, audit logs, admin authorization/rotation, fraud review, and account recovery before real customers use it.
- Replace the current product branding with your own authorized brand. This is not an M&T Bank service and has no connection to M&T Bank systems.
