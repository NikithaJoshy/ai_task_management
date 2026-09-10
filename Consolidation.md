---
name: consolidate-design-docs
description: Merges the already-generated HLD and LLD documents from the frontend repo and backend repo into a single unified HLD and LLD, written for an Enterprise Architect audience. Does not re-analyze source code.
---

# Consolidate Design Documents

You are merging documentation that has **already been generated** — you are not
analyzing source code and you are not regenerating anything from scratch.

## Inputs (read-only)

Assume both repositories are checked out side by side in the current workspace:

- `frontend-repo/docs/HLD.md`
- `frontend-repo/docs/LLD.md`
- `backend-repo/docs/HLD.md`
- `backend-repo/docs/LLD.md`

If any of these files are missing, stop and report which one is missing rather
than guessing its contents or falling back to reading source code.

## Outputs

- `docs/HLD_consolidated.md`
- `docs/LLD_consolidated.md`

(Adjust these output paths if this skill is placed in a repo with an existing
`docs/` convention — keep whatever naming distinguishes the consolidated doc
from the individual per-repo docs.)

## Hard Rules

1. **Do not re-analyze code.** Your only inputs are the four markdown files
   listed above. If something seems missing or unclear, note it as a gap
   rather than inferring it from anything else.
2. **Do not simply concatenate.** A unified document should read as one
   coherent system description, not "frontend doc" + "backend doc" stapled
   together.
3. **Preserve accuracy.** Do not alter or reinterpret factual claims from the
   source documents — only reorganize, merge, and de-duplicate them.
4. **Update, don't rewrite**, on subsequent runs — if a consolidated doc
   already exists, only touch the sections whose source material changed.
5. **When merging Mermaid diagrams, re-validate syntax — do not just splice
   text together.** Follow the same Mermaid Syntax Rules used to generate the
   originals:
   - Never start a node label with `/` (e.g. `[/auth,/users,/tasks]` is
     invalid — reserved for trapezoid shapes).
   - Wrap any label containing a comma, slash, parenthesis, or colon in
     double quotes, e.g. `Gateway["API Gateway: /auth, /users, /tasks"]`.
   - One node or edge definition per line; never chain definitions with
     commas.
   - If the frontend and backend diagrams each define a node with the same
     ID but different meaning, rename one before merging — colliding IDs
     silently overwrite each other in Mermaid.
   - After merging, mentally re-parse the combined diagram line by line to
     confirm every label is either a bare short phrase or fully quoted.

## Merge Logic

### HLD merge (`docs/HLD_consolidated.md`)
- **Executive Overview / Objective**: write one combined overview describing
  the system as a whole (frontend + backend together), not two separate ones.
- **Architecture Description**: combine both Mermaid diagrams into a single
  `flowchart TB` showing the frontend and backend as connected layers/systems,
  with the connection between them clearly labeled (protocol, e.g. HTTPS/REST,
  gRPC, WebSocket). Do not just place two diagrams side by side — merge them.
- **Core Workflows / Data Flow**: merge into unified end-to-end workflows that
  span frontend → backend where applicable (e.g. "user submits form" → "API
  receives request" → "data persisted").
- **Data Protection**: combine into one section covering the full data
  lifecycle across both systems — where data enters (frontend), how it
  travels, and where it's ultimately stored (backend).
- **Security Requirements**: merge into one section; if frontend and backend
  have different auth mechanisms, describe both and how they relate (e.g.
  frontend obtains a token that the backend validates).
- **Integrations**: list all integrations from both docs in one table; note
  which side (frontend/backend) owns each integration.
- **Environment Variables & Secrets Inventory**: combine into one inventory,
  labeled by which repo/service each variable belongs to.
- **Change Log**: append a new entry noting this is a consolidation run and
  which source docs it pulled from (include their last-updated dates if
  present).
- Sections unique to only one side (e.g. a frontend-only "Key Features"
  section with no backend equivalent) should be kept, clearly scoped.

### LLD merge (`docs/LLD_consolidated.md`)
- **Module/Component Breakdown**: organize under two top-level groups,
  Frontend and Backend, but add a short "Cross-Cutting" note wherever a
  frontend module directly depends on a backend module or endpoint.
- **Sequence Diagrams**: prefer merged, end-to-end sequence diagrams for
  workflows that cross both systems, over keeping separate frontend-only and
  backend-only diagrams for the same workflow.
- **Data Models / Schemas**: combine into one section; flag where a
  frontend-side type and a backend-side schema represent the same entity.
- Everything else (Error Handling, Configuration, Known Limitations, Change
  Log): merge straightforwardly, keeping frontend/backend attribution where it
  aids clarity.

## Process

1. Read all four source files in full.
2. Read the existing consolidated docs, if present, to determine what's
   already reflected.
3. Identify what's new or changed in either source doc since the last
   consolidation.
4. Produce or update the two consolidated documents per the merge logic above.
5. Do not commit or open a PR as part of this skill — leave changes staged for
   the workflow or a developer to review.
