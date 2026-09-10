# High-Level Design (HLD) — AI Task & Knowledge Management Engine

## 1) Title & Metadata

| Field | Value |
|---|---|
| Repository | `NikithaJoshy/ai_task_management` |
| Last updated | 2026-09-10 |
| Doc owner | Not determined from repository |
| Source baseline | `README.md`, `backend/`, `frontend/`, dependency manifests, Alembic config |

## 2) Executive Overview

Full-stack task and knowledge platform composed of:
- React SPA (`frontend/`) for user workflows.
- FastAPI backend (`backend/app/`) for auth, tasks, documents, AI task generation, analytics, and audit access.
- MySQL via SQLAlchemy for operational records.
- Local vector search (FAISS + sentence-transformers) and local file storage for uploaded documents.

## 3) Objective

Provide authenticated users with:
- Task lifecycle management.
- Document upload/search for knowledge retrieval.
- AI-assisted task drafting.
- Basic analytics and audit visibility (admin for audit endpoint).

## 4) Architecture Description

```mermaid
flowchart TB
  subgraph Client[Client Layer]
    U[User Browser]
    SPA[React SPA\nfrontend/src]
    LS[localStorage\naccess_token, user]
  end

  subgraph API[API Layer]
    FAPI[FastAPI App\nbackend/app/main.py]
    R1[/auth,/users,/tasks]
    R2[/documents,/ai,/analytics,/audit-logs]
    DEP[Auth Dependencies\nJWT + RBAC]
  end

  subgraph Core[Core/Service Layer]
    TASKS[Task Service Logic]
    DOCS[Document Processing\nextract + chunk]
    VEC[Vector Service\nFAISS + embeddings]
    AIGEN[AI Task Generation\nrule/regex + retrieved context]
  end

  subgraph Data[Data Layer]
    DB[(MySQL\nusers, tasks, activity_logs, documents)]
    FS[(Local Filesystem\nuploads/documents)]
    MEM[(In-memory FAISS index)]
  end

  EXT[Model Artifact Source\nSentenceTransformer model]

  U -->|HTTP(S) via browser| SPA
  SPA -->|HTTP + JWT header| FAPI
  SPA -->|read/write token| LS
  FAPI --> R1
  FAPI --> R2
  R1 --> DEP
  R2 --> DEP
  R1 --> TASKS
  R2 --> DOCS
  R2 --> VEC
  R2 --> AIGEN
  TASKS -->|ORM| DB
  DOCS -->|metadata| DB
  DOCS -->|save/read files| FS
  DOCS -->|index chunks| VEC
  VEC --> MEM
  AIGEN -->|top-k retrieval| VEC
  VEC -->|initial model load via HTTPS| EXT
```

## 5) Core Workflows

- Authentication: `POST /auth/login` validates credentials, logs `LOGIN`, returns JWT.
- Task management: authenticated user creates/updates/deletes own tasks; assignment validates active assignee.
- Document knowledge flow: upload (PDF/TXT) -> extraction -> chunking -> embedding index -> search endpoint for semantic retrieval.
- AI task generation: prompt plus top search hit context -> generated title/description/priority/due date.

## 6) Data Flow

- Credentials and task/document payloads originate from SPA API calls.
- JWT travels in `Authorization` header (stored in browser `localStorage`).
- Structured entities persisted to MySQL (`users`, `tasks`, `activity_logs`, `documents` model).
- Uploaded document binaries stored in `backend/uploads/documents`; extracted/chunked text vectors stored only in-memory FAISS process state.
- Analytics reads task and activity-log aggregates; audit endpoint returns activity log rows.

## 7) Key Features

- JWT authentication + admin role guard dependency.
- Task CRUD with ownership/assignment filtering.
- Document upload, view, download, and semantic search.
- AI-assisted task generation endpoint.
- Operational analytics and audit-log retrieval.

## 8) Infrastructure & Deployment Overview

- Runtime topology: separately started backend (Uvicorn) and frontend (Vite dev server).
- Database: external MySQL expected via `DATABASE_URL`.
- Migration/config tooling: Alembic configured in `backend/alembic.ini` and `backend/alembic/env.py`.
- CI/CD, containerization, and IaC: Not determined from repository (no workflows/Docker/Terraform manifests found).

## 9) Deployment Strategy

- Documented deployment is local/manual:
  - Backend: `uvicorn app.main:app --reload`.
  - Frontend: `npm run dev`.
- Branching/release strategy and automated promotion pipeline: Not determined from repository.

## 10) Data Protection

| Control area | Repository evidence |
|---|---|
| Data in transit | Client/server interaction uses HTTP base URL `http://127.0.0.1:8000` in frontend API client (TLS termination strategy not defined). |
| Data at rest | MySQL stores operational entities; uploaded files persist on local filesystem under `uploads/documents`. |
| Secrets management | `.env` is gitignored; `DATABASE_URL` loaded from env. |
| LLM/third-party sharing | No external hosted LLM API call path found; AI generation is local heuristic plus local vector retrieval. |
| Logging | Activity events persisted in `activity_logs` for auth/task/document actions. |
| Retention policy | Not determined from repository. |

## 11) Security Requirements

Current implemented controls:
- Password hashing (`pwdlib`).
- JWT validation and active-user enforcement.
- Admin authorization dependency for privileged endpoints (`/users`, `/audit-logs`).
- File-type allowlist for uploads (`.pdf`, `.txt`).

Observed risk items from code:
- Hardcoded JWT secret (`change-this-secret-key`) in backend code.
- Document endpoints return documents without per-owner restriction; all authenticated users can list/view/download.
- Document listing response includes internal `file_path` value.
- Token stored in browser `localStorage` (XSS exposure risk).

Dependency vulnerability posture:
- Not determined from repository (no SCA report committed).

## 12) Integrations

| Integration | Purpose | Auth mechanism |
|---|---|---|
| MySQL (`DATABASE_URL`) | Persistence for user/task/activity/document metadata | DB credentials in connection URL (env provided). |
| SentenceTransformer model source (`all-MiniLM-L6-v2`) | Embedding generation for semantic search | Not determined from repository (library-managed model download path). |

## 13) Environment Variables & Secrets Inventory

| Name | Usage |
|---|---|
| `DATABASE_URL` | SQLAlchemy engine connection string (`backend/app/database.py`). |
| `AI_API_KEY` | Declared in settings model (`backend/app/core/config.py`), active use not determined in current code path. |
| `SECRET_KEY` | Mentioned in `README.md` example; runtime code currently uses hardcoded constant instead of env loading. |
| `ALGORITHM` | Mentioned in `README.md` example; runtime code currently sets `HS256` constant. |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Mentioned in `README.md` example; runtime code currently sets `60` constant. |

## 14) Change Log

- **2026-09-10**: Bootstrapped `docs/HLD.md` from repository analysis. Added enterprise architecture baseline, layered Mermaid diagram, workflow/data/security/integration summaries, and explicit "Not determined from repository" markers for absent CI/IaC/retention details.
