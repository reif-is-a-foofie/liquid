# Liquid Gold Roadmap

This roadmap translates the MVP vision into sequenced milestones so agents can plan, execute, and report progress autonomously. Agents should keep this document current—update it whenever scope or sequencing changes.

## Guiding Principles

- Deliver demo-ready functionality quickly; production-hardening can wait until after milestone sign-off.
- Prefer stitching together APIs and open-source packages to bespoke builds.
- Maintain tight feedback loops: finish small increments, test them, and surface blockers via GitHub issues.

## Milestone 0 — Project Setup & Guardrails

**Goal:** Establish shared context, automation rules, and baseline tooling.

- [x] Publish `README.md` with Cursor development rules.
- [x] Create root `AGENTS.md` with automation guardrails.
- [ ] Add issue/PR templates and labeling scheme.
- [ ] Stand up GitHub Project board + automations.
- [ ] Define environment variable catalog (`docs/env.md`).

## Milestone 1 — Backend Foundation

**Goal:** Stand up a FastAPI backend with auth, database models, and basic account flows.

- Scaffold FastAPI project under `backend/` (uv + SQLAlchemy 2.x).
- Implement database models + Alembic migrations for `users`, `wallets`, `transactions`, `cards`.
- Build auth endpoints (`create_user`, `login_user`) with JWT + hashed passwords.
- Provide `get_balance` and transaction logging service stubs.
- Configure test harness (Pytest + SQLite in-memory or containerized Postgres).
- Document setup in `backend/README.md`.

## Milestone 2 — Conversion & Integrations

**Goal:** Wire sandbox integrations to support USD↔PAXG conversions and card lifecycle.

- Implement Coinbase/Kraken sandbox exchange service with retry + rate limiting.
- Add Paxos/PAXG balance verification service.
- Implement `convert_usd_to_paxg`, `convert_paxg_to_usd` endpoints.
- Create Stripe Issuing virtual card service + card creation API.
- Build Stripe webhook handler for `issuing_authorization.request` with auto-sell logic.
- Mock Plaid + Persona/Alloy flows for demo accounts.
- Extend tests to cover happy-path conversions + webhook handling.

## Milestone 3 — Frontend Experience

**Goal:** Deliver a minimal React + Tailwind dashboard covering auth, balances, cards, and Treasure Grid.

- Scaffold Vite React TypeScript app under `frontend/` with Tailwind + React Query.
- Implement Login/Register forms calling backend auth.
- Build Dashboard view showing PAXG (oz) + USD balances and transaction history.
- Implement Card Management view with virtual card details, freeze toggle.
- Create Treasure Grid with Leaflet map + placeholder geodata.
- Add Vitest + RTL tests for critical flows.
- Document frontend setup + .env usage.

## Milestone 4 — Demo Readiness & Deployment

**Goal:** Containerize the stack, wire monitoring, and script demo data.

- Create Dockerfiles for backend + frontend; add `docker-compose.yml` under `infra/`.
- Provide Railway/Fly deploy configs and document deployment steps.
- Seed demo data/script for sample users, wallets, and transactions.
- Integrate Sentry (or stub) for backend + frontend.
- Add pgAdmin or SQLPad instructions for database visibility.
- Run end-to-end smoke (manual/automated) covering deposit→conversion→card swipe.

## Beyond MVP (Future Considerations)

- Harden security (rate limiting, audit logging, secrets rotation).
- Add real KYC workflows + compliance logging.
- Implement production billing/reconciliation.
- Expand Treasure Grid with real geospatial data + filters.

## How Agents Use This Roadmap

1. Locate the highest-priority milestone with unchecked tasks.
2. Open a GitHub issue per task (or logical grouping) using the agent template.
3. Execute work per `AGENTS.md`, updating checkboxes as tasks complete.
4. Surface blockers by commenting on the relevant issue and linking here if scope changes.

Keep this document synchronized with reality—if priorities shift, update milestones and note the change in the associated issue or project log.
