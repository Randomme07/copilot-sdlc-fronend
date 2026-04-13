
# Copilot Instructions — Frontend SPA (React)

These instructions apply to all contributions to the **frontend SPA** for the `copilot-sdlc-backend` system.

---

## 1) Source of truth (TRD is in a different repository)
All UI behavior, workflows, fields, and API contracts must follow the TRDs stored in the separate repository:

- **Repo:** `Randomme07/ai-sdlc`
- **Path:** `docs/trd/`
- Primary TRD for Vehicle Onboarding & Lifecycle:
  - `docs/trd/trd-car-management-lifecycle.md`
- DB design TRD is backend-focused, but UI must reflect field constraints (required/unique) from TRD.

**Rule:** If a UI requirement is missing/unclear, do **not** guess—propose a TRD update (or a UI requirement doc) first.

---

## 2) Frontend technology standards (required)
- Framework: **React** (Single Page Application)
- Language: **TypeScript**
- Node.js: use **current LTS** (pin in `.nvmrc` or `engines` in `package.json`)
- Package manager: **pnpm** preferred (use npm/yarn only if repo standard requires)
- Build tool: **Vite** preferred (CRA only if already mandated)

### Styling
- Prefer a single approach consistently:
  - CSS Modules **or** Tailwind **or** styled-components
- Do not mix styling systems without a strong reason.

### State & data fetching
- Prefer **TanStack Query (React Query)** for server state (fetching/caching/retries)
- Prefer local component state for UI-only state
- If global state is needed, use a small and explicit solution (Context/Zustand). Avoid Redux unless mandated.

---

## 3) Architecture / folder structure (keep consistent)
Use a clean, feature-oriented structure:

- `src/app/` — app bootstrap, routing, providers, query client, global config
- `src/features/` — feature modules (vehicle onboarding, lifecycle management, etc.)
- `src/components/` — reusable UI components (buttons, inputs, tables)
- `src/api/` — API client, typed endpoints, DTOs
- `src/hooks/` — reusable hooks
- `src/utils/` — helpers (formatters, guards)
- `src/types/` — shared types (if not generated)
- `src/test/` — test utilities, mocks

**Rules**
- No business rules inside UI components if it can be centralized (e.g., lifecycle transition validation should be derived from TRD and centralized).
- Keep “smart” logic in feature containers/hooks; keep components mostly presentational.

---

## 4) Backend integration rules (must match TRD)
### Base API
- All API calls must target backend REST endpoints defined in TRD (e.g., `/api/v1/...`).
- Do not invent endpoints or fields.
- Use a single API client wrapper (fetch/axios) with:
  - base URL from environment variable (e.g., `VITE_API_BASE_URL`)
  - JSON default headers
  - standardized error parsing

### Required endpoints (Vehicle Lifecycle feature)
Implement UI flows based on these TRD endpoints:
- `POST /api/v1/vehicles` — register vehicle
- `GET /api/v1/vehicles/{vehicleId}` — vehicle detail
- `PUT /api/v1/vehicles/{vehicleId}/lifecycle-status` — transition status
- `GET /api/v1/vehicles/{vehicleId}/lifecycle-history` — audit history

### Error contract (must honor)
Backend errors use this envelope (TRD):
```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": "object | null"
  }
}
```

**Frontend rule:** Always display meaningful user feedback based on:
- `error.message` (human readable)
- optionally map known `error.code` to specific UI messages/actions

---

## 5) UI requirements & behavior (TRD-aligned)
### Vehicle onboarding form
- Must collect all required fields from TRD request schema for `POST /api/v1/vehicles`.
- Perform basic client-side validation (required fields, types, simple constraints), but always rely on server validation as source of truth.
- Handle `409 DUPLICATE_VEHICLE` with a clear UI message indicating VIN/license plate conflict.

### Lifecycle status transitions
- UI must only offer transitions allowed by TRD state machine:
  - `Incoming -> Active`
  - `Active -> Maintenance | Decommissioning`
  - `Maintenance -> Active | Decommissioning`
  - `Decommissioning -> Sold`
  - `Sold -> none`
- If backend returns `400 INVALID_LIFECYCLE_TRANSITION`, show error and refresh vehicle details.

### Lifecycle history
- Display lifecycle history in reverse chronological order.
- Show: fromStatus, toStatus, transitionedAt, transitionedBy, reason (when present)

---

## 6) Security / identity assumptions
Until authentication is implemented end-to-end:
- Treat the frontend as operating in **MVP mode**.
- If backend expects an actor identifier (e.g., `X-User-Id`), include it consistently from a single source:
  - configurable dev value (env var), or
  - temporary UI input in a dev-only settings panel.

Do not hardcode identity values across multiple components.

---

## 7) Quality bar (non-negotiable)
### Clean code
- Prefer small components, clear names, and typed props.
- No duplicated API calls scattered across components—centralize in `src/api/` and use hooks.

### Testing
- Unit tests: **Vitest + React Testing Library**
- Add tests for:
  - onboarding form validation + submit behavior
  - lifecycle transition UI logic (available actions per status)
  - error handling (TRD envelope parsing and UI messaging)
- Mock API calls with **MSW** when possible.

### Accessibility & UX
- Use semantic HTML and accessible form labels.
- Loading states, empty states, and error states must be implemented for every page that fetches data.

---

## 8) Pull request / change management
### Conventional commits (required)
PR title must follow Conventional Commits:
- `feat: ...`
- `fix: ...`
- `docs: ...`
- `test: ...`
- `refactor: ...`
- `chore: ...`

### PR description must include
- TRD reference (file path in `Randomme07/ai-sdlc/docs/trd/...`)
- Summary of UI change
- Test evidence (what you ran)

---

## 9) What NOT to do
- Do not add new libraries unless necessary and approved (keep dependencies minimal).
- Do not change API contracts or create frontend-only “fake” fields that diverge from TRD.
- Do not bypass the standard error handling.
- Do not merge without tests and basic lint/typecheck passing.

---

## 10) Development scripts (recommended to include in project)
Ensure the project supports:
- `pnpm dev` — run locally
- `pnpm test` — run unit tests
- `pnpm lint` — lint
- `pnpm typecheck` — TS type check
- `pnpm build` — production build
