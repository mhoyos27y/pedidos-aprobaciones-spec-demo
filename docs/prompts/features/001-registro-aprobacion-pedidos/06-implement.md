# Prompt – Implementación de la feature 001: Registro y aprobación de pedidos

> Entrada para `/speckit-implement` (Sesión 07).
> Uso en Claude Code (una fase por ejecución):
> `/speckit-implement Ejecuta la fase indicada siguiendo @docs/prompts/features/001-registro-aprobacion-pedidos/06-implement.md — Fase: <nombre de la fase>`
> La fuente de verdad son `spec.md`, `plan.md`, `data-model.md`, `contracts/` y `tasks.md`. Este archivo solo fija **cómo** trabajar.

## Alcance de cada ejecución
- Ejecuta **solo la fase indicada** en el mensaje (p. ej. "Fase 1 – Setup y Fase 2 – Fundacional", "Fase 3 – US1").
- Al llegar al **Checkpoint** de la fase: detente, muestra el resultado de las pruebas y espera mi revisión.
- No implementes nada que no esté en `tasks.md` (Principio VI). Si falta una tarea, propónla y espera confirmación.

## Ciclo Test-First (Principio III, no negociable)
Para cada tarea de prueba seguida de su implementación:
1. Escribe la prueba.
2. Ejecútala y **muéstrame que falla** (Rojo).
3. Implementa lo mínimo para que pase (Verde).
4. Refactoriza si hace falta, manteniendo las pruebas en verde.

## Reglas de código
- Respeta la estructura y los nombres del plan (`pe.galaxy.orders.{domain,application,infrastructure}`) y el glosario de `data-model.md`.
- Código, base de datos, configuración y API en **inglés**.
- `domain` en Java puro; reglas de negocio solo en el dominio; dinero en `BigDecimal` escala 2 `HALF_UP`.
- Nombres de prueba que citan la regla o el escenario (p. ej. `RN04_upTo5000Inclusive_requiresSupervisorOnly`).
- Sin secretos reales en archivos; credenciales locales solo en `compose.yaml` de desarrollo.

## Al terminar cada tarea
- Marca la tarea como `[x]` en `tasks.md`.

## Al terminar cada fase
- Ejecuta las pruebas de la fase (`mvn test`; las de integración con `mvn verify`, requieren Docker).
- Resume: tareas completadas, pruebas ejecutadas (verde/rojo), archivos creados o modificados y cualquier desviación respecto del plan.
- Propón el mensaje de commit en Conventional Commits con los IDs de tarea, p. ej. `feat(orders): T015-T021 registrar pedido en borrador (US1)`. **No hagas el commit**: lo hago yo tras revisar.

## Si encuentras una diferencia con la spec o el plan
No la resuelvas en el código. Detente y repórtala indicando el artefacto y la sección; la corrección se hace primero en la spec o el plan (Principio I).
