# AGENTS.md – Liquid Gold

This file provides the canonical instructions for automation agents (Cursor, Codex CLI, etc.) working inside the Liquid Gold repository. It supplements the human-facing `README.md` with task-specific guardrails, structure, and tooling expectations. Unless a more specific `AGENTS.md` exists deeper in the tree, treat these rules as authoritative.

## Project Overview

- **Goal**: Ship the Liquid Gold MVP quickly by stitching together open-source services and third-party APIs that provide gold-backed card spending, conversions between USD and PAXG, and a lightweight customer dashboard.
- **Mindset**: Follow the README’s mantra—scaffold fast, rely on existing libraries, and optimize for demo-ready functionality over production polish. Custom code should only glue systems together.
- **Primary Stack**:
  - Backend: FastAPI + SQLAlchemy, Postgres, JWT auth.
  - Frontend: React (Vite) + Tailwind CSS.
  - Integrations: Stripe Issuing (test mode), Paxos PAXG, Coinbase/Kraken sandbox, Plaid sandbox, Persona/Alloy mocks.
- **Source of truth**: Always re-read `README.md` before large changes to ensure alignment with current priorities and API selections.

## Repository Layout Expectations

Create or maintain the following top-level directories as the codebase grows:

- `backend/` – FastAPI app code.
  - `app/main.py` – FastAPI entrypoint with router registration and startup/shutdown hooks.
  - `app/api/` – Route modules grouped by domain (`auth`, `wallet`, `cards`, etc.).
  - `app/schemas/` – Pydantic models for request/response payloads.
  - `app/models/` – SQLAlchemy ORM models mapping to Postgres tables.
  - `app/services/` – Business logic and API integrations (Paxos, Coinbase, Stripe, Plaid, Persona/Alloy).
  - `app/core/` – Settings, security, utils (JWT helpers, dependency wiring).
  - `app/db/` – Session management, engine setup, Alembic integration.
  - `tests/` – Pytest suites targeting services and routes (prioritize happy-path demos first).
  - `migrations/` – Alembic migration scripts; never hand-edit once generated.
- `frontend/` – React + Tailwind client.
  - `src/pages/` – Route-level screens (`Login`, `Register`, `Dashboard`, `CardManagement`, `TreasureGrid`).
  - `src/components/` – Shared UI pieces (tables, balance widgets, form controls).
  - `src/hooks/` – Data fetching and state abstractions (React Query/SWR wrappers).
  - `src/lib/` – API clients, configuration helpers.
  - `src/styles/` – Tailwind entrypoint, any global styles.
  - `tests/` – Vitest/RTL specs for critical flows (auth form submits, balance rendering).
- `infra/` – Dockerfiles, docker-compose samples, Fly.io/Railway deployment manifests, Terraform if added.
- `scripts/` – Automation helpers (database seeding, sandbox data loaders). Prefer Python or shell.
- `docs/` – Diagrams, API playbooks, integration notes.
- `.github/` – Workflows for CI (lint/test/build) and issue templates.

Only introduce new top-level folders after confirming they do not duplicate existing structure. If a subdirectory needs specialized agent guidance, add another `AGENTS.md` inside that folder.

## General Guidance for Agents

1. **Leverage open source first**: before coding, search PyPI/npm for mature packages (e.g., `stripe`, `coinbase`, `krakenex`, `plaid-python`, `persona`). Do not hand-roll SDKs.
2. **Scaffold with Cursor**: begin each feature with a high-level prompt that mirrors README language (e.g., “Create FastAPI route for converting USD to PAXG using Coinbase sandbox API”). Review and adjust generated code manually.
3. **Keep modules swappable**: isolate third-party logic in service modules so APIs can be replaced without touching routers or schemas.
4. **Favor clarity over cleverness**: stick to conventional patterns the broader ecosystem expects (FastAPI dependency injection, React Query for data fetching, etc.).
5. **Respect secrets**: treat API keys and connection strings as env vars wired via `.env`, Docker secrets, or Railway/Fly configuration. Never hard-code credentials.
6. **Document as you go**: when adding notable behavior or setup quirks, update `docs/` or relevant README subsections to keep humans in the loop.

## Backend Standards (FastAPI)

- **Python version**: Target Python 3.11 unless a higher version is explicitly requested.
- **Dependency management**: prefer `uv` or `pip-tools` for repeatable installs. If creating scaffolding, include `requirements.txt` plus optional `requirements-dev.txt`.
- **Project layout**: follow the structure outlined above. Avoid monolithic files—split routers and services by domain.
- **Key functions to implement** (per README): `create_user`, `login_user`, `get_balance`, `convert_usd_to_paxg`, `convert_paxg_to_usd`, `create_virtual_card`, `log_transaction`.
- **Authentication**: use JWT (PyJWT or FastAPI’s `OAuth2PasswordBearer`). Store hashed passwords using `passlib`’s `bcrypt` context.
- **Database**:
  - ORM: SQLAlchemy 2.x with Declarative Base.
  - Run `alembic init` under `backend/` and keep migrations synced with models.
  - Schema matches README tables (users, wallets, transactions, cards). Ensure foreign keys and indexes for lookup speed.
