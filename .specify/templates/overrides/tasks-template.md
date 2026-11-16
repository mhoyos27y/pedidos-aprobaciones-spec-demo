---

description: "Task list template for feature implementation"
---

# Tasks: [FEATURE NAME]

**Input**: Design documents from `/specs/[###-feature-name]/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: OBLIGATORIAS (Constitución, Principio II — TDD no negociable). Por cada historia, las
tareas de prueba DEBEN tener IDs menores que las de implementación y DEBEN fallar antes de
implementar (Red → Green → Refactor). Nombres `given_when_then` o `@DisplayName` en español.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- Contrato: `api/openapi/[recurso].yaml`
- Producción: `src/main/java/[base/package]/[modulo]/{domain,application,infrastructure}/`
- Pruebas: `src/test/java/[base/package]/[modulo]/{domain,application,infrastructure}/`, `.../contract/`, `.../architecture/`
- Migraciones: `src/main/resources/db/migration/V[N]__[descripcion].sql`
- Versiones: solo en `gradle/libs.versions.toml`
- Ajustar según la estructura de plan.md

<!--
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for illustration purposes only.

  The /speckit-tasks command MUST replace these with actual tasks based on:
  - User stories from spec.md (with their priorities P1, P2, P3...)
  - Feature requirements from plan.md
  - Entities from data-model.md
  - Endpoints from contracts/ and api/openapi/

  Tasks MUST be organized by user story so each story can be:
  - Implemented independently
  - Tested independently
  - Delivered as an MVP increment

  Orden obligatorio dentro de cada historia (Constitución I, II, III):
  contrato OpenAPI → pruebas (fallando) → dominio → aplicación → infraestructura.

  DO NOT keep these sample tasks in the generated tasks.md file.
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Crear estructura por módulo y capa hexagonal según plan.md
- [ ] T002 Configurar Gradle Kotlin DSL + Wrapper + `gradle/libs.versions.toml` (Java 25, Spring Boot 4.x)
- [ ] T003 [P] Configurar Spotless, Checkstyle/PMD y SpotBugs/Error Prone enlazados a `check`
- [ ] T004 [P] Configurar JaCoCo con umbrales (90 % domain/application, 80 % global) enlazado a `check`
- [ ] T005 [P] Configurar openapi-generator (plugin Gradle) con salida en `build/generated/openapi`
- [ ] T006 [P] Configurar lint de Spectral y detección de breaking changes del contrato en CI
- [ ] T007 [P] Configurar escaneo de dependencias/imagen (OWASP Dependency-Check o Trivy)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

Examples of foundational tasks (adjust based on your project):

- [ ] T008 [P] Prueba ArchUnit de dependencias `infrastructure → application → domain` y dominio sin frameworks en `src/test/java/.../architecture/`
- [ ] T009 [P] Esquema `ProblemDetail` (RFC 9457), paginación y `Idempotency-Key` comunes en `api/openapi/`
- [ ] T010 Configurar Flyway, PostgreSQL 16 y `ddl-auto: validate`; base de Testcontainers para pruebas de integración
- [ ] T011 [P] Configurar OAuth2 Resource Server (JWT) y roles SOLICITANTE / APROBADOR / ADMINISTRADOR
- [ ] T012 [P] Manejador global de errores → Problem Details con códigos de negocio
- [ ] T013 [P] Logs JSON con `correlationId`, Actuator (health/readiness/liveness), Micrometer, OpenTelemetry
- [ ] T014 [P] Perfiles `local`, `test`, `prod` (12-Factor); secretos solo por variables de entorno
- [ ] T015 [P] Dockerfile multi-stage (no root, HEALTHCHECK) y docker-compose (app + PostgreSQL 16)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Contrato for User Story 1 (API First) ⚠️

- [ ] T016 [US1] Definir/actualizar endpoints en `api/openapi/[recurso].yaml` (versionado `/api/v1`, errores, paginación, idempotencia) y validar con Spectral

### Tests for User Story 1 (OBLIGATORIAS — escribir PRIMERO y verificar que FALLAN) ⚠️

- [ ] T017 [P] [US1] Unitarias de dominio (transiciones válidas/inválidas, invariantes) en `src/test/java/.../[modulo]/domain/[Agregado]Test.java`
- [ ] T018 [P] [US1] Caso de uso con puertos simulados/fakes (incluye autorización por rol) en `src/test/java/.../[modulo]/application/[CasoDeUso]Test.java`
- [ ] T019 [P] [US1] Integración de persistencia con Testcontainers (incluye conflicto por `@Version` si aplica) en `src/test/java/.../[modulo]/infrastructure/`
- [ ] T020 [P] [US1] Prueba de contrato del endpoint contra el OpenAPI (incluye 401/403 y Problem Details) en `src/test/java/.../contract/`

### Implementation for User Story 1

- [ ] T021 [P] [US1] Value objects y agregado en `src/main/java/.../[modulo]/domain/model/`
- [ ] T022 [US1] Puertos `port/in` y `port/out` en `src/main/java/.../[modulo]/application/port/`
- [ ] T023 [US1] Caso de uso (una clase) en `src/main/java/.../[modulo]/application/usecase/` (depends on T021, T022)
- [ ] T024 [US1] Migración Flyway `V[N]__[descripcion].sql`
- [ ] T025 [US1] Adaptador de persistencia (entidad JPA + mapper explícito) en `.../infrastructure/adapter/out/persistence/`
- [ ] T026 [US1] Adaptador REST implementando la interfaz generada en `.../infrastructure/adapter/in/rest/`
- [ ] T027 [US1] Refactor con pruebas en verde

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Contrato for User Story 2 (API First) ⚠️

- [ ] T028 [US2] Actualizar `api/openapi/[recurso].yaml` y validar con Spectral

### Tests for User Story 2 (OBLIGATORIAS — escribir PRIMERO y verificar que FALLAN) ⚠️

- [ ] T029 [P] [US2] Unitarias de dominio en `src/test/java/.../[modulo]/domain/`
- [ ] T030 [P] [US2] Caso de uso en `src/test/java/.../[modulo]/application/`
- [ ] T031 [P] [US2] Integración / contrato en `src/test/java/.../`

### Implementation for User Story 2

- [ ] T032 [P] [US2] Dominio en `src/main/java/.../[modulo]/domain/`
- [ ] T033 [US2] Caso de uso y puertos en `src/main/java/.../[modulo]/application/`
- [ ] T034 [US2] Adaptadores en `src/main/java/.../[modulo]/infrastructure/`

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

[Add more user story phases as needed, following the same pattern]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] TXXX [P] Actualizar `README.md` y ADRs en `docs/adr/`
- [ ] TXXX Refactor global con pruebas en verde
- [ ] TXXX Verificar `./gradlew build` en verde (pruebas, cobertura, ArchUnit, lint, análisis estático)
- [ ] TXXX Verificar build de imagen Docker y `docker-compose up` en estado saludable
- [ ] TXXX Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but should be independently testable

### Within Each User Story

- Contrato OpenAPI antes que todo (Principio I)
- Pruebas escritas y FALLANDO antes de la implementación (Principio II)
- Dominio → aplicación → infraestructura (Principio III)
- Refactor solo con pruebas en verde
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tests for a user story marked [P] can run in parallel
- Domain classes within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Lanzar juntas las pruebas de la US1 (después del contrato):
Task: "Unitarias de dominio en src/test/java/.../[modulo]/domain/[Agregado]Test.java"
Task: "Caso de uso en src/test/java/.../[modulo]/application/[CasoDeUso]Test.java"
Task: "Contrato en src/test/java/.../contract/[Recurso]ContractTest.java"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Each story adds value without breaking previous stories

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Verify tests fail before implementing (Red → Green → Refactor)
- Nunca editar código generado desde OpenAPI
- Commits pequeños con Conventional Commits (`test:`, `feat:`, `refactor:`...)
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
