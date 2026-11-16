# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command; its definition describes the execution workflow.
Override del proyecto alineado con la Constitución v1.0.0 (`.specify/memory/constitution.md`).

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  Valores fijados por la Constitución (Principio IV). NO cambiarlos aquí: cualquier
  desviación va en "Complexity Tracking". Completa solo los campos marcados.
-->

**Language/Version**: Java 25 (LTS) — records, sealed interfaces, pattern matching, virtual threads cuando aporten valor

**Primary Dependencies**: Spring Boot 4.x (Spring Framework 7): Web MVC, Validation, Data JPA, Security (OAuth2 Resource Server), Actuator; Flyway; openapi-generator (plugin Gradle); Micrometer; OpenTelemetry

**Storage**: PostgreSQL 16, migraciones Flyway (`ddl-auto: validate`)

**Testing**: JUnit Jupiter + AssertJ, Mockito, Testcontainers (PostgreSQL 16), pruebas de contrato contra OpenAPI, ArchUnit, JaCoCo (≥ 90 % domain/application, ≥ 80 % global)

**Build**: Gradle Kotlin DSL + Wrapper + Version Catalog (`gradle/libs.versions.toml`)

**Target Platform**: Contenedor Linux (Dockerfile multi-stage, usuario no root, HEALTHCHECK) + docker-compose local

**Project Type**: web-service (REST API First, `/api/v1`)

**Módulos afectados**: [pedidos | aprobaciones | catalogo | usuarios]

**Contrato OpenAPI**: [ruta en `api/openapi/*.yaml` y endpoints nuevos/modificados; ¿cambio compatible o nueva versión mayor?]

**Performance Goals**: [domain-specific or NEEDS CLARIFICATION]

**Constraints**: [domain-specific or NEEDS CLARIFICATION]

**Scale/Scope**: [domain-specific or NEEDS CLARIFICATION]

**Nuevas dependencias**: [Ninguna | lista — cada una DEBE justificarse en la tabla "Dependencias nuevas"]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

Marca cada puerta. Todo "NO" DEBE justificarse en "Complexity Tracking"; G1–G3 (Principios I y II)
no admiten desviación.

| # | Puerta | Principio | Pre-Fase 0 | Post-Fase 1 |
|---|--------|-----------|------------|-------------|
| G1 | Contrato OpenAPI 3.1 en `api/openapi/` definido antes de implementar; rutas `/api/v{N}`; errores RFC 9457 con códigos de negocio; paginación/filtrado/orden; `Idempotency-Key` en POST de creación; Spectral + detección de breaking changes | I | [ ] | [ ] |
| G2 | Tareas de prueba preceden a las de implementación y cubren dominio, casos de uso, integración (Testcontainers, sin H2), contrato y ArchUnit | II | [ ] | [ ] |
| G3 | Se mantienen umbrales JaCoCo (90 % domain/application, 80 % global) | II | [ ] | [ ] |
| G4 | Dependencias `infrastructure → application → domain`; dominio sin frameworks; una clase por caso de uso; entidades JPA no salen del adaptador; verificado con ArchUnit | III | [ ] | [ ] |
| G5 | Solo stack obligatorio; versiones solo en Version Catalog; dependencias nuevas justificadas | IV | [ ] | [ ] |
| G6 | Máquina de estados explícita; segregación de funciones (no autoaprobación); auditoría inmutable (quién, cuándo, origen, destino, motivo); bloqueo optimista; `BigDecimal` + moneda; UTC con `java.time` | V | [ ] | [ ] |
| G7 | JWT/OIDC; autorización por rol en borde y en caso de uso; Bean Validation en el borde; secretos fuera del repo; escaneo de dependencias/imagen | VI | [ ] | [ ] |
| G8 | Logs JSON con `correlationId` y sin datos sensibles; health/readiness/liveness; Micrometer; OpenTelemetry; perfiles local/test/prod | VII | [ ] | [ ] |
| G9 | Todo lo planificado proviene de la spec aprobada (YAGNI); lenguaje ubicuo (español dominio / inglés técnico); análisis estático en el build | VIII | [ ] | [ ] |

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command) — borrador; la fuente de verdad es api/openapi/
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Conserva solo los módulos afectados por esta feature y expande con
  las clases reales. La estructura por módulo y capa hexagonal es obligatoria (Principio III).
-->

```text
api/openapi/
└── [recurso].yaml                         # Contrato OpenAPI 3.1 (fuente de verdad)

gradle/libs.versions.toml                  # Única fuente de versiones
build.gradle.kts / settings.gradle.kts

src/main/java/[base/package]/
└── [modulo]/                              # pedidos | aprobaciones | catalogo | usuarios
    ├── domain/
    │   ├── model/                         # Agregados, entidades, value objects
    │   ├── event/                         # Eventos de dominio
    │   └── exception/                     # Violaciones de invariantes
    ├── application/
    │   ├── port/in/                       # Puertos de entrada (casos de uso)
    │   ├── port/out/                      # Puertos de salida
    │   └── usecase/                       # Una clase por caso de uso
    └── infrastructure/
        ├── adapter/in/rest/               # Implementa interfaces generadas desde OpenAPI + mappers
        └── adapter/out/persistence/       # Entidades JPA, repositorios Spring Data, mappers

src/main/resources/
├── db/migration/V[N]__[descripcion].sql   # Flyway
└── application-{local,test,prod}.yml

src/test/java/[base/package]/
├── [modulo]/domain/                       # Unitarias (JUnit + AssertJ, sin Spring)
├── [modulo]/application/                  # Casos de uso (Mockito / fakes)
├── [modulo]/infrastructure/               # Integración (Testcontainers PostgreSQL 16)
├── contract/                              # Cumplimiento del OpenAPI
└── architecture/                          # ArchUnit (y Spring Modulith si aplica)

build/generated/openapi/                   # Código generado — NO editar ni versionar
Dockerfile
docker-compose.yml
docs/adr/
```

**Structure Decision**: [Módulos y paquetes reales afectados por esta feature]

## Dependencias nuevas

> Obligatorio si "Nuevas dependencias" no es "Ninguna" (Principio IV).

| Dependencia (alias en libs.versions.toml) | Propósito | Alternativa descartada y por qué |
|-------------------------------------------|-----------|----------------------------------|
| [alias] | [necesidad concreta] | [alternativa más simple y motivo] |

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**
> Los Principios I (API First) y II (TDD) no admiten desviaciones.

| Violación (puerta / principio) | Por qué es necesaria | Alternativa más simple descartada y por qué |
|--------------------------------|----------------------|---------------------------------------------|
| [p. ej. G5 / IV: librería X] | [necesidad concreta] | [por qué la opción más simple no basta] |
