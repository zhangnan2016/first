# Frontend Scaffold

## Purpose

Defines the runnable Next.js 16 (App Router) + React 19 + TypeScript + Ant Design 6 frontend, the layout template, and per-environment configuration.

## Requirements

### Requirement: Runnable Next.js build
The frontend SHALL be a Next.js 16 project bootstrappable with `npm run dev` (development server) and buildable with `npm run build` (production bundle) without errors. The project SHALL use the App Router with React 19, TypeScript (strict mode), and Ant Design 6. State management SHALL use Zustand for client-side state only; token management SHALL use httpOnly cookies, not client stores.

#### Scenario: Dev server starts
- **WHEN** `npm run dev` runs in `frontend/`
- **THEN** the Next.js dev server starts and serves the application with SSR enabled on a local port

#### Scenario: Production build succeeds
- **WHEN** `npm run build` runs in `frontend/`
- **THEN** a production build is emitted, optimized for Vercel deployment

### Requirement: Layout template
The frontend SHALL provide a layout with a sidebar, a header, a breadcrumb, and a tags-view. Authenticated pages SHALL render inside this layout. The layout SHALL be implemented as a React component tree using Ant Design components, with Server Components for static regions and Client Components for interactive regions.

#### Scenario: Layout renders regions
- **WHEN** an authenticated page is displayed
- **THEN** the sidebar, header, breadcrumb, and tags-view regions are all present

### Requirement: Environment configuration
The frontend SHALL support per-environment configuration via `.env.development` and `.env.production` (or `.env.local`), including a base API URL exposed as `NEXT_PUBLIC_API_BASE_URL` for client-side access and a server-only `API_BASE_URL` for Server Components and Route Handlers.

#### Scenario: Dev points to local backend
- **WHEN** the app runs in development mode
- **THEN** `NEXT_PUBLIC_API_BASE_URL` and `API_BASE_URL` resolve to the local backend address
