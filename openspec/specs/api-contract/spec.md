# API Contract

## Purpose

Defines the unified response envelope, error code enumeration, global exception handling, and pagination structure shared by all backend endpoints and consumed by the frontend.

## Requirements

### Requirement: Unified response envelope
Every HTTP endpoint SHALL return a body shaped as `R<T>` with fields `code` (integer), `msg` (string), and `data` (generic payload). `code` equal to `0` SHALL indicate success; any non-zero value SHALL indicate an error.

#### Scenario: Successful response shape
- **WHEN** any controller endpoint completes successfully
- **THEN** the serialized JSON body contains `code`, `msg`, and `data` keys, with `code` equal to `0`

#### Scenario: Error code semantics
- **WHEN** a business error occurs
- **THEN** `code` is a non-zero value sourced from `ResultCode` and `data` is null

### Requirement: ResultCode enumeration
The system SHALL define a `ResultCode` enumeration mapping every non-zero code to a human-readable message. Business and system errors SHALL reference these codes exclusively.

#### Scenario: Known code resolves to message
- **WHEN** a `ResultCode` value is looked up
- **THEN** it exposes both its numeric `code` and its `msg`

### Requirement: Global exception handling
A single `@RestControllerAdvice` SHALL translate exceptions into `R<Void>` responses so the frontend always receives a parsable envelope. Validation errors, business errors (`BizException`), and uncaught errors SHALL each be mapped to an appropriate `ResultCode`.

#### Scenario: Validation error is mapped
- **WHEN** a `MethodArgumentNotValidException` is thrown
- **THEN** the response body is `R<Void>` with a validation-related `ResultCode` and the field error message in `msg`

#### Scenario: Business exception is mapped
- **WHEN** a `BizException` carrying a `ResultCode` is thrown
- **THEN** the response body is `R<Void>` carrying that same `ResultCode`

#### Scenario: Unknown error is handled gracefully
- **WHEN** an uncaught exception propagates to the advice
- **THEN** the response is a `R<Void>` with a generic server-error code, and the stack trace is not leaked to the client

### Requirement: PageResult pagination envelope
Pagination responses SHALL use `PageResult<T>` exposing `records`, `total`, `current`, and `size`. List endpoints accepting page parameters SHALL return this envelope inside `R<PageResult<T>>`.

#### Scenario: Paginated list returns full metadata
- **WHEN** a list endpoint is called with `current` and `size` parameters
- **THEN** `data` contains `records`, `total`, `current`, and `size`

#### Scenario: Pagination parameters are bounded
- **WHEN** `current` is 0, negative, or omitted
- **THEN** it SHALL default to 1; values < 1 SHALL be clamped to 1, not rejected
- **WHEN** `size` exceeds the maximum page size (200)
- **THEN** it SHALL be silently truncated to 200 to prevent OOM; values ≤ 0 SHALL default to 10

### Requirement: Input validation constraints
All DTO string fields SHALL declare `@Size` constraints matching the corresponding entity column lengths. Numeric fields SHALL declare `@Min`/`@Max` where applicable. Validation failures SHALL be caught by the global exception handler and mapped to `PARAM_ERROR`. Unvalidated fields reaching the database layer indicate a spec violation.

#### Scenario: Oversized input is rejected
- **WHEN** a client submits a field value exceeding its declared `@Size` limit
- **THEN** the request is rejected with `PARAM_ERROR` before reaching the persistence layer

### Requirement: Idempotency for mutating operations
Create and update endpoints SHALL be idempotent at the business-key level: duplicate submissions with the same business key SHALL NOT produce duplicate side effects. This SHALL be achieved via database unique constraints (create) or idempotency keys (update).

#### Scenario: Duplicate create is rejected
- **WHEN** two concurrent or sequential create requests submit the same unique business key (e.g. username)
- **THEN** the second request SHALL fail with `DATA_DUPLICATED`, not create a duplicate record
