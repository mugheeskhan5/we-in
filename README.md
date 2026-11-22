[![Frontend integration](https://github.com/mugheeskhan5/we-in/actions/workflows/frontend.yml/badge.svg)](https://github.com/mugheeskhan5/we-in/actions/workflows/frontend.yml)

[![Backend checks](https://github.com/mugheeskhan5/we-in/actions/workflows/backend.yml/badge.svg)](https://github.com/mugheeskhan5/we-in/actions/workflows/backend.yml)

# Dastak - Assisted Fraud Reporting

Dastak helps a citizen turn a text explanation or Urdu recording into a reviewed complaint
form, approve its sharing and submission, and follow human review through to a recorded outcome.

This is a hackathon prototype with **simulated reporting destinations and services**.
It is not an official government portal. Use synthetic information only.

## What the application does

- Gemini-assisted Urdu transcription, structured fact suggestions, classification and follow-up questions.
- Citizen-reviewed form filling, prepared complaint download and saved Urdu spoken review.
- Explicit, scoped sharing permissions and separate final submission authorization.
- LangGraph filing steps with durable checkpoints, idempotent recovery and verified simulated receipts.
- Citizen-requested Helper assistance with acknowledgment, replies and revocable access.
- Admin assignment to a matching Officer; clarification requests, responses and recorded decisions.
- Private evidence uploads with size/content validation and SHA-256 verification.
- Role/workspace isolation, version checks, immutable audit events and in-app notifications.

## Deploy from the repository

Follow **[DEPLOYMENT.md](DEPLOYMENT.md)** for Render Free and Supabase Free.
The root `render.yaml` defines the static frontend and Docker API. Supply credentials in the
hosting dashboards; no application code edits are required for normal setup.

The deployment uses Supabase PostgreSQL and a private evidence bucket. Saved Urdu speech
stays in PostgreSQL. Local Docker development retains its existing private file volume.

Release workflow: verify the deployment candidate on `integration/deploy-render`, merge the
verified code into `main`, then connect hosting to `main` and complete the hosted acceptance
checks. CI readiness does not create hosting accounts or supply credentials automatically.

## Run locally

Requirements: Docker Desktop/Compose, Node.js 24 and npm. Python 3.12 is needed for backend
development outside Docker and is useful for generating a local JWT secret.

From the repository root:

```powershell
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

Set the generated JWT secret, a URL-safe PostgreSQL password and a demo password of at least
12 characters in `.env`. Set your Gemini key and an available model to use AI features.
Keep existing `.env` values when updating an established installation.

```powershell
docker compose up --build -d
docker compose run --rm api python -m app.seed
docker compose exec api python -m app.release_check
```

Compose waits for PostgreSQL and applies migrations before starting the API.
Seeding preserves existing accounts and passwords. Data persists in named database/evidence
volumes; do not remove those volumes as an update procedure.

In a second terminal:

```powershell
Set-Location "hackathon frontend"
if (-not (Test-Path .env)) { Copy-Item .env.example .env }
npm ci
npm run dev -- --host 127.0.0.1 --port 5173 --strictPort
```

Open `http://127.0.0.1:5173/login`. The API is at `http://localhost:8000`, its implemented
OpenAPI documentation at `/docs`, and readiness at `/api/v1/health/ready`.
For local CORS configuration, allow both `http://localhost:5173` and `http://127.0.0.1:5173`.
Production uses exact HTTPS origins instead.

## Synthetic accounts

Freshly seeded accounts use the password you supplied in `DASTAK_DEMO_PASSWORD`.
Rerunning the seed does not reset existing passwords.

| Account | Role | Scope |
| --- | --- | --- |
| demo-citizen-1 | Citizen | Own complaints |
| demo-citizen-2 | Citizen | Own complaints |
| demo-helper | Helper | Citizen-granted fields |
| demo-admin | Admin | Governance, assignment metadata and audit |
| demo-officer-financial | Officer | Assigned financial workspace cases |
| demo-officer-digital | Officer | Assigned digital-safety workspace cases |

## Verification

GitHub Actions runs backend checks against SQLite and PostgreSQL, contract/migration checks,
frontend deployment configuration tests, production builds and browser workflows. Docker checks
exercise non-root storage, the audio decoder and the actual migration/seed/startup command.

From an activated Python 3.12 backend development environment:

```powershell
python -m pip install -r requirements-dev.txt
python -m pytest -q
ruff check app migrations tests
ruff format --check app migrations tests
```

PostgreSQL tests use `TEST_DATABASE_URL` and apply/downgrade migrations for each test.
Use a dedicated, disposable test database; never an application database containing wanted data.

From the frontend directory:

```powershell
node --test deployment-tests/config.test.mjs
npm run build
npm run lint
npm run test:e2e
```

The browser suite starts isolated API/storage fixtures and requires Playwright Chromium.
See [DEMO_AND_EVALUATION.md](DEMO_AND_EVALUATION.md) for the full-role demo and the 36-case
Gemini evaluation. AI calls require explicit permission and an available provider quota.

## Architecture and documentation

React/Vite frontend; FastAPI/Pydantic API; SQLAlchemy/Alembic/PostgreSQL persistence;
Gemini interpretation; gTTS spoken review; LangGraph approved-workflow coordination.
Run one backend worker initially: process-local guards and login limits are not distributed.

| Topic | Guide |
| --- | --- |
| Cloud deployment, persistence, backups and rollback | [DEPLOYMENT.md](DEPLOYMENT.md) |
| Demo and AI evaluation | [DEMO_AND_EVALUATION.md](DEMO_AND_EVALUATION.md) |
| Frontend/backend integration | [INTEGRATION.md](INTEGRATION.md) |
| Voice workflow | [backend/INTEGRATED_VOICE.md](backend/INTEGRATED_VOICE.md) |
| Evidence storage and access | [backend/EVIDENCE.md](backend/EVIDENCE.md) |
| Approved submission and recovery | [backend/SUBMISSION.md](backend/SUBMISSION.md) |
| Officer review | [backend/OFFICER_REVIEW.md](backend/OFFICER_REVIEW.md) |
| Scoped API keys and audit | [backend/API_KEYS_AND_AUDIT.md](backend/API_KEYS_AND_AUDIT.md) |
| Canonical API contracts | [contracts/openapi.yaml](contracts/openapi.yaml) |

`plan.md` remains the project plan. The `be`, `frontend` and integration branches retain the
development history; `main` is the release branch once the verified deployment pass is merged.
