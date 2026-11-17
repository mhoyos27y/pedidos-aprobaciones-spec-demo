# Prompt – Tareas de la feature 001: Registro y aprobación de pedidos

> Entrada para `/speckit-tasks` (Sesión 06).
> Uso en Claude Code: `/speckit-tasks Genera el backlog usando como entrada @docs/prompts/features/001-registro-aprobacion-pedidos/05-tasks.md`
> Las tareas se derivan de `plan.md`, `spec.md`, `data-model.md`, `contracts/` y `research.md`; este archivo solo fija **cómo** organizarlas.

## Organización
- Fases: **Setup** → **Fundacional** (bloquea todas las historias) → una fase por historia en orden de prioridad (**US1 = MVP**, US2, US3, US4) → **Pulido**.
- Cada historia debe poder implementarse, probarse y demostrarse de forma independiente; terminar cada fase con un **Checkpoint** verificable.

## Formato de cada tarea
`- [ ] T0xx [P] [USx] Descripción con la ruta exacta del archivo`
- `[P]` solo si es paralelizable (archivos distintos y sin dependencias pendientes).
- `[USx]` en toda tarea de una historia; las de Setup, Fundacional y Pulido no llevan historia.
- Rutas reales del plan (`src/main/java/pe/galaxy/orders/...`, `src/test/java/...`, `src/main/resources/db/migration/...`).

## Reglas obligatorias (Constitución)
- **Test-First (Principio III)**: dentro de cada historia la tarea de prueba va **antes** que su implementación y debe fallar primero.
- Pruebas unitarias del dominio por regla de negocio (incluidos valores frontera) y pruebas de integración con Testcontainers por escenario de aceptación.
- Tarea explícita para concurrencia (bloqueo optimista) y para la prueba de arquitectura (dominio sin frameworks, dinero sin `double`/`float`).
- Ninguna tarea fuera del alcance de la spec (Principio VI).

## Trazabilidad
- Al final de `tasks.md`, una **matriz de trazabilidad**: cada FR-xxx y RN-xx → tareas → pruebas.
- Sección de **dependencias y orden de ejecución**, indicando qué historias pueden avanzar en paralelo.
