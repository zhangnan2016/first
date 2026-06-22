## ADDED Requirements

### Requirement: Hero brand visual
The system SHALL display a full-width hero section at the top of the homepage with a brand background image, a tagline, a subtitle, and a search input placeholder.

#### Scenario: Hero displays brand content
- **WHEN** the homepage loads
- **THEN** the hero section displays a full-width background image
- **AND** displays the tagline "Unlock the Real China" prominently
- **AND** displays the subtitle "Curated English guides, instant AI answers, and a traveler community — everything you need to explore China with confidence."

### Requirement: Hero search placeholder
The system SHALL display a search input in the hero area as a visual placeholder. The search input SHALL NOT connect to any backend search functionality in the current release.

#### Scenario: Search input visual placeholder
- **WHEN** a user sees the search input in the hero area
- **THEN** the input displays placeholder text "Search destinations, guides, or ask anything…"
- **AND** submitting the search does not trigger a full-site search (out of scope)

#### Scenario: Search submit shows coming-soon feedback
- **WHEN** a user submits a query in the hero search input
- **THEN** the system displays a non-blocking message indicating search is coming soon
- **AND** no API call is made to a search backend

### Requirement: Hero responsive layout
The hero section SHALL be responsive and display correctly on mobile, tablet, and desktop viewports.

#### Scenario: Mobile viewport
- **WHEN** the homepage is viewed on a mobile device (width < 768px)
- **THEN** the hero section adjusts its height and font sizes for mobile readability
- **AND** the background image scales to cover the viewport without distortion
