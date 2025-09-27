# Cursor Development Rules – Liquid Gold

## Purpose

Provide explicit rules for using **Cursor (AI IDE)** to build Liquid Gold’s MVP quickly, with maximum reliance on open-source and APIs. This document defines how engineers should structure code, prompts, and integrations.

---

## 1. General Rules

1. Always search for **existing open-source packages** first; do not reinvent.
2. Keep code **modular** (functions/services) for swapping APIs easily.
3. **Scaffold fast**: use Cursor to generate boilerplate, then refine manually.
4. Prioritize **demo functionality over production robustness** for MVP.
5. Minimum custom code → glue existing systems.

---

## 2. Backend Rules (FastAPI)

* Generate API scaffolds using Cursor prompt: *“Build FastAPI app with JWT auth, Postgres models for Users, Transactions, Wallets.”*
* Use `sqlalchemy` for ORM.
* Functions:

  * `create_user()`
  * `login_user()`
  * `get_balance(user_id)`
  * `convert_usd_to_paxg()`
  * `convert_paxg_to_usd()`
  * `create_virtual_card()`
  * `log_transaction()`
* Webhooks: implement Stripe Issuing `issuing_authorization.request` listener with auto-sell logic.

---

## 3. Database Rules (Postgres)

* Schema tables:

  * `users (id, email, hashed_pw, kyc_status)`
  * `wallets (user_id, paxg_balance, usd_float_balance)`
  * `transactions (id, user_id, type, amount, asset, timestamp, status)`
  * `cards (id, user_id, card_token, status)`
* Use migrations with `alembic`.
* Always log conversions + swipes.

---

## 4. Frontend Rules (React + Tailwind)

* Scaffold with Create React App or Vite.
* Pages:

  * Login/Register
  * Dashboard (balance, tx history)
  * Card Management (show virtual card)
  * Treasure Grid (Leaflet.js)
* Keep UI minimal: table for tx history, balance in ounces + USD, freeze/unfreeze button for card.

---

## 5. API Integration Rules

* **PAXG:** use Paxos API wrappers for balance verification.
* **Exchange:** Kraken/Coinbase API → always sandbox mode first.
* **Stripe Issuing:** generate cards in test mode. Prompt Cursor: *“Integrate Stripe Issuing API with FastAPI backend to create virtual cards and approve authorizations via webhook.”*
* **Plaid Sandbox:** simulate ACH deposits.
* **Persona/Alloy:** mock KYC flows for MVP.

---

## 6. Deployment Rules

* Containerize with Docker.
* Use Railway/Fly.io deploy templates.
* Secrets stored as env vars.
* Monitoring: Sentry + pgAdmin.

---

## 7. Development Workflow in Cursor

1. Start each feature with a **high-level prompt** (e.g., *“Create FastAPI route for converting USD to PAXG using Coinbase API”*).
2. Cursor generates scaffold.
3. Engineer edits for keys, configs, error handling.
4. Commit to GitHub repo.
5. Run container locally before pushing.

---

## 8. Success Criteria

* Cursor should generate **80% of boilerplate**.
* Engineers only focus on stitching APIs, debugging, and securing flows.
* By end of MVP, system must:

  * Accept deposit → convert to PAXG
  * Show balance → in ounces + USD
  * Swipe card → trigger float top-up → transaction logged
  * Display results in dashboard

---

**Rule of Thumb: If you can stitch it, don’t code it. Cursor is for scaffolding, not artistry.**
