## ADDED Requirements

### Requirement: MyBatis-Plus plugin configuration
The data access layer SHALL register MyBatis-Plus pagination, optimistic-lock, and logical-delete plugins. Logical delete SHALL be driven by a configured deleted-flag column.

#### Scenario: Pagination plugin is active
- **WHEN** a mapper executes a paged query with `current` and `size`
- **THEN** the returned page reports the correct `total` and limits results to `size`

#### Scenario: Logical delete soft-removes rows
- **WHEN** a `deleteById` is invoked on an entity with a logical-delete field
- **THEN** the row's deleted flag is updated instead of being physically removed, and subsequent selects exclude it

### Requirement: BaseEntity audit fields
Entities SHALL extend a `BaseEntity` providing `id`, `createTime`, `updateTime`, and `createBy`. A `MetaObjectHandler` SHALL auto-fill `createTime` and `createBy` on insert, and `updateTime` on update.

#### Scenario: Insert fills audit fields
- **WHEN** a new entity is inserted without explicit audit values
- **THEN** `createTime` and `createBy` are populated automatically by the fill handler

#### Scenario: Update refreshes update time
- **WHEN** an existing entity is updated
- **THEN** `updateTime` is refreshed automatically

### Requirement: BaseMapperX common queries
The data access layer SHALL provide a `BaseMapperX` extending MyBatis-Plus `BaseMapper` with reusable query helpers. Business mappers SHALL extend `BaseMapperX` to inherit these helpers.

#### Scenario: Business mapper inherits helpers
- **WHEN** a business mapper extends `BaseMapperX`
- **THEN** it exposes the common query helpers without redefining them
