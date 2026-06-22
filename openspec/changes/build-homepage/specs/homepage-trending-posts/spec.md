## ADDED Requirements

### Requirement: Trending posts section
The system SHALL display a "Trending Posts" section on the homepage showing the top 10 community posts ranked by upvote count from the past 7 days.

#### Scenario: Display trending posts
- **WHEN** the homepage loads and the community post API returns data
- **THEN** the section displays up to 10 post cards in descending order of `upvote_count`
- **AND** only posts from the past 7 days (based on `create_time`) are included
- **AND** only published posts (`status=1 AND deleted=0`) are included

#### Scenario: Each post card displays key info
- **WHEN** a trending post card is rendered
- **THEN** the card displays the post title, author name, city tag, upvote count, and comment count
- **AND** clicking the card navigates to the post detail page

### Requirement: Trending posts data source
The system SHALL fetch trending posts from the community post list API with parameters specifying 7-day range, `upvote_count` sort order, and a limit of 10.

#### Scenario: API request parameters
- **WHEN** the homepage requests trending posts data
- **THEN** the request includes a date-range filter for posts created within the last 7 days
- **AND** the request specifies sorting by `upvote_count` descending
- **AND** the request limits results to 10 items

#### Scenario: API dependency on upvote_count field
- **WHEN** the community `post` table does not yet have the `upvote_count` field
- **THEN** the trending posts section falls back to sorting by `create_time` descending and displays a warning in development mode
- **AND** the `upvote_count` field addition is a prerequisite for full functionality (see cross-module dependency R-T2)

### Requirement: Trending posts empty state
The system SHALL display an appropriate empty-state message when no trending posts are available.

#### Scenario: No posts in last 7 days
- **WHEN** the community API returns an empty list for the 7-day trending query
- **THEN** the section displays a friendly empty-state message (e.g., "Be the first to share your China travel story!")
- **AND** the section does NOT display an error or blank container
