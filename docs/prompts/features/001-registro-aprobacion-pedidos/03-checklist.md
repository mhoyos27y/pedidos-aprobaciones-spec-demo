# Prompt – Checklist de calidad de la feature 001: Registro y aprobación de pedidos

> Entrada para `/speckit-checklist` (Sesión 04).
> Uso en Claude Code: `/speckit-checklist Genera el checklist de calidad de requisitos usando como entrada @docs/prompts/features/001-registro-aprobacion-pedidos/03-checklist.md`
> Un checklist de Spec Kit son "pruebas unitarias para los requisitos": evalúa si la **spec** está bien escrita, no si el código funciona.

## Dominio del checklist
Calidad de requisitos del flujo de aprobación: montos, límites, estados y autorizaciones.

## Dimensiones a evaluar
1. **Completitud**: niveles definidos para todos los rangos de monto; qué pasa al rechazar en un nivel intermedio; quién puede anular y en qué estados; datos mínimos del historial; concurrencia.
2. **Claridad**: umbrales inclusivos o exclusivos; precisión decimal y redondeo; longitud del motivo de rechazo cuantificada; criterios de rendimiento con números.
3. **Consistencia**: máquina de estados coherente con los requisitos; roles iguales en escenarios y entidades; el monto máximo es el mismo en reglas, casos límite y escenarios.
4. **Cobertura de escenarios**: valores frontera (5,000.00 / 5,000.01 y 50,000.00 / 50,000.01); segregación de funciones; acciones sobre estados finales.
5. **Separación negocio / técnica**: la spec no nombra tecnologías; los supuestos técnicos están marcados como supuestos.
6. **Cierre de clarificaciones**: no quedan `[NEEDS CLARIFICATION]`.

## Formato esperado
- Ítems `CHK001`, `CHK002`… redactados como **preguntas** sobre la spec, con la dimensión y la referencia a la sección (p. ej. `[Claridad, Spec §RN-04]`).
- Archivo en `specs/001-…/checklists/`.
