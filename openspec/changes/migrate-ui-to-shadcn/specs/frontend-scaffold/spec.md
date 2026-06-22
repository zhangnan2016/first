## MODIFIED Requirements

### Requirement: Runnable Next.js build
The frontend SHALL be a Next.js 16 project bootstrappable with `npm run dev` (development server) and buildable with `npm run build` (production bundle) without errors. The project SHALL use the App Router with React 19, TypeScript (strict mode), Tailwind CSS v4 for styling, and shadcn/ui for component primitives. State management SHALL use Zustand for client-side state only; token management SHALL use httpOnly cookies, not client stores. The project SHALL NOT use Ant Design (`antd`), `@ant-design/icons`, or `@ant-design/nextjs-registry`.

#### Scenario: Dev server starts
- **WHEN** `npm run dev` runs in `frontend/`
- **THEN** the Next.js dev server starts and serves the application with SSR enabled on a local port
- **AND** Tailwind CSS utility classes are applied correctly (styles are visible)

#### Scenario: Production build succeeds
- **WHEN** `npm run build` runs in `frontend/`
- **THEN** a production build is emitted, optimized for Vercel deployment
- **AND** no Ant Design imports remain in the bundle (verified by absence of `antd` in import graph)

#### Scenario: No Ant Design dependencies
- **WHEN** the project dependencies are inspected (`package.json`)
- **THEN** `antd`, `@ant-design/icons`, and `@ant-design/nextjs-registry` are NOT present in dependencies
- **AND** `tailwindcss`, `class-variance-authority`, `clsx`, `tailwind-merge`, and `lucide-react` ARE present

### Requirement: Layout template
The frontend SHALL provide a layout with a sidebar, a header, a breadcrumb, and a tags-view. Authenticated pages SHALL render inside this layout. The layout SHALL be implemented as a React component tree using shadcn/ui components and Tailwind CSS for styling, with Server Components for static regions and Client Components for interactive regions.

#### Scenario: Layout renders regions
- **WHEN** an authenticated page is displayed
- **THEN** the sidebar, header, breadcrumb, and tags-view regions are all present
- **AND** all layout styling uses Tailwind CSS utility classes (no inline styles for layout structure)

#### Scenario: Sidebar collapse on mobile
- **WHEN** the layout is viewed on a mobile device (width < 768px)
- **THEN** the sidebar is hidden by default and toggled via a Sheet (slide-out drawer) component
- **AND** a hamburger menu button is visible in the header to toggle the sidebar

#### Scenario: User dropdown menu
- **WHEN** the user clicks the user info area in the header
- **THEN** a dropdown menu appears with the logout option
- **AND** the logout confirmation uses a Dialog component (not `Modal.confirm`)

### Requirement: Toast notifications
The system SHALL provide toast notifications for user feedback (success, error, info messages) using the `sonner` library, replacing Ant Design's `message` API. The toast container SHALL be mounted at the root layout level.

#### Scenario: Error toast on API failure
- **WHEN** an API request fails (non-401 error)
- **THEN** an error toast is displayed with the error message
- **AND** the toast uses `sonner`'s `toast.error()` function (not Ant Design `message.error()`)

#### Scenario: Success toast on login
- **WHEN** a user successfully logs in
- **THEN** a success toast is displayed with "登录成功" message
- **AND** the toast uses `sonner`'s `toast.success()` function

### Requirement: Environment configuration
The frontend SHALL support per-environment configuration via `.env.development` and `.env.production` (or `.env.local`), including a base API URL exposed as `NEXT_PUBLIC_API_BASE_URL` for client-side access and a server-only `API_BASE_URL` for Server Components and Route Handlers.

#### Scenario: Dev points to local backend
- **WHEN** the app runs in development mode
- **THEN** `NEXT_PUBLIC_API_BASE_URL` and `API_BASE_URL` resolve to the local backend address

### Requirement: Design token system
The frontend SHALL define a design token system using CSS custom properties (CSS variables) in `globals.css`, compatible with shadcn/ui's theming approach. The tokens SHALL include color palette (background, foreground, primary, muted, destructive, border, ring), border radius, and typography scales for both light and dark modes.

#### Scenario: CSS variables defined
- **WHEN** `globals.css` is inspected
- **THEN** `:root` contains shadcn/ui-compatible CSS variables (`--background`, `--foreground`, `--primary`, etc.)
- **AND** `.dark` class overrides these variables for dark mode support

#### Scenario: Tailwind integration
- **WHEN** Tailwind CSS is configured
- **THEN** `@theme inline` maps the CSS variables to Tailwind utility classes (e.g., `bg-background`, `text-foreground`)
