# Constitución del Sistema de Pedidos y Aprobaciones

**Contexto del sistema.** Plataforma para crear, enviar, revisar, aprobar, rechazar y cancelar
pedidos, con flujos de aprobación configurables (uno o varios niveles, según monto, área o rol).
Toda transición de estado DEBE quedar trazada y ser auditable.

Esta constitución es la fuente de verdad no negociable que guía `/speckit-specify`,
`/speckit-clarify`, `/speckit-plan`, `/speckit-tasks` y `/speckit-implement`. Las palabras
**DEBE**, **NO DEBE** y **DEBERÍA** se interpretan según RFC 2119: DEBE/NO DEBE son obligatorias;
DEBERÍA admite excepción solo si se justifica en el "Complexity Tracking" del plan.

## Core Principles

### I. API First (NO NEGOCIABLE)

**Reglas**

- El contrato OpenAPI 3.1 DEBE escribirse, revisarse y versionarse ANTES de cualquier
  implementación. Ubicación única: `api/openapi/*.yaml`.
- Los DTOs y las interfaces de los controladores DEBEN generarse desde el contrato con
  `openapi-generator` (plugin de Gradle). El código generado NO DEBE editarse a mano ni
  versionarse en `src/`; DEBE regenerarse en cada build.
- La API DEBE versionarse en la URL (`/api/v1/...`). Un cambio incompatible DEBE publicarse
  como una nueva versión mayor (`/api/v2/...`); NO DEBE romperse una versión publicada.
- Los errores DEBEN seguir RFC 9457 (Problem Details, `application/problem+json`) e incluir un
  código de error de negocio estable y documentado en el contrato (p. ej.
  `PEDIDO_TRANSICION_INVALIDA`, `APROBACION_AUTOAPROBACION_PROHIBIDA`).
- Paginación, filtrado y ordenamiento DEBEN definirse en el contrato para todo endpoint de
  colección. Los `POST` de creación DEBEN aceptar el header `Idempotency-Key` y su semántica
  (respuesta repetida, conflicto ante payload distinto) DEBE documentarse en el contrato.
- El pipeline DEBE validar el contrato con Spectral y DEBE fallar ante cambios incompatibles
  respecto a la última versión publicada (p. ej. con `oasdiff`).

**Justificación.** El contrato es el acuerdo con los consumidores; definirlo primero evita que la
implementación dicte la API y permite trabajar en paralelo, generar clientes y probar contratos.

**Criterios verificables**

- Existe un archivo en `api/openapi/` modificado en el mismo PR (o en uno previo) que cualquier
  cambio de endpoint.
- `./gradlew build` ejecuta `openApiGenerate`; no hay clases generadas bajo `src/main/java`.
- Todas las rutas empiezan por `/api/v{N}/`.
- Todas las respuestas 4xx/5xx del contrato referencian el esquema `ProblemDetail`.
- Spectral y la detección de cambios incompatibles corren en CI y bloquean el merge.

### II. Test First / TDD (NO NEGOCIABLE)

**Reglas**

- El ciclo Red → Green → Refactor es obligatorio: NO DEBE escribirse código de producción sin
  una prueba que falle primero.
- En `/speckit-tasks`, las tareas de prueba DEBEN preceder a las de implementación de la misma
  historia y son OBLIGATORIAS (no opcionales).
- Pirámide de pruebas obligatoria:
  1. **Unitarias del dominio** (JUnit Jupiter + AssertJ), sin Spring y sin infraestructura.
  2. **Casos de uso** con puertos simulados (Mockito) o fakes en memoria.
  3. **Integración de adaptadores** con Testcontainers sobre PostgreSQL 16 real. H2 u otras
     bases embebidas NO DEBEN usarse.
  4. **Contrato**: verifican que la API implementada cumple el OpenAPI.
  5. **Arquitectura** con ArchUnit.
- Cobertura mínima (JaCoCo): paquetes `domain` y `application` ≥ 90 %; global ≥ 80 %. El build
  DEBE fallar si no se alcanza.
- Las pruebas DEBEN nombrarse con el formato `given_when_then` o con un `@DisplayName`
  descriptivo en español.

