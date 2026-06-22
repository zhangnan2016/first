## ADDED Requirements

### Requirement: Homepage page shell
The system SHALL provide a homepage page that assembles content sections in a fixed order: Hero, Trending Posts, Popular Attractions, City Quick Access, and Feature Navigation. The page SHALL be rendered via SSR (Server Components) for SEO friendliness.

#### Scenario: Page renders all sections
- **WHEN** a visitor (including unauthenticated) opens the homepage
- **THEN** the page renders all content sections in order: Hero → Trending Posts → Popular Attractions → City Quick Access → Feature Navigation
- **AND** each section is independently loaded and rendered (a failed section SHALL NOT break the entire page)

#### Scenario: Loading state
- **WHEN** a data-dependent section is fetching data
- **THEN** the section displays a skeleton or loading placeholder
- **AND** the rest of the page renders normally without blocking

#### Scenario: Empty state fallback
- **WHEN** a data-dependent section receives an empty result set (e.g., no trending posts)
- **THEN** the section displays a friendly empty-state message instead of an empty container

#### Scenario: Error boundary per section
- **WHEN** a data-dependent section encounters an error (e.g., API timeout)
- **THEN** only that section displays an error fallback with a retry option
- **AND** other sections continue to render normally

### Requirement: Homepage SEO metadata
The system SHALL set appropriate SEO metadata on the homepage page, including page title, meta description, and Open Graph tags.

#### Scenario: SEO meta tags present
- **WHEN** the homepage HTML is served
- **THEN** the document includes a `<title>` tag with the site name and tagline
- **AND** includes a `<meta name="description">` tag with a concise English description
- **AND** includes Open Graph tags (`og:title`, `og:description`, `og:image`) for social sharing

### Requirement: Public access
The homepage SHALL be accessible to unauthenticated visitors without requiring login.

#### Scenario: Guest access
- **WHEN** an unauthenticated visitor navigates to the homepage
- **THEN** the page loads and displays all content without any login wall or redirect
- **AND** the homepage API endpoints are within the Sa-Token whitelist (public access)
