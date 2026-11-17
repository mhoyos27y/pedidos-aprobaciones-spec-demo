# Prompt – Especificación de la feature 001: Registro y aprobación de pedidos

> Entrada para `/speckit-specify`.
> Uso en Claude Code: `/speckit-specify Especifica la feature usando como entrada @docs/prompts/features/001-registro-aprobacion-pedidos/01-specify.md`
> Solo **necesidad de negocio**: no incluir tecnologías, endpoints, tablas ni nombres de clases.

## Nombre corto de la feature
registro-aprobacion-pedidos

## Necesidad de negocio
Los colaboradores de la empresa (**Solicitantes**) registran pedidos de compra con una o más líneas (descripción, cantidad, precio unitario) en soles. El pedido nace en borrador; mientras está en borrador el solicitante puede modificarlo y luego lo envía a aprobación.

Según el monto total, lo aprueba un **Jefe** y, si el monto es alto, además un **Gerente**, siempre en ese orden. Cualquier aprobador puede rechazar el pedido indicando un motivo. El solicitante puede anular su pedido.

Nadie puede aprobar su propio pedido. Se debe dejar constancia de quién registró, envió, aprobó, rechazó o anuló cada pedido y cuándo.

## Necesidades de consulta
- Los solicitantes consultan sus pedidos.
- Los aprobadores necesitan una bandeja con los pedidos pendientes que les corresponden.
- Un **Auditor** necesita ver el historial completo de cada pedido.

## Indicaciones para la spec
- Historias de usuario priorizadas, cada una con prueba independiente y escenarios Dado / Cuando / Entonces (caso principal, alternativo y límite).
- Requisitos FR-xxx, reglas de negocio RN-xx y criterios de éxito SC-xxx medibles y sin tecnología.
- Diagrama Mermaid `stateDiagram-v2` con los estados del pedido.
- Secciones de Supuestos y Fuera de alcance (notificaciones por correo, delegación de aprobaciones, expiración de pendientes, adjuntos, integración con ERP).
- Lo que no se pueda deducir debe quedar como `[NEEDS CLARIFICATION]`; no inventar umbrales ni límites.
