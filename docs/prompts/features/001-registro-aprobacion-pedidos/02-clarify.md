# Prompt – Clarificación de la feature 001: Registro y aprobación de pedidos

> Entrada para `/speckit-clarify` (Sesión 04).
> Uso en Claude Code: `/speckit-clarify Clarifica la spec usando como foco y fuente de decisiones @docs/prompts/features/001-registro-aprobacion-pedidos/02-clarify.md`
> Este archivo es el **registro de decisiones de negocio**: lo redacta y aprueba el dueño del producto, no la IA.

## Foco de la clarificación
Montos, límites, estados y autorizaciones del flujo de aprobación. No abordar decisiones técnicas (base de datos, API, frameworks): esas van en el plan.

## Decisiones de negocio
Usa estas respuestas cuando la pregunta coincida. Si surge una pregunta que **no** está aquí, pregúntala y espera mi respuesta; no la decidas por tu cuenta.

| # | Pregunta | Decisión |
|---|---|---|
| D1 | ¿Cuál es el umbral entre aprobación del Jefe y Jefe + Gerente? ¿Es inclusivo? | 5,000.00 PEN, inclusivo: total ≤ 5,000.00 requiere solo Jefe; > 5,000.00 requiere Jefe y luego Gerente |
| D2 | ¿Existe un monto máximo por pedido? | Sí, 50,000.00 PEN (inclusivo). Un pedido mayor se rechaza al registrarlo o modificarlo |
| D3 | ¿Un Gerente puede aprobar en lugar del Jefe? | No. Aprobación estrictamente secuencial: cada nivel solo lo aprueba su rol |
| D4 | ¿Un rechazo en el nivel del Jefe termina el flujo? | Sí. RECHAZADO es estado final; el solicitante debe registrar un pedido nuevo |
| D5 | ¿Se puede anular un pedido que ya tiene la aprobación del Jefe? | Sí, mientras siga en PENDIENTE_APROBACION. Solo lo anula su solicitante |
| D6 | ¿Qué moneda se usa? | Solo soles (PEN); sin conversión |
| D7 | ¿Qué pasa con precios de más de 2 decimales? | Se rechazan; no se redondean en silencio |
| D8 | ¿Longitud del motivo de rechazo? | Entre 10 y 500 caracteres |
| D9 | ¿Cuántas líneas puede tener un pedido? | Entre 1 y 50 |
| D10 | ¿Qué significa "la bandeja debe ser rápida"? | Responde en menos de 1 segundo con 10,000 pedidos registrados |
| D11 | ¿Anular un pedido exige motivo? | Sí, obligatorio, entre 10 y 500 caracteres |
| D12 | ¿El comentario es obligatorio al aprobar? | No; opcional, hasta 500 caracteres; si se indica, queda en el historial |
| D13 | ¿Límites de la descripción y la cantidad de una línea? | Descripción de 3 a 200 caracteres; cantidad entera de 1 a 9,999 |
| D14 | ¿Orden y datos de la bandeja del aprobador? | Más antiguo primero (fecha de envío ascendente); número, solicitante, fecha de envío y total |
| D15 | ¿Puede el Solicitante ver el historial de sus pedidos? | Sí, solo el de sus propios pedidos; el Auditor ve el de todos |
| D16 | ¿Cuánto tiempo se conserva el historial? | Indefinidamente; sin borrado ni depuración en esta versión (cualquier política requiere CR) |
| D17 | ¿Cómo se cuentan los espacios en motivos y comentarios? | Sin espacios iniciales ni finales; solo espacios cuenta como vacío |
| D18 | ¿Cómo se mide el "menos de 1 segundo" de la bandeja? | El 95 % de las consultas responde en menos de 1 segundo |
| D19 | ¿Cómo se mide SC-001? | Solicitantes nuevos, sin ayuda, desde el inicio del registro hasta el envío; el 90 % en menos de 3 minutos |
| D20 | ¿Composición de los 10,000 pedidos de la prueba de rendimiento? | Todos los estados; 500 pendientes del nivel del aprobador que consulta |
| D21 | ¿Un precio `10.500` cuenta como más de 2 decimales? | No; los ceros finales no cuentan (10.500 = 10.50). Se rechaza solo si hay cifras distintas de cero después del 2.º decimal (10.555) |
| D22 | ¿Quién ve el detalle (con líneas) de un pedido? | El Solicitante sus pedidos; Jefe/Gerente los pendientes que pueden atender (los de su bandeja); el Auditor cualquiera |
| D23 | ¿Bandeja de un usuario con roles Jefe y Gerente? | Una sola bandeja con ambos niveles, ordenada por fecha de envío, indicando el nivel de cada pedido |
| D24 | ¿Orden y filtro de "mis pedidos"? | Fecha de registro descendente (más reciente primero); filtro opcional por estado |
| D25 | ¿La feature incluye interfaz de usuario? | No; solo el servicio (API). La interfaz es otra feature |
| D26 | ¿Cómo se verifica SC-001 sin interfaz? | Se traslada a la feature de interfaz (sustituye la medición de D19); aquí: registrar y enviar en 2 operaciones |
| D27 | ¿Qué es "un mensaje" en bandeja y lista vacías? | El servicio devuelve una lista vacía; el texto lo muestra el cliente |

## Resultado esperado
- Sección `## Clarificaciones` con la sesión de hoy y cada pregunta → respuesta.
- Reglas, escenarios, casos límite y criterios de éxito actualizados con las decisiones (no solo listadas al final).
- **Cero** marcadores `[NEEDS CLARIFICATION]` en la spec.
