# Prompt – Constitución del proyecto

> Entrada para `/speckit-constitution`.
> Uso en Claude Code: `/speckit-constitution Crea o actualiza la constitución usando como entrada @docs/prompts/constitution.md`
> Para enmiendas: edita este archivo, haz commit y vuelve a ejecutar la skill indicando el tipo de cambio (MAJOR / MINOR / PATCH).

## Proyecto
Sistema de Pedidos y Aprobaciones.

## Principios
1. **La especificación es la fuente de verdad**: nada se implementa sin requisito (FR-xxx) y criterio de aceptación verificable; si código y spec difieren se corrige primero la spec vía Change Request; toda tarea referencia una historia (US-x) o requisito.
2. **Dominio primero e independiente del framework**: el paquete `domain` es Java puro (sin Spring, JPA ni Jackson); las reglas de negocio viven solo en el dominio; arquitectura hexagonal (puertos y adaptadores) `domain ← application ← infrastructure`, con puertos de entrada y de salida en la capa de aplicación.
3. **Test-First NO NEGOCIABLE**: cada regla de negocio (RN-xx) tiene una prueba unitaria escrita antes de su implementación (Rojo → Verde → Refactor); cada criterio de aceptación tiene una prueba de integración con Testcontainers; cobertura del paquete `domain` ≥ 90 %; ningún merge con pruebas fallidas.
4. **Trazabilidad de extremo a extremo**: Necesidad → US → FR/RN → Tarea (Txxx) → Commit → Prueba; commits en Conventional Commits con el ID de tarea, p. ej. `feat(orders): T021 aprobar pedido nivel 1`; los nombres de prueba citan la regla, p. ej. `RN04_above5000_requiresSupervisorAndManager()`.
5. **Integridad y auditoría de las aprobaciones**: toda transición de estado queda en un historial inmutable (quién, cuándo, acción, nivel, comentario); segregación de funciones: quien registra un pedido nunca puede aprobarlo; montos en `BigDecimal` con 2 decimales y redondeo `HALF_UP`, prohibido `double`/`float`.
6. **Simplicidad (YAGNI)**: solo se implementa lo especificado en la iteración vigente; un único servicio desplegable; sin mensajería, caché ni microservicios salvo Change Request aprobado.
7. **Uso controlado de asistentes de IA**: Claude Code y Copilot generan código solo a partir de artefactos aprobados (spec, plan, tasks) y dentro de la tarea activa; todo código generado se revisa por una persona y debe pasar las pruebas; la IA no inventa requisitos: marca los vacíos como `[NEEDS CLARIFICATION]`; no se envían secretos ni datos reales en los prompts.

## Restricciones técnicas
- Java 25, Spring Boot 3.5.x, Maven.
- PostgreSQL 17 con migraciones Flyway (prohibido `ddl-auto=update`).
- API REST/JSON con contrato OpenAPI 3.1 y errores RFC 9457.
- Pruebas con JUnit 5, AssertJ, Spring Boot Test y Testcontainers.
- Entorno local con Docker Compose.

## Idioma
Código, base de datos, configuración y API en **inglés**; artefactos SDD (constitución, spec, plan, tasks) en **español**, con un glosario spec → código en `data-model.md`.

## Flujo de desarrollo
- specify → clarify → checklist → plan → tasks → analyze → implement.
- No se planifica con `[NEEDS CLARIFICATION]` abiertos.
- No se implementa con hallazgos CRITICAL de analyze.
- Una rama por feature (`001-...`, `002-...`); merge a `master` por Pull Request con revisión.
- Los cambios a funcionalidades existentes entran como Change Request (CR-xxx) con análisis de impacto y pruebas de regresión.

## Gobernanza
- La constitución prevalece sobre cualquier otra práctica.
- Las enmiendas requieren PR, justificación y versión semántica (MAJOR: se elimina o redefine un principio; MINOR: se añade un principio; PATCH: redacción).
- Todo `plan.md` incluye un Constitution Check.
- Versión inicial 1.0.0, fecha de ratificación: la fecha de hoy.
