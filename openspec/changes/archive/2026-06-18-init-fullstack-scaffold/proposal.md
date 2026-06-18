## Why

This repository is greenfield — it contains no application code. Teams starting new fullstack services repeatedly rebuild the same cross-cutting infrastructure (unified API contract, data access, auth, logging, frontend request layer). This change establishes a reusable, framework-grade fullstack scaffold so every future service starts from a battle-tested baseline, front and back evolve independently, and the front/back contract stays aligned from day one.

## What Changes

- Add a Maven multi-module SpringBoot 3.x backend (`common` / `model` / `dal` / `service` / `web`) with strict one-way dependency flow and a single executable `web` module
- Add a Vue 3 + Vite + TypeScript frontend in the same repository (mono-repo, isolated under `frontend/`)
- Introduce a unified API contract: `R<T>` response envelope, `ResultCode` enum, global exception handling, and `PageResult` pagination envelope
- Add a MyBatis-Plus data access layer with audit field auto-fill, logical delete, and pagination support
- Add Sa-Token based authentication (login/logout, token interceptor, permission annotations)
- Add Knife4j aggregated API documentation with security enhancements
- Add a Vue 3 frontend scaffold with an Axios request layer (token injection, `R<T>` unpacking, unified error toast), a layout template, and a route permission guard

## Capabilities

### New Capabilities

- `backend-module-structure`: Maven multi-module layout, parent POM dependency/version management, one-way module dependency flow, single executable `web` module
- `api-contract`: Unified `R<T>` response, `ResultCode`, global exception advice, `PageResult` pagination envelope
- `data-access-layer`: MyBatis-Plus config (pagination / optimistic lock / logical delete), `BaseEntity` + auto-fill handler, `BaseMapperX`
- `auth-session`: Sa-Token login/logout, token interceptor, permission annotations
- `api-documentation`: Knife4j aggregated doc UI + security enhancements
- `frontend-scaffold`: Vite + Vue 3 + TS + Element Plus + Pinia, layout template (sidebar / header / breadcrumb / tags)
- `frontend-request-layer`: Axios instance with token injection, `R<T>` unpacking, unified error toast, route permission guard + dynamic routes

### Modified Capabilities

<!-- Greenfield project — no existing specs to modify. -->

## Impact

- **New top-level directories**: `backend/` (Maven aggregate root) and `frontend/` (npm workspace)
- **New backend dependencies**: SpringBoot 3.x, MyBatis-Plus, Sa-Token, Knife4j, Redisson, MySQL Driver
- **New frontend dependencies**: Vue 3, Vite, TypeScript, Pinia, Vue Router, Element Plus, Axios
- **Runtime requirements**: JDK 17, Node.js 20+, MySQL 8.x, Redis
- **Tooling impact**: root `.gitignore` must cover both Maven (`target/`) and npm (`node_modules/`, `dist/`) artifacts
- **No breaking changes**: greenfield repository
