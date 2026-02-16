# Architecture v1

## Metadata
- Concept ID: `44593131-19af-4667-8f9a-e0e4458a9389`
- Artifact Type: `architecture`
- Version: `v1`

## System Overview
A single web application provides:
1. REST API for deterministic pipeline orchestration.
2. Persistence layer for concepts and versioned markdown artifacts.
3. UI at `/pipeline` for status visibility and artifact retrieval.

## High-Level Components

### 1) Web Client
- Route: `/pipeline`
- Responsibilities:
  - Capture one-line idea.
  - Drive clarify step.
  - Trigger generation.
  - Display concept state and artifact version metadata.
  - Allow copy/download of generated markdown.

### 2) API Service
- Endpoints:
  - `POST /api/pipeline/start`
  - `POST /api/pipeline/clarify`
  - `POST /api/pipeline/generate`
  - `GET /api/pipeline/concepts`
  - `GET /api/pipeline/concepts/:conceptId/artifacts/:type`
- Responsibilities:
  - Validate required fields.
  - Enforce deterministic generation rules.
  - Manage concept lifecycle state transitions.
  - Increment artifact versions.

### 3) Generator Engine (Deterministic)
- Inputs: structured concept fields only.
- Outputs: markdown for `prd`, `architecture`, and `execution-pack`.
- Constraint: no web/market/competitor data.
- Determinism rule: same structured input ⇒ same output body.

### 4) Persistence Layer
- Entities:
  - `Concept`
  - `Artifact`
- Requirements:
  - Append-only artifact versions.
  - Explicit version metadata (`v1`, `v2`, ...).
  - Latest-version pointer for quick retrieval.

## Data Model (Logical)

### Concept
- `id` (UUID)
- `rawIdea` (string)
- `productType` (string)
- `primaryOutcome` (string)
- `exclusions` (string[])
- `buildStrategy` (enum)
- `decisionStance` (enum)
- `status` (`draft | clarified | generated`)
- `createdAt`, `updatedAt`

### Artifact
- `id` (UUID)
- `conceptId` (UUID)
- `type` (`prd | architecture | execution-pack`)
- `version` (int)
- `contentMarkdown` (text)
- `createdAt`

## Lifecycle & State Transitions
1. `start` creates concept in `draft`.
2. `clarify` validates required fields and moves concept to `clarified`.
3. `generate` creates three markdown artifacts and moves concept to `generated`.
4. Re-generate appends new versions without deleting old versions.

## Validation Rules
- Generation requires non-empty:
  - `productType`
  - `primaryOutcome`
  - `exclusions`
- Reject invalid state transitions with clear error messages.

## Observability & Operations
- Log request IDs and concept IDs for tracing.
- Track generation success/failure counters.
- Monitor core flow endpoint error rates.
- Keep rollback path by preserving prior artifact versions.

## Security & Privacy (Phase 1)
- Basic server-side validation and sanitized markdown rendering.
- Avoid storing unnecessary sensitive personal health information.
- Restrict content scope to product artifact generation workflow.

## Deployment Notes
- Deploy as a single service for MVP.
- Use durable storage for artifact versioning.
- Keep migration path open for queued generation in Phase 2.
