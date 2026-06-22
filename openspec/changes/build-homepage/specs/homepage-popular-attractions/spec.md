## ADDED Requirements

### Requirement: Popular attractions section
The system SHALL display a "Popular Attractions" section on the homepage showing 6–8 attraction/city cards ranked by view count in descending order.

#### Scenario: Display popular attractions
- **WHEN** the homepage loads and the attraction API returns data
- **THEN** the section displays 6 to 8 attraction cards in descending order of `view_count`
- **AND** only active attractions (`status=1 AND deleted=0`) are included

#### Scenario: Each attraction card displays key info
- **WHEN** an attraction card is rendered
- **THEN** the card displays the attraction name (English), city name, a cover image, and a brief description
- **AND** clicking the card navigates to the attraction detail page (or placeholder page if detail is not yet implemented)

### Requirement: Popular attractions data source
The system SHALL fetch popular attractions from the attraction list API with a `view_count` sort parameter and a limit of 8.

#### Scenario: API request parameters
- **WHEN** the homepage requests popular attractions data
- **THEN** the request specifies sorting by `view_count` descending
- **AND** the request limits results to 8 items

#### Scenario: API dependency on view_count field
- **WHEN** the `attraction` table does not yet have the `view_count` field
- **THEN** the popular attractions section falls back to sorting by `popularity` descending (the existing preset field)
- **AND** the `view_count` field addition is a prerequisite for full functionality (see cross-module dependency R-T3)

### Requirement: Popular attractions empty state
The system SHALL display an appropriate empty-state message when no attractions are available.

#### Scenario: No attractions data
- **WHEN** the attraction API returns an empty list
- **THEN** the section displays a friendly empty-state message (e.g., "Attractions are coming soon!")
- **AND** the section does NOT display an error or blank container
