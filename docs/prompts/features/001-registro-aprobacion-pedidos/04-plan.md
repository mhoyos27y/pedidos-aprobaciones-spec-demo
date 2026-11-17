# Prompt – Plan técnico de la feature 001: Registro y aprobación de pedidos

> Entrada para `/speckit-plan` (Sesión 05).
> Uso en Claude Code: `/speckit-plan Genera el plan técnico usando como entrada @docs/prompts/features/001-registro-aprobacion-pedidos/04-plan.md`
> Aquí sí va la **tecnología** (el *cómo*). El *qué* ya está en `spec.md` y no debe cambiar en este paso.

## Stack
- Java 25, Spring Boot 3.5.x (Web, Data JPA, Validation, Actuator), Maven.
- PostgreSQL 17 con migraciones Flyway (`ddl-auto=validate`).
- springdoc-openapi para exponer el contrato.
- Docker Compose para PostgreSQL en local.

## Arquitectura
Hexagonal (puertos y adaptadores) en un único módulo Maven, paquete raíz `pe.galaxy.orders`:
- `domain` (Java puro, sin Spring/JPA/Jackson): agregado `PurchaseOrder`, `OrderLine`, `HistoryEntry`, enums `OrderStatus`, `ApprovalLevel`, `Role`, `HistoryAction`, record `User`; política de aprobación; excepciones de dominio. **Sin puertos.**
- `application`:
  - **Puertos de entrada** en `application/port/in`, agrupados por actor: `ManagePurchaseOrderUseCase` (Solicitante: create, update, submit, cancel), `ApprovePurchaseOrderUseCase` (Jefe/Gerente: approve, reject), `QueryPurchaseOrderUseCase` (get, myOrders, inbox, history); comandos en `application/port/in/command`.
  - **Puertos de salida** en `application/port/out` (`PurchaseOrderPort`).
  - `application/service/PurchaseOrderService` implementa los puertos de entrada y usa los de salida.
- `infrastructure`: controlador REST (depende **solo** de los puertos de entrada), adaptador de persistencia `PurchaseOrderPersistenceAdapter` que implementa el puerto de salida (entidades JPA + `PurchaseOrderJpaRepository` de Spring Data + mapeo dominio ↔ entidad), configuración.
- **Convención de nombres**: puertos de entrada con sufijo `UseCase`; puertos de salida con sufijo `Port`; implementaciones de puertos con sufijo `Adapter` (en pruebas, `InMemoryPurchaseOrderAdapter`).
- Las reglas de negocio viven **solo** en el dominio; controlador y repositorio no deciden reglas.
- Dependencias: `infrastructure → application → domain`. El dominio no depende de `application` ni de `infrastructure`; `application` no depende de `infrastructure`.

## Decisiones de diseño
- **Política de aprobación configurable**: estrategia `ApprovalPolicy` (`AmountBasedApprovalPolicy` + `Threshold`) cuyos umbrales se leen de `application.yml` (`orders.approval.thresholds`), para absorber cambios futuros de montos y niveles sin tocar el agregado.
- **Identidad**: llega de un gateway externo en las cabeceras `X-User-Id` y `X-User-Roles`; se convierte en `User` de dominio (sin Spring Security en esta feature).
- **Concurrencia**: bloqueo optimista con `@Version` → HTTP 409.
- **Código de pedido**: secuencia de PostgreSQL → `PED-{año}-{%06d}`.
- **Historial**: inmutable, solo INSERT.
- **Dinero**: `BigDecimal(15,2)`.

## Convención de idioma
Todo el código, la base de datos, la configuración y la API en **inglés**. Documentar en `data-model.md` un **glosario spec → código**:
- Pedido → `PurchaseOrder`; Línea → `OrderLine`; Historial → `HistoryEntry`.
- Solicitante → `REQUESTER`; Jefe → `SUPERVISOR`; Gerente → `MANAGER`; Auditor → `AUDITOR`.
- BORRADOR / PENDIENTE_APROBACION / APROBADO / RECHAZADO / ANULADO → `DRAFT` / `PENDING_APPROVAL` / `APPROVED` / `REJECTED` / `CANCELLED`.
- Acciones del historial: `CREATED`, `UPDATED`, `SUBMITTED`, `APPROVED`, `REJECTED`, `CANCELLED`.

## API REST
Base `/api/orders`:
- `POST /api/orders` (crear), `PUT /{id}` (modificar), `GET /{id}`, `GET ?status=` (mis pedidos).
- `POST /{id}/submission`, `POST /{id}/approval`, `POST /{id}/rejection`, `POST /{id}/cancellation`.
- `GET /inbox` (bandeja), `GET /{id}/history` (historial).
- Errores RFC 9457 (`application/problem+json`) con propiedad `rule` (p. ej. `RN-03`): 400 validación, 401 sin identidad, 403 rol/segregación, 404 no encontrado, 409 transición inválida o concurrencia, 422 regla de negocio.
- Contrato OpenAPI 3.1 en `contracts/openapi.yaml`.

## Persistencia
Tablas `purchase_order`, `purchase_order_line`, `purchase_order_history`; secuencia `purchase_order_code_seq`; índices para la bandeja `(status, current_level)` y para "mis pedidos" `(requester_id, status)`. Columna `current_level` desnormalizada para cumplir el criterio de rendimiento de la bandeja.

## Estrategia de pruebas
- Unitarias de dominio (JUnit 5 + AssertJ) por regla de negocio, incluidos los valores frontera.
- Casos de uso con un adaptador en memoria (`InMemoryPurchaseOrderAdapter`).
- Integración por escenario de aceptación: `@SpringBootTest` + MockMvc + Testcontainers (PostgreSQL real).
- Prueba de arquitectura: `domain` no importa frameworks ni depende de `application`/`infrastructure`; `application` no depende de `infrastructure`; `infrastructure.web` solo usa `application.port.in`; dinero sin `double`/`float`.
- Nombres de prueba que citan la regla o el escenario (p. ej. `RN04_upTo5000Inclusive_requiresSupervisorOnly`, `US2_scenario6_…`).

## Resultado esperado
- `plan.md` con **Constitution Check** (todos los principios ✅, antes y después del diseño) y tabla de trazabilidad operación → FR/RN.
- `research.md` con cada decisión en formato *Decisión / Razón / Alternativas*.
- `data-model.md` con glosario, modelo de dominio, validaciones con su RN de origen y modelo relacional (Mermaid).
- `contracts/openapi.yaml` y `quickstart.md` con un recorrido `curl` de punta a punta.
- No modificar `spec.md`. Si detectas un vacío de negocio, repórtalo en lugar de decidirlo.