**Justificación.** Las reglas de aprobación son críticas para el negocio; escribir la prueba
primero fija el comportamiento esperado y evita regresiones en la máquina de estados.

**Criterios verificables**

- El historial del PR muestra la prueba antes (o junto con) la implementación, y la prueba
  falló al menos una vez (evidencia en la descripción del PR o en commits `test:` previos).
- `jacocoTestCoverageVerification` está enlazado a `check` con los umbrales indicados.
- No existe dependencia a H2 en `gradle/libs.versions.toml`.
- `tasks.md` lista, por cada historia, tareas de prueba con IDs menores que las de implementación.

### III. Arquitectura Hexagonal (Puertos y Adaptadores)

**Reglas**

- La dependencia DEBE ir en una sola dirección: `infrastructure → application → domain`.
  El dominio NO DEBE depender de Spring, JPA, Jackson ni de ningún otro framework.
- `domain`: entidades, value objects, agregados (`Pedido`, `FlujoAprobacion`, `Aprobacion`),
  eventos de dominio, reglas e invariantes y la máquina de estados del pedido.
- `application`: casos de uso (una clase por caso de uso), puertos de entrada (`port.in`) y de
  salida (`port.out`), y orquestación transaccional.
- `infrastructure`: adaptadores de entrada (REST generado desde OpenAPI) y de salida
  (persistencia JPA, mensajería, notificaciones). El mapeo entre entidades JPA y el modelo de
  dominio DEBE ser explícito; las entidades JPA NO DEBEN salir del adaptador.
- El código DEBE organizarse por bounded context o módulo funcional (`pedidos`, `aprobaciones`,
  `catalogo`, `usuarios`) y, dentro de cada uno, por capa hexagonal. Se PUEDE usar Spring
  Modulith para verificar los límites entre módulos.
- Las reglas anteriores DEBEN verificarse automáticamente con ArchUnit en el build.

**Justificación.** Aislar el dominio permite probarlo sin infraestructura, sustituir adaptadores
sin tocar reglas de negocio y mantener límites claros entre módulos.

**Criterios verificables**

- Pruebas ArchUnit que fallan si `..domain..` importa `org.springframework..`, `jakarta.persistence..`
  o `com.fasterxml.jackson..`/`tools.jackson..`, o si `..application..` depende de `..infrastructure..`.
- Prueba ArchUnit que impide que clases anotadas con `@Entity` sean referenciadas fuera de
  `..infrastructure..persistence..`.
- Cada caso de uso es una clase única que implementa un puerto de entrada.

### IV. Stack Tecnológico Obligatorio

**Reglas**

- **Java 25 (LTS)**. Se DEBERÍAN usar records, sealed interfaces para estados y resultados,
  pattern matching y virtual threads cuando aporten valor.
- **Spring Boot 4.x (Spring Framework 7)**: Web MVC, Validation, Data JPA, Security, Actuator.
- **PostgreSQL 16** con migraciones versionadas en **Flyway**. `spring.jpa.hibernate.ddl-auto`
  NO DEBE tener un valor distinto de `validate`.
- **Gradle con Kotlin DSL**, Gradle Wrapper obligatorio y Version Catalog
  (`gradle/libs.versions.toml`) como única fuente de versiones. Los build scripts NO DEBEN
  contener versiones hardcodeadas.
- **Docker**: Dockerfile multi-stage con imagen base mínima, usuario no root y `HEALTHCHECK`,
  más `docker-compose` para el entorno local (app + PostgreSQL 16).
- Toda dependencia nueva DEBE justificarse en `plan.md` (propósito, alternativa descartada).

**Justificación.** Un stack único y fijado reduce la variabilidad, facilita la revisión y
garantiza builds reproducibles.

**Criterios verificables**

- `toolchain` de Gradle fijado en Java 25; `gradlew` y `gradle/wrapper/` versionados.
- `grep` de versiones literales en `*.gradle.kts` no devuelve resultados.
- Perfiles de configuración declaran `ddl-auto: validate`; migraciones en `db/migration/V*__*.sql`.
- El Dockerfile tiene ≥ 2 etapas, una instrucción `USER` no root y `HEALTHCHECK`.

