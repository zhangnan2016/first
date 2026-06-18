# Database Design Conventions

## Purpose

Defines the cross-cutting database schema design conventions for all tables: primary key strategy, mandatory common columns, nullability decision rules, and index design strategy. This spec governs **table-level (DDL) design decisions**; the JPA/Hibernate implementation of primary key generation, audit field auto-fill, and logical delete is governed by the `data-access-layer` spec and SHALL NOT be duplicated here.

## Requirements

### Requirement: Primary key strategy SHALL be Snowflake BIGINT

All business tables SHALL use a single `id` column of type `BIGINT` as the primary key, populated by the Snowflake algorithm. Auto-increment (`AUTO_INCREMENT`) and UUID primary keys SHALL NOT be used on any table.

The Snowflake strategy is chosen over alternatives for the following reasons:

| Strategy | Verdict | Rationale |
|----------|---------|-----------|
| Auto-increment | Forbidden | Single-point write bottleneck; IDs are enumerable (security risk); incompatible with horizontal sharding |
| UUID | Forbidden | Fully random → B+tree page splits and index fragmentation; 16-byte storage; poor readability |
| Snowflake | **Required** | Trend-increasing (index-friendly); distributed-safe; 8-byte compact storage |

#### Scenario: High-volume write tables use trend-increasing IDs
- **WHEN** a conversation or message table receives frequent inserts
- **THEN** new rows are appended near the end of the primary key B+tree index, minimizing page splits and write amplification

#### Scenario: Sharding-ready unique IDs
- **WHEN** a table is horizontally split across multiple database instances
- **THEN** Snowflake IDs remain globally unique without relying on any single-table sequence or cross-node coordination

#### Scenario: Frontend precision safety
- **WHEN** a Snowflake ID exceeds JavaScript's `Number.MAX_SAFE_INTEGER`
- **THEN** the API layer serializes the `Long` value as a JSON string to prevent precision loss (implementation detail owned by `data-access-layer`)

### Requirement: Mandatory common columns on every business table

Every business table SHALL contain the following common columns. Tables modeled by the `BaseEntity` `@MappedSuperclass` inherit them automatically.

| Column | Type | Nullability | Default | Responsibility |
|--------|------|-------------|---------|----------------|
| `id` | BIGINT | NOT NULL | — | Primary key (Snowflake) |
| `create_time` | DATETIME | NOT NULL | — | Row creation timestamp |
| `update_time` | DATETIME | NOT NULL | — | Last update timestamp |
| `create_by` | BIGINT | NULL | NULL | Creator user ID |
| `update_by` | BIGINT | NULL | NULL | Last updater user ID |
| `deleted` | TINYINT | NOT NULL | 0 | Logical delete flag: 0=active, 1=deleted |
| `version` | INT | NOT NULL | 0 | Optimistic lock version (mandatory only on concurrently-modified tables) |

#### Scenario: Audit columns are non-null and auto-filled
- **WHEN** a row is inserted or updated
- **THEN** `create_time` and `update_time` are populated automatically and are never NULL

#### Scenario: Creator columns tolerate absent session context
- **WHEN** a row is created by a background job, data seeding, or anonymous operation with no authenticated user
- **THEN** `create_by` and `update_by` remain NULL rather than forcing a write failure

#### Scenario: Version column is optional for low-contention tables
- **WHEN** a table is not subject to concurrent modification conflicts
- **THEN** it MAY omit the `version` column to avoid unnecessary version-check overhead

### Requirement: Nullability defaults to NOT NULL

Columns SHALL be declared `NOT NULL` with a sensible default value unless the field genuinely represents an "absent/unknown/not-yet-produced" state that has no meaningful default. NULL introduces three-valued logic that breaks equality, `COUNT()`, `ORDER BY`, and index matching, so it SHALL be avoided whenever a default can express the intended empty state.

#### Scenario: Flag and status columns are never nullable
- **WHEN** a column represents a boolean flag, status code, or logical-delete marker
- **THEN** it is declared `NOT NULL DEFAULT 0`

#### Scenario: String columns use empty string, not NULL
- **WHEN** a column represents a value that may be empty but not "unknown" (e.g. a display name)
- **THEN** it is declared `NOT NULL DEFAULT ''` rather than allowing NULL

#### Scenario: Counter and numeric columns default to zero
- **WHEN** a column represents a count or numeric metric
- **THEN** it is declared `NOT NULL DEFAULT 0`

#### Scenario: NULL is permitted only with explicit business semantics
- **WHEN** a column represents a value that may genuinely not exist or not yet exist (e.g. `last_login_time`, a `parent_id` for top-level rows, a `tokens_used` metric available only after streaming completes)
- **THEN** NULL is permitted, and the column documents why no default is applicable

### Requirement: Index design follows selectivity and access patterns

Indexes SHALL be added only when justified by query access patterns. Each index has a write-cost and storage cost, so indexes SHALL be deliberate and minimal.

Indexes SHALL be created when **any** of the following holds:
- The column is a high-frequency `WHERE` filter
- The column is a `JOIN` key or foreign-key reference
- The column drives `ORDER BY` or `GROUP BY`
- The column requires uniqueness enforcement (unique index)

Indexes SHALL NOT be created when **any** of the following holds:
- The table is write-heavy and read-light (e.g. raw operation logs)
- The column has very low selectivity (e.g. `deleted`, `status` with only 0/1) as a standalone index
- The column is updated frequently
- The table is small enough that a full scan is faster

#### Scenario: Composite index leads with the most selective column
- **WHEN** a query filters by `conversation_id` and sorts by `create_time`
- **THEN** a composite index `INDEX(conversation_id, create_time)` is created, leading with the high-selectivity key

#### Scenario: Low-selectivity columns are never indexed alone
- **WHEN** the `deleted` column has only values 0 and 1
- **THEN** it is not given a standalone index; it MAY appear only as part of a composite unique index such as `UNIQUE(username, deleted)`

#### Scenario: Index count is bounded per table
- **WHEN** a table's index count is reviewed
- **THEN** it contains no more than 5 indexes unless justified by a documented query-pattern analysis

#### Scenario: Long string columns use prefix indexes
- **WHEN** a long `VARCHAR` column (e.g. a URL) needs indexing
- **THEN** a prefix index is used (e.g. `INDEX(url(50))`) to balance lookup speed and index size

### Requirement: Typical AI-conversation tables follow canonical indexing

The core conversation-domain tables SHALL be indexed according to their dominant access patterns.

| Table | Index | Purpose |
|-------|-------|---------|
| `conversation` | `INDEX(user_id, update_time)` | List a user's conversations ordered by most recent |
| `message` | `INDEX(conversation_id, create_time)` | Load messages of a conversation in chronological order |
| `sys_user` | `UNIQUE(username, deleted)` | Uniqueness that tolerates logical delete and re-registration |

#### Scenario: Conversation list query uses a covering composite index
- **WHEN** the dashboard fetches a user's conversations sorted by last activity
- **THEN** the `(user_id, update_time)` composite index serves both the filter and the sort without a filesort

#### Scenario: Message loading is served by a single composite index
- **WHEN** a conversation view loads its messages page by page in time order
- **THEN** the `(conversation_id, create_time)` composite index serves the filter and the ordering together
