## ADDED Requirements

### Requirement: Feature navigation cards
The system SHALL display a feature navigation section on the homepage with three entry cards: Travel Community, Attraction Guides, and AI Assistant. Each card SHALL display an icon, a title, and a one-line description.

#### Scenario: Display three feature cards
- **WHEN** the homepage loads
- **THEN** the section displays three cards: Travel Community, Attraction Guides, AI Assistant
- **AND** each card displays an icon, a title, and a short descriptive sentence

#### Scenario: Card content
- **WHEN** the feature navigation section renders
- **THEN** the Travel Community card displays: icon + "Travel Community" + "Share experiences and get real advice from fellow travelers"
- **AND** the Attraction Guides card displays: icon + "Attraction Guides" + "Explore curated guides for China's top destinations"
- **AND** the AI Assistant card displays: icon + "AI Assistant" + "Get instant answers about traveling in China, anytime"

### Requirement: Feature navigation links
Clicking a feature navigation card SHALL navigate to the corresponding module page. Some destinations may be placeholder pages in the current release.

#### Scenario: Navigate to community
- **WHEN** a user clicks the Travel Community card
- **THEN** the browser navigates to the community module route (e.g., `/community`)

#### Scenario: Navigate to attraction guides
- **WHEN** a user clicks the Attraction Guides card
- **THEN** the browser navigates to the attraction guides route (e.g., `/guides`)
- **AND** the destination page is a "Coming Soon" placeholder (P1 module not fully implemented in this release)

#### Scenario: Navigate to AI assistant
- **WHEN** a user clicks the AI Assistant card
- **THEN** the system opens the AI assistant chat interface (same behavior as the floating button)
- **OR** navigates to the AI assistant module route (e.g., `/assistant`)

### Requirement: Feature navigation responsive layout
The feature navigation cards SHALL be responsive and display in a grid that adapts to viewport size.

#### Scenario: Responsive card layout
- **WHEN** the homepage is viewed on desktop
- **THEN** the three cards display in a single row (3 columns)
- **AND** on mobile the cards stack vertically (1 column)
