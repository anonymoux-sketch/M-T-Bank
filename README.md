# M&T-style Personal Banking Dashboard Demo — Render-ready package

This is an independent demonstration project. It is **not affiliated with, endorsed by, or connected to M&T Bank**, and it does not access M&T Bank systems or accounts. The original three-bar green mark is retained, with an “M&T Bank” text label beside it throughout the demo UI.

## Deploy on Render (recommended: deploy the whole app, not GitHub Pages)

This repository includes `render.yaml`, which describes one Node web service and one managed PostgreSQL database. The web service serves the frontend and API from the same origin, so the existing `/api/...` sign-up and login requests work without cross-origin cookie configuration.

1. Extract this ZIP and create a new GitHub repository, then upload the **contents** of the `mt_bank_dashboard_final` folder to its repository root (including `render.yaml`).
2. In Render, choose **New → Blueprint**, connect that GitHub repository, and let Render read `render.yaml`.
3. Review the services and costs shown by Render, then approve creation of the web service and managed PostgreSQL database. The database plan in the blueprint is `basic-256mb`; the web service is `starter`. Render plan availability and prices can change, so check the amounts shown in your account before confirming.
4. Render creates `DATABASE_URL`, generates `JWT_SECRET` and `ADMIN_API_KEY`, and deploys the Node app. On startup, the app applies `schema.sql` automatically; you do not need to paste SQL manually.
5. When deployment is healthy, open the **Render web service URL** (the `onrender.com` address), not the old GitHub Pages URL. Create an account there, then sign in.

### Important about GitHub Pages

GitHub Pages cannot run this Node backend or connect directly to PostgreSQL. You can keep the repository on GitHub as the source, but use the Render web-service URL as the working application. Do not expect the old `https://anonymoux-sketch.github.io/M-T-Bank/` URL to process sign-up or login.

## What is included

- Registration and login backed by PostgreSQL; passwords are stored as bcrypt hashes, not plain text.
- HttpOnly session cookies, rate limiting, and user-scoped account queries.
- Database tables for users, USD accounts, transactions, and incoming international-payment records.
- Render Blueprint for the Node web service and managed PostgreSQL database.
- Automatic, idempotent database schema setup during server startup.
- Responsive UI, mobile navigation, and persistent light/dark appearance.

New registrations receive an Everyday Checking account with a zero balance. This is intentional: no real funds are created or represented as available. Any sample figures in the visual design must not be treated as real bank balances.

## Local development

Requires Node.js 20+ and PostgreSQL. Copy `.env.example` to `.env`, set `DATABASE_URL` and a strong `JWT_SECRET` (at least 32 characters), then run `npm install` and `npm start`. The schema is applied automatically on startup.

## Incoming payment records and transfers

The protected server-to-server `/api/admin/incoming-payments` endpoint can record payment information after a trusted provider confirms it. Keep `ADMIN_API_KEY` private; never put it in browser code or commit it to GitHub. Do not mark a payment received based only on screenshots, emails, or unverified claims.

**Real transfers are not enabled.** The transfer endpoint intentionally returns HTTP 503 until a licensed provider, contractual authorization, identity/compliance checks, signed webhooks, reconciliation, fraud controls, and a reviewed provider integration are implemented. Changing an environment variable alone will not enable money movement.

## Before real-world use

Add email verification, password reset, MFA, audit logging, administrator access controls, monitoring, backups, security review, and jurisdiction-appropriate legal/compliance controls. This package is a prototype starter, not production banking software.
