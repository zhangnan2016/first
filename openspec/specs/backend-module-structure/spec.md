# Backend Module Structure

## Purpose

Defines the Maven multi-module layout, dependency version management, and one-way module dependency flow for the backend aggregate.

## Requirements

### Requirement: Maven multi-module aggregation
The backend SHALL be a Maven aggregate composed of exactly five modules: `imooc-common`, `imooc-model`, `imooc-dal`, `imooc-service`, and `imooc-web`. The parent POM SHALL centralize all dependency versions via `<dependencyManagement>` so child modules declare artifacts without versions.

#### Scenario: Parent POM manages versions
- **WHEN** a child module declares a dependency without a `<version>` tag
- **THEN** the build resolves the version from the parent POM's `<dependencyManagement>`

#### Scenario: All five modules compile
- **WHEN** `mvn clean compile` runs from the `backend/` aggregate root
- **THEN** all five modules compile successfully

### Requirement: One-way module dependency flow
Modules SHALL depend only on modules declared to their left in the order `common → model → dal → service → web`. No cyclic or same-layer dependencies SHALL exist.

#### Scenario: Lowest layer has no business dependencies
- **WHEN** the `imooc-common` module dependencies are inspected
- **THEN** it declares no dependency on `imooc-model`, `imooc-dal`, `imooc-service`, or `imooc-web`

#### Scenario: Dependency graph is acyclic
- **WHEN** module dependencies are graphed
- **THEN** the graph is acyclic and flows strictly left to right

### Requirement: Single executable web module
Only `imooc-web` SHALL contain the application entry point (`@SpringBootApplication`) and declare the `spring-boot-maven-plugin`. All other modules SHALL produce plain JAR artifacts.

#### Scenario: Only web module is executable
- **WHEN** the modules are inspected
- **THEN** exactly one module (`imooc-web`) declares `spring-boot-maven-plugin` and a main class

#### Scenario: Backend boots
- **WHEN** `spring-boot:run` executes on `imooc-web`
- **THEN** the Spring application context starts without errors
