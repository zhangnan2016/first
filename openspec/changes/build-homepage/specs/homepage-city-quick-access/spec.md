## ADDED Requirements

### Requirement: City quick access grid
The system SHALL display a city quick access section on the homepage as an icon grid of 8 MVP cities. Clicking a city icon SHALL navigate to the corresponding city guide page.

#### Scenario: Display 8 city icons
- **WHEN** the homepage loads
- **THEN** the section displays 8 city icons in a grid layout: Beijing, Shanghai, Chengdu, Xi'an, Guangzhou, Shenzhen, Hangzhou, Guilin
- **AND** each city displays its English name and a representative icon or image

#### Scenario: City grid responsive layout
- **WHEN** the homepage is viewed on different viewport sizes
- **THEN** on desktop the grid displays 4 columns
- **AND** on tablet the grid displays 3 columns
- **AND** on mobile the grid displays 2 columns

### Requirement: City navigation to guide page
Clicking a city icon SHALL navigate to the corresponding city guide page. In the current release, city guide pages are placeholder "Coming Soon" pages.

#### Scenario: Click city navigates to placeholder
- **WHEN** a user clicks a city icon (e.g., Beijing)
- **THEN** the browser navigates to the city guide route (e.g., `/guides/beijing`)
- **AND** the destination page displays a "Coming Soon" placeholder with a back-to-home link
- **AND** the page returns HTTP 200 (not 404, for SEO friendliness)

### Requirement: Static city data
The 8 MVP city entries SHALL be defined as static configuration data, not requiring an API call.

#### Scenario: City data is static
- **WHEN** the city quick access section renders
- **THEN** the 8 cities are rendered from a static configuration constant (city name, city code, icon)
- **AND** no network request is made to fetch the city list (the `city` dictionary may be used for validation but is not required for rendering)
