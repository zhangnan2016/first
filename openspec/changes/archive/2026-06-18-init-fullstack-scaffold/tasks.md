## 1. Repository foundation

- [x] 1.1 Update root `.gitignore` to ignore Maven (`target/`) and npm (`node_modules/`, `dist/`, `*.local`) artifacts
- [x] 1.2 Create `backend/` parent `pom.xml`: SpringBoot 3.x parent, JDK 17, `<dependencyManagement>` pinning MyBatis-Plus, Sa-Token, Knife4j, Redisson, MySQL connector, Lombok
- [x] 1.3 Scaffold `frontend/` with the Vite Vue 3 + TS template; add Pinia, Vue Router, Element Plus, Axios
- [x] 1.4 Add `frontend/.env.development` and `.env.production` defining `VITE_API_BASE_URL`

## 2. Backend: imooc-common

- [x] 2.1 Add the `imooc-common` module and its pom (no business dependencies)
- [x] 2.2 Implement `R<T>`, `PageResult<T>`, `ResultCode`, and `BizException` — write failing unit tests first (RED), then pass (GREEN)
- [x] 2.3 Implement `BaseEntity` with `id`, `createTime`, `updateTime`, `createBy`

## 3. Backend: imooc-model

- [x] 3.1 Add the `imooc-model` module and its pom (depends on `imooc-common`)
- [x] 3.2 Create the `SysUser` entity (extends `BaseEntity`) plus `SysUserDTO`, `SysUserVO`, and `SysUserQuery`

## 4. Backend: imooc-dal

- [x] 4.1 Add the `imooc-dal` module and its pom (depends on `imooc-model` + MyBatis-Plus)
- [x] 4.2 Configure `MybatisPlusConfig` (pagination, optimistic-lock, logical-delete plugins)
- [x] 4.3 Implement `MetaObjectHandler` auto-fill — write failing tests for insert/update fill (RED), then pass (GREEN)
- [x] 4.4 Implement `BaseMapperX` and `SysUserMapper` (extends `BaseMapperX`)

## 5. Backend: imooc-service

- [x] 5.1 Add the `imooc-service` module and its pom (depends on `imooc-dal` + `imooc-model`)
- [x] 5.2 Implement `ISysUserService` and its impl with `@Transactional` CRUD — write failing service tests (RED), then pass (GREEN)

## 6. Backend: imooc-web

- [x] 6.1 Add the `imooc-web` module and its pom (depends on `imooc-service` + `imooc-common` + web starters; declares `spring-boot-maven-plugin`)
- [x] 6.2 Create the `@SpringBootApplication` startup class and `application.yml` / `application-dev.yml` / `application-prod.yml`
- [x] 6.3 Implement `GlobalExceptionHandler` advice — write failing tests for validation, business, and unknown exceptions (RED), then pass (GREEN)
- [x] 6.4 Implement CORS config and Jackson config (unified date format, null handling)
- [x] 6.5 Implement Sa-Token config: interceptor with login/docs allow-list + login endpoint — write failing tests (RED), then pass (GREEN)
- [x] 6.6 Implement Knife4j config (open in non-prod, disabled in prod)
- [x] 6.7 Implement `SysUserController` exposing CRUD through `R<T>` / `R<PageResult<T>>` — write failing controller tests (RED), then pass (GREEN)

## 7. Frontend: request layer and login

- [x] 7.1 Implement `utils/request.ts`: Axios instance with token injection, `R<T>` unwrap, error toast, and 401 redirect
- [x] 7.2 Implement the user Pinia store (token + permissions persistence in `localStorage`)
- [x] 7.3 Implement the login page wired to the backend login endpoint

## 8. Frontend: layout and routing

- [x] 8.1 Implement the layout template (sidebar, header, breadcrumb, tags-view)
- [x] 8.2 Implement the router with the permission guard and dynamic-routes scaffold

## 9. End-to-end verification

- [x] 9.1 Verify the backend boots via `spring-boot:run`
- [x] 9.2 Verify the frontend dev server runs via `npm run dev`
- [x] 9.3 Verify the login flow end-to-end: token issuance, `R<T>` unwrap, and protected-route guard
