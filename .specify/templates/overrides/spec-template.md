# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`

**Created**: [DATE]

**Status**: Draft

**Input**: User description: "$ARGUMENTS"

<!--
  Override del proyecto alineado con la Constitución v1.0.0. La spec describe QUÉ y POR QUÉ,
  sin detalles de implementación. Usa el lenguaje ubicuo del dominio en español
  (Pedido, FlujoAprobacion, Aprobacion, SOLICITANTE, APROBADOR...). Solo lo que esté en una
  spec aprobada se implementa (YAGNI, Principio VIII).
-->

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.

  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Los escenarios Given/When/Then se convierten directamente en pruebas given_when_then (Principio II).
-->

### User Story 1 - [Brief Title] (Priority: P1)

[Describe this user journey in plain language]

**Rol(es)**: [SOLICITANTE | APROBADOR | ADMINISTRADOR]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 2 - [Brief Title] (Priority: P2)

[Describe this user journey in plain language]

**Rol(es)**: [SOLICITANTE | APROBADOR | ADMINISTRADOR]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

[Add more user stories as needed, each with an assigned priority]

### Edge Cases

<!--
  ACTION REQUIRED: Incluye como mínimo los casos que exige la Constitución (Principio V y VI)
  cuando apliquen a la feature.
-->

- ¿Qué ocurre ante una transición de estado no permitida?
- ¿Qué ocurre si el creador del pedido intenta aprobarlo (segregación de funciones)?
- ¿Qué ocurre si dos aprobadores actúan sobre el mismo pedido a la vez (concurrencia)?
- ¿Qué ocurre si una creación se reintenta con la misma `Idempotency-Key`?
- ¿Qué ocurre si el usuario no tiene el rol requerido?
- [Otros casos límite propios de la feature]

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema DEBE [capacidad específica]
- **FR-002**: El sistema DEBE [capacidad específica]
- **FR-003**: Los usuarios con rol [ROL] DEBEN poder [interacción clave]

*Example of marking unclear requirements:*

- **FR-004**: El sistema DEBE enrutar la aprobación según [NEEDS CLARIFICATION: ¿monto, área o rol?]

### Transiciones de estado *(include if feature changes Pedido lifecycle)*

<!--
  Ciclo de vida definido por la Constitución: BORRADOR → ENVIADO → EN_APROBACION →
  APROBADO | RECHAZADO | CANCELADO. Toda transición se audita (quién, cuándo, origen, destino, motivo).
-->

| Desde | Hacia | Acción | Rol autorizado | ¿Motivo obligatorio? |
|-------|-------|--------|----------------|----------------------|
| [ESTADO] | [ESTADO] | [acción] | [ROL] | [Sí/No] |

### Reglas de negocio y errores *(include if feature can reject operations)*

<!-- Cada rechazo de negocio tendrá un código estable documentado en el contrato (RFC 9457). -->

| Regla | Código de error de negocio | Cuándo se produce |
|-------|----------------------------|-------------------|
| [regla] | [CODIGO_EN_MAYUSCULAS] | [condición] |

### Key Entities *(include if feature involves data)*

- **[Entidad 1]**: [Qué representa, atributos clave sin implementación; montos con moneda explícita, fechas en UTC]
- **[Entidad 2]**: [Qué representa, relaciones con otras entidades]

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: [Métrica medible]
- **SC-002**: [Métrica medible, p. ej. "El 100 % de las transiciones queda registrado en la auditoría"]
- **SC-003**: [Métrica medible]

## Assumptions

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right assumptions based on reasonable defaults
  chosen when the feature description did not specify certain details.
-->

- [Supuesto sobre usuarios y roles]
- [Supuesto sobre límites del alcance]
- [Dependencia de otro módulo: pedidos | aprobaciones | catalogo | usuarios]
