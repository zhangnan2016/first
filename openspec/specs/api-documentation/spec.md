# API Documentation

## Purpose

Defines the Knife4j (OpenAPI 3) aggregated documentation UI, its reachability in non-production profiles, and production access control.

## Requirements

### Requirement: Knife4j documentation UI
The backend SHALL expose a Knife4j (OpenAPI 3) documentation UI aggregating all controllers. The UI SHALL be reachable at a configured path in every non-production profile.

#### Scenario: Documentation UI is reachable
- **WHEN** a developer opens the configured Knife4j path in a browser
- **THEN** the aggregated API documentation UI loads with all controller endpoints listed

#### Scenario: Endpoints carry documentation
- **WHEN** a controller endpoint is viewed in the UI
- **THEN** its summary, parameters, and response schema are rendered

### Requirement: Documentation access control
In production the documentation UI SHALL be completely disabled (Knife4j `enabled: false`); it SHALL NOT be accessible via any URL. If runtime access is required, it SHALL require Basic Auth credentials sourced from environment variables, not hardcoded. In non-production it SHALL be open to developers.

#### Scenario: Production hides the UI
- **WHEN** the application runs with the production profile
- **THEN** the Knife4j UI is not exposed unless explicitly enabled
