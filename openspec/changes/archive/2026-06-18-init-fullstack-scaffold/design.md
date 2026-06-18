## Context

Greenfield repository with no application code. The only existing assets are the OpenSpec/Superpowers/Qoder workflow directories. The goal is to establish a reusable, framework-grade fullstack scaffold so future services start from a battle-tested baseline. The codebase lives in a single repository (mono-repo) hosting both a Maven backend and an npm frontend, evolving independently but sharing a strict API contract.

Stakeholders: any team bootstrapping a new SpringBoot + Vue service. Constraints: Java ecosystem maturity, long-term-support JVM, and mainstream Chinese-team tooling familiarity.

## Goals / Non-Goals

**Goals:**

- Provide a runnable backend on first build (`mvnw -pl imooc-web spring-boot:run`)
- Provide a runnable frontend on first build (`npm run dev`)
- Encode cross-cutting standards (response envelope, exception handling, audit fields, auth, docs) as the default, not an option
- Keep module dependencies strictly one-way so the structure stays maintainable
- Lock all dependency versions in the parent POM and `package.json` for reproducible builds

**Non-Goals:**

- Business features (a single `sys_user` CRUD example is included only to demonstrate the framework conventions)
- Microservices / service registry / gateway (single deployable service)
- Docker, CI/CD pipelines (may follow as separate changes)
- Internationalization framework, design-system tokens, charting libraries
- Production secret management (dev defaults only; secrets via env vars)

## Decisions

### D1 — Mono-Repo with isolated subdirectories

`backend/` (Maven aggregate root) and `frontend/` (npm) live in one repository. Each OpenSpec change can cover both sides of a contract atomically, and version/issue tracking stays unified.

*Alternative considered:* two repositories with a shared API-contract package. Rejected — contract drift is harder to catch, and a single change touching both sides becomes two coordinated PRs.

### D2 — Module layout (5 modules, one-way dependency)

```
imooc-common ◄── imooc-model ◄── imooc-dal ◄── imooc-service ◄── imooc-web
                                                                (only executable)
```

- `common`: `R<T>`, `ResultCode`, `BizException`, `PageResult<T>`, `BaseEntity`, utils, enums
- `model`: entities, DTOs, VOs, query objects
- `dal`: mappers + MyBatis-Plus config + `MetaObjectHandler` + `BaseMapperX`
- `service`: business interfaces + `@Transactional` impls
- `web`: controllers + startup class + config (CORS, Sa-Token, Knife4j, Jackson)

Rule: a module may only depend on modules to its left. No cycles, no same-layer references. Only `web` declares `spring-boot-maven-plugin` (fat-jar packaging).

*Alternative considered:* full starter extraction (`starter-core`, `starter-mybatis`) to produce reusable jars. Deferred — premature for a first scaffold; starter extraction can be introduced in a later change once the conventions prove stable (YAGNI).

### D3 — SpringBoot 3.x + JDK 17

LTS JVM, Jakarta namespace (`jakarta.*`), current security baseline. All third-party libraries must therefore be Jakarta-compatible versions (see Risks).

### D4 — MyBatis-Plus over JPA

CRUD auto-generation, built-in pagination / logical-delete / optimistic-lock plugins, and idiomatic Chinese-team SQL control. `BaseMapperX` wraps common query helpers so business mappers inherit them.

### D5 — Sa-Token over Spring Security

Lightweight, API-first auth with annotation-driven permissions. Sufficient for the scaffold; Spring Security can replace it later if compliance requirements demand a heavier stack.

### D6 — Unified API contract

Every endpoint returns `R<T> = { code:int, msg:string, data:T }`. `code == 0` means success; non-zero is an error code from `ResultCode`. Pagination uses `PageResult<T> = { records, total, current, size }`. Token is sent via `Authorization: Bearer <token>`. Global exception advice translates validation, business, and unknown errors into `R<Void>` so the frontend always receives a parsable envelope.

### D7 — Frontend stack and conventions

Vue 3 + Vite + TypeScript + Pinia + Vue Router + Element Plus. `utils/request.ts` wraps Axios: request interceptor injects the bearer token, response interceptor unwraps `R<T>` (rejects on `code != 0` with a toast) and handles 401 by redirecting to login. Layout provides sidebar / header / breadcrumb / tags-view. Route guard loads dynamic routes (server-returned menu) after login.

## Risks / Trade-offs

- **Jakarta compatibility**: SpringBoot 3 moved to `jakarta.*`. MyBatis-Plus, Sa-Token, and Knife4j must be on their SpringBoot-3-compatible release lines, or startup fails with `ClassNotFoundException`. → Mitigation: pin explicit Jakarta-ready versions in the parent POM; verify with a smoke test that boots the context.
- **JDK 17 strong encapsulation**: some libraries need reflective access. → Mitigation: add `--add-opens` JVM args in `application-dev.yml` / runner config only if a concrete library requires it; do not blanket-disable.
- **Mono-repo tooling**: `.gitignore` and CI must handle both `target/` and `node_modules/`. → Mitigation: root `.gitignore` covers both; each toolchain is confined to its subdirectory.
- **Dynamic-route contract drift**: frontend dynamic routes depend on a server menu shape that is not yet finalized. → Mitigation: ship a minimal fixed contract in `design.md` (this file) and keep the `sys_user` example as the reference shape; formalize in a future auth change.
- **Premature abstraction**: module boundaries chosen now may not fit future needs. → Mitigation: 5-module split is reversible; defer starter extraction until conventions are proven.

## Migration Plan

Greenfield — no existing system to migrate. Rollout is a single initial commit. Rollback = delete `backend/` and `frontend/` directories; no data or API consumers exist yet.

## Open Questions

- Final database choice defaults to MySQL 8.x; PostgreSQL remains a drop-in swap (driver + dialect only) if the team prefers it.
- Whether to include a Docker Compose for local MySQL/Redis now or as a separate change — currently left out (Non-Goal).