### V. Integridad del Dominio de Aprobaciones

**Reglas**

- El ciclo de vida del pedido DEBE modelarse como una máquina de estados explícita:
  `BORRADOR → ENVIADO → EN_APROBACION → APROBADO | RECHAZADO | CANCELADO`. El dominio DEBE
  rechazar toda transición no definida.
- Un usuario NO DEBE aprobar un pedido creado por él mismo (segregación de funciones).
- Toda transición DEBE registrar quién, cuándo, estado origen, estado destino y motivo en una
  auditoría inmutable (solo inserción; nunca `UPDATE` ni `DELETE`).
- DEBE usarse control de concurrencia optimista (versionado) para evitar aprobaciones dobles.
- Los montos DEBEN representarse con `BigDecimal` y moneda explícita (value object). Las fechas
  DEBEN almacenarse en UTC con tipos `java.time` (`Instant`/`OffsetDateTime`).

**Justificación.** La confianza en el sistema depende de que ningún pedido cambie de estado de
forma indebida, duplicada o sin rastro.

**Criterios verificables**

- Pruebas unitarias cubren cada transición válida y al menos una inválida por estado.
- Prueba de dominio y de caso de uso que verifica el rechazo de la autoaprobación.
- La tabla de auditoría no tiene permisos `UPDATE`/`DELETE` para el usuario de la aplicación
  (o un trigger los impide), verificado en prueba de integración.
- Entidades JPA de agregados tienen `@Version`; prueba de integración demuestra el conflicto
  ante dos aprobaciones concurrentes.
- No se usan `double`/`float` para montos ni `LocalDateTime`/`java.util.Date` para instantes.

### VI. Seguridad

**Reglas**

- La autenticación DEBE realizarse con OAuth2/OIDC (JWT, Spring Resource Server). La
  autorización por roles (`SOLICITANTE`, `APROBADOR`, `ADMINISTRADOR`) DEBE aplicarse en el
  borde y también en los casos de uso.
- La entrada DEBE validarse en el borde (Bean Validation) y las invariantes en el dominio.
- Los secretos DEBEN residir solo en variables de entorno o en un gestor de secretos; NO DEBEN
  estar en el repositorio.
- El pipeline DEBE escanear dependencias y la imagen Docker (OWASP Dependency-Check o Trivy) y
  fallar ante vulnerabilidades de severidad alta o crítica sin excepción documentada.

**Justificación.** Las aprobaciones tienen impacto financiero; la defensa en profundidad evita
que un fallo en una sola capa permita acciones no autorizadas.

**Criterios verificables**

- Pruebas de integración verifican 401 sin token y 403 con rol insuficiente por endpoint.
- Pruebas de caso de uso verifican el rechazo por rol sin pasar por la capa web.
- Escaneo de secretos (p. ej. gitleaks) y de dependencias/imagen en CI sin hallazgos bloqueantes.

### VII. Observabilidad y Operación

**Reglas**

- Los logs DEBEN ser estructurados en JSON e incluir `correlationId`. NO DEBEN registrarse datos
  sensibles (tokens, credenciales, datos personales).
- Actuator DEBE exponer `health`, `readiness` y `liveness`; las métricas DEBEN publicarse con
  Micrometer y las trazas con OpenTelemetry.
- La configuración DEBE organizarse por perfiles (`local`, `test`, `prod`) siguiendo 12-Factor.

**Justificación.** Un flujo de aprobaciones auditable requiere poder reconstruir y diagnosticar
cada solicitud en producción.

**Criterios verificables**

- Prueba que verifica que un request propaga/genera `correlationId` en logs y en la respuesta.
- `/actuator/health/readiness` y `/actuator/health/liveness` responden `UP` en docker-compose.
- No existen valores específicos de entorno hardcodeados fuera de los perfiles.

### VIII. Calidad y Simplicidad

**Reglas**

- YAGNI: NO DEBE implementarse nada que no esté en una especificación aprobada.
- Los métodos DEBERÍAN ser cortos y los nombres DEBEN tomarse del lenguaje ubicuo: español para
  el dominio (`Pedido`, `aprobar`, `FlujoAprobacion`), inglés para lo técnico (`Repository`,
  `Controller`, `Mapper`), con el mismo criterio en todo el proyecto.