- **Services & Integrations**:
  - Wrap external API calls in dedicated modules under `app/services`. Provide mockable interfaces for testing.
  - Default to sandbox/test environments; expose environment toggles for production later.
  - Stripe Issuing webhook (`issuing_authorization.request`) lives under `app/api/webhooks/stripe.py` (or similar). Implement basic authorization/resell logic.
- **Validation & errors**: use Pydantic models for request validation, raise `HTTPException` with helpful messages for known failure modes.
- **Testing**: configure Pytest with a temporary Postgres (or SQLite for unit tests) and fixture data. Focus on integration tests that mirror critical flows: signup, conversion, card auth webhook.

## Frontend Standards (React + Tailwind)

- **Scaffold** with Vite + React + TypeScript. Tailwind should be wired through `postcss.config.cjs` and `tailwind.config.cjs`.
- **Package manager**: default to `pnpm`. If not available, `npm` is acceptable; keep lockfiles committed (`pnpm-lock.yaml` or `package-lock.json`).
- **State/data**: prefer lightweight data fetching (React Query/SWR) to keep API integration code consistent. Global state via Context only when necessary.
- **Pages**: implement the four required views (Login/Register, Dashboard, Card Management, Treasure Grid). Keep UI minimal—table for history, simple cards for balances.
- **Components**: use functional components with TypeScript props interfaces. Co-locate component-specific styles.
- **Maps**: Treasure Grid page should integrate Leaflet.js with a simple tile provider. Wrap map initialization in a component under `src/components/Map`.
- **Testing**: use Vitest + React Testing Library for essential flows (authentication form submit, balance display, card freeze toggle).
- **Accessibility**: ensure forms and buttons use semantic elements and focus states provided by Tailwind.

## API Integration Notes

- **Paxos/PAXG**: use official or community SDKs for account balance verification. Mock responses for local development.
- **Coinbase/Kraken**: only sandbox endpoints. Create abstraction in `app/services/exchange.py` to allow swapping providers.
- **Stripe Issuing**: keep webhook secrets in env vars. Validate signatures using Stripe’s SDK helpers. Provide idempotency on authorization handling.
- **Plaid Sandbox**: generate link tokens via sandbox API; store public tokens temporarily for ACH simulation.
- **KYC (Persona/Alloy)**: stub or mock flows; expose toggle to bypass for demo accounts.
- When adding new integrations, update `.env.example` and `docs/integrations.md` (create if missing).

## Deployment & Ops

- **Docker**: maintain `backend/Dockerfile`, `frontend/Dockerfile`, and optional `docker-compose.yml` under `infra/`. Compose should start FastAPI, Postgres, and frontend dev server or prebuilt assets.
- **Railway/Fly**: keep deployment manifests (service.toml, fly.toml) versioned under `infra/`. Ensure env var names match those used in settings modules.
- **Monitoring**: integrate Sentry SDK (backend + frontend) when feasible. Document DSN retrieval in `docs/monitoring.md`.
- **Database admin**: include `pgAdmin` or alternative instructions in `infra/` for local debugging.

## Development Workflow

1. Start with a Cursor prompt that outlines the feature’s goal and key APIs.
2. Review generated scaffolding before applying patches; do not blindly accept.
3. Use `apply_patch` for edits and keep diffs focused on the requested change.
4. Run the smallest relevant tests whenever possible:
   - Backend: `cd backend && uv run pytest` (or `pip install -r requirements-dev.txt && pytest`).
   - Frontend: `cd frontend && pnpm test`.
5. Update documentation (`README.md`, `docs/`) when workflows or env vars change.
6. Keep commit scope tight; avoid unrelated refactors.

## File Ownership & Nested Instructions

- This `AGENTS.md` applies to the entire repository. If you create subdirectories with specialized processes (e.g., `infra/`, `docs/`), add another `AGENTS.md` within that folder for overrides. The most specific file always wins.
- Do not modify or remove `AGENTS.md` instructions without explicit user approval.

## Quick Reference Commands

```sh
# Backend
cd backend
uv sync && uv run fastapi dev app/main.py          # local dev
uv run pytest                                      # tests
alembic revision --autogenerate -m "<message>"     # create migration
alembic upgrade head                               # apply migration

# Frontend
cd frontend
pnpm install
pnpm dev                                           # start Vite
pnpm build                                         # production build
pnpm test                                          # vitest suite

# Docker (example)
cd infra
docker compose up --build
```

Keep this file current as the architecture evolves—agents rely on it for deterministic behavior.
