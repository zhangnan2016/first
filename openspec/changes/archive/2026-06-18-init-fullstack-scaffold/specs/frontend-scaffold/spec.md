## ADDED Requirements

### Requirement: Runnable Vite build
The frontend SHALL be a Vite project bootstrappable with `npm run dev` (development server) and buildable with `npm run build` (production bundle) without errors.

#### Scenario: Dev server starts
- **WHEN** `npm run dev` runs in `frontend/`
- **THEN** the Vite dev server starts and serves the application on a local port

#### Scenario: Production build succeeds
- **WHEN** `npm run build` runs in `frontend/`
- **THEN** a production bundle is emitted to the configured output directory

### Requirement: Layout template
The frontend SHALL provide a layout with a sidebar, a header, a breadcrumb, and a tags-view. Authenticated pages SHALL render inside this layout.

#### Scenario: Layout renders regions
- **WHEN** an authenticated page is displayed
- **THEN** the sidebar, header, breadcrumb, and tags-view regions are all present

### Requirement: Environment configuration
The frontend SHALL support per-environment configuration via `.env.development` and `.env.production`, including a base API URL (`VITE_API_BASE_URL`).

#### Scenario: Dev points to local backend
- **WHEN** the app runs in development mode
- **THEN** `VITE_API_BASE_URL` resolves to the local backend address