- Formato y análisis estático obligatorios: Spotless, Checkstyle o PMD, y SpotBugs o Error Prone,
  todos enlazados a `./gradlew build`.
- Pull requests pequeños, Conventional Commits y revisión obligatoria por al menos una persona
  distinta del autor.

**Justificación.** La simplicidad y la consistencia reducen el costo de mantenimiento y hacen que
la revisión sea efectiva.

**Criterios verificables**

- `spotlessCheck`, Checkstyle/PMD y SpotBugs/Error Prone forman parte de `check` y fallan el build.
- Cada funcionalidad implementada se traza a un requisito de `spec.md`.
- Los mensajes de commit cumplen Conventional Commits (validado en CI o hook).

## Puertas de Calidad y Constitution Check

`/speckit-plan` DEBE evaluar estas puertas antes de la Fase 0 y de nuevo tras el diseño de la
Fase 1. Cualquier "NO" DEBE justificarse en "Complexity Tracking" o el plan no puede avanzar.

| # | Puerta | Principio |
|---|--------|-----------|
| G1 | ¿El contrato OpenAPI 3.1 en `api/openapi/` se define/actualiza antes de implementar, con `/api/v{N}`, Problem Details, paginación e `Idempotency-Key`? | I |
| G2 | ¿Las tareas de prueba preceden a las de implementación y cubren las cinco capas de la pirámide? | II |
| G3 | ¿Se mantienen los umbrales JaCoCo (90 % dominio/aplicación, 80 % global)? | II |
| G4 | ¿El diseño respeta `infrastructure → application → domain` y lo verifica ArchUnit? | III |
| G5 | ¿Solo se usa el stack obligatorio y toda dependencia nueva está justificada? | IV |
| G6 | ¿Las transiciones, la segregación de funciones, la auditoría inmutable y el versionado optimista están contemplados? | V |
| G7 | ¿Autenticación JWT, autorización por rol en borde y caso de uso, y secretos fuera del repo? | VI |
| G8 | ¿Logs JSON con `correlationId`, probes de Actuator, métricas y trazas? | VII |
| G9 | ¿Todo lo planificado proviene de la especificación aprobada (YAGNI)? | VIII |

## Flujo de Desarrollo (Definition of Done)

Una funcionalidad está terminada solo cuando se cumplen, en orden, todos los puntos:

1. Especificación aprobada (`/speckit-specify` y `/speckit-clarify`).
2. Contrato OpenAPI actualizado y validado (Spectral + detección de cambios incompatibles).
3. Plan técnico que pasa el Constitution Check (`/speckit-plan`).
4. Pruebas escritas y fallando (Red), implementación hasta verde (Green) y refactor.
5. `./gradlew build` en verde: pruebas, cobertura, ArchUnit, lint y análisis estático.
6. La imagen Docker se construye y `docker-compose` levanta el sistema en estado saludable.
7. Documentación actualizada: `README.md` y ADRs en `docs/adr/`.

## Governance

- Esta constitución prevalece sobre cualquier otra práctica, guía o preferencia.
- Toda desviación DEBE documentarse en la sección "Complexity Tracking" de `plan.md`, con su
  justificación y la alternativa más simple que se descartó. Las desviaciones de los Principios I
  y II no son admisibles.
- Toda enmienda DEBE acompañarse de un ADR en `docs/adr/`, contar con la aprobación del equipo y
  actualizar la versión con SemVer:
  - **MAYOR**: elimina o redefine principios de forma incompatible.
  - **MENOR**: agrega principios o secciones, o amplía materialmente una guía.
  - **PARCHE**: aclaraciones, redacción o correcciones no semánticas.
- Cumplimiento: toda revisión de PR DEBE verificar el cumplimiento de esta constitución;
  `/speckit-analyze` DEBE señalar como CRÍTICO cualquier conflicto con un principio.

**Version**: 1.0.0 | **Ratified**: 2026-11-16 | **Last Amended**: 2026-11-16
