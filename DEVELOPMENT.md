# Development Guide — Frontend SPA (React)

This document explains how to set up, run, test, and contribute to the React-based frontend for the **copilot-sdlc-backend** system.

> Source of truth for requirements (TRD) lives in a different repository:
> - Repo: `Randomme07/ai-sdlc`
> - Path: `docs/trd/`
> - Primary TRD: `docs/trd/trd-car-management-lifecycle.md`

---

## 1) Prerequisites

### Required
- **Node.js (LTS)** (recommended: install via `nvm`)
- **pnpm** (preferred package manager)

### Recommended
- VS Code
- Extensions:
  - ESLint
  - Prettier
  - EditorConfig
  - Vitest (optional)
  - Tailwind CSS IntelliSense (only if using Tailwind)

---

## 2) Local setup

### 2.1 Install dependencies
```bash
pnpm install
```

### 2.2 Configure environment variables
Create a `.env.local` file in the project root:

```bash
# Backend base URL (example)
VITE_API_BASE_URL=http://localhost:8080
```

If the backend expects a user identifier header (MVP mode), configure:
```bash
VITE_X_USER_ID=dev-fleet-manager
```

> Do not commit `.env.local`. Use `.env.example` for documented defaults.

---

## 3) Run locally
Start the dev server:
```bash
pnpm dev
```

The terminal output will show the local URL (typically `http://localhost:5173`).

---

## 4) API integration notes

### 4.1 Endpoint base path
All backend APIs are versioned:
- `/api/v1/...`

The frontend must follow the TRD contracts and should not invent fields or endpoints.

### 4.2 Error envelope (TRD)
Backend errors follow:
```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": "object | null"
  }
}
```

Frontend code should parse this envelope and display meaningful user feedback.

### 4.3 Actor header (if required)
If the backend requires `X-User-Id`, the frontend should include it from a single centralized place (API client interceptor/wrapper), using `VITE_X_USER_ID` in dev.

---

## 5) Testing

### 5.1 Unit tests
Run unit tests:
```bash
pnpm test
```

If the project supports watch mode:
```bash
pnpm test --watch
```

### 5.2 What to test (minimum)
- Vehicle onboarding form (validation + submit behavior)
- Lifecycle transition UI (available actions per current lifecycle status)
- Error handling (TRD envelope parsing and messaging)

> Prefer React Testing Library + Vitest. Use MSW for API mocking when possible.

---

## 6) Linting & formatting

### 6.1 Lint
```bash
pnpm lint
```

### 6.2 Format (if configured)
```bash
pnpm format
```

### 6.3 Typecheck
```bash
pnpm typecheck
```

---

## 7) Build
Create a production build:
```bash
pnpm build
```

Preview the build locally (if supported):
```bash
pnpm preview
```

---

## 8) Project structure (recommended)
A clean, feature-oriented structure is recommended:

- `src/app/` — app bootstrap, routing, providers
- `src/api/` — API client + typed endpoint functions
- `src/features/` — feature modules (vehicles, lifecycle)
- `src/components/` — shared UI components
- `src/hooks/` — reusable hooks
- `src/utils/` — utilities (formatters, guards)
- `src/test/` — test utilities, mocks

---

## 9) Contribution workflow

### 9.1 Branching
Use short-lived branches:
- `feat/...`
- `fix/...`
- `chore/...`

### 9.2 PR title (Conventional Commits)
PR titles must follow Conventional Commits:
- `feat: ...`
- `fix: ...`
- `docs: ...`
- `test: ...`
- `refactor: ...`
- `chore: ...`

### 9.3 PR checklist
Before opening a PR:
- [ ] Requirements match TRD (`Randomme07/ai-sdlc/docs/trd/...`)
- [ ] `pnpm test` passes
- [ ] `pnpm lint` passes
- [ ] `pnpm typecheck` passes
- [ ] UI includes loading/error/empty states where applicable

---

## 10) Troubleshooting

### Backend API not reachable
- Confirm backend is running
- Check `.env.local` `VITE_API_BASE_URL`
- Verify CORS settings on backend (may require backend configuration)

### 409 conflicts during onboarding
- TRD requires VIN and license plate uniqueness
- Try a new VIN/license plate or reset the database (dev environment)

### Invalid lifecycle transition errors
- UI must only allow transitions defined in TRD state machine
- Refresh vehicle details and retry

---

## 11) TRD references
- Vehicle Onboarding & Lifecycle Management TRD:
  - `Randomme07/ai-sdlc/docs/trd/trd-car-management-lifecycle.md`
- DB design (backend reference for fields/constraints):
  - `Randomme07/ai-sdlc/docs/trd/database-design-car-management-lifecycle.md`
