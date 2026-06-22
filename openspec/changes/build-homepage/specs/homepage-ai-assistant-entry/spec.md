## ADDED Requirements

### Requirement: AI assistant floating entry
The system SHALL display a floating action button (FAB) on the homepage that allows users to summon the AI assistant at any time. The button SHALL be visible across the entire homepage and persistent during scroll.

#### Scenario: Floating button visible
- **WHEN** a user views the homepage at any scroll position
- **THEN** a floating AI assistant button is visible in a fixed position (e.g., bottom-right corner)
- **AND** the button displays an AI/chat icon that is distinguishable from other UI elements

#### Scenario: Floating button accessibility
- **WHEN** a user hovers over the floating button
- **THEN** a tooltip or label appears (e.g., "Ask AI")
- **AND** the button has an appropriate `aria-label` for screen readers

### Requirement: AI assistant invocation
Clicking the floating AI assistant button SHALL open the AI assistant chat interface. The behavior depends on the user's authentication status.

#### Scenario: Authenticated user invokes AI
- **WHEN** an authenticated user clicks the floating AI assistant button
- **THEN** the AI assistant chat interface opens (modal or side panel)
- **AND** the user can immediately start a conversation

#### Scenario: Unauthenticated user invokes AI
- **WHEN** an unauthenticated user clicks the floating AI assistant button
- **THEN** the system redirects the user to the login page or displays a login prompt
- **AND** the AI assistant chat interface is not opened until authentication is complete

### Requirement: AI assistant entry global placement
The floating AI assistant button SHALL be mounted at the global layout level, not within the homepage shell, so it persists across page navigations (if applicable) and is not tied to a single content section.

#### Scenario: Button independent of homepage shell
- **WHEN** the floating AI assistant button is rendered
- **THEN** it is a child of the global layout component, not the homepage shell
- **AND** its visibility is controlled independently from homepage content sections
