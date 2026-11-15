# Paso 4 – Especificar la feature

**Sesión**: 03 – Feature Specification & Requirements Modeling
**Objetivo**: convertir la necesidad de negocio en una especificación con actores, historias, escenarios, reglas de negocio y criterios de aceptación verificables, **sin hablar de tecnología**.
**Requisito previo**: [Paso 3 – Constitución](paso-03-crear-constitution.md) ✅ (versión 1.0.0 en `master`)
**Comando**: `/speckit-specify`
**Artefacto generado**: `specs/001-<nombre-feature>/spec.md`
**Tiempo estimado**: 45–60 minutos

---

## 1. Qué va (y qué NO va) en una spec

| ✅ Va en la spec (el **qué** y el **por qué**) | ❌ No va (es del plan, paso 6) |
|---|---|
| Actores: Solicitante, Jefe, Gerente, Auditor | Spring Boot, PostgreSQL, Flyway |
| Historias de usuario priorizadas (US1, US2…) | Endpoints, JSON, códigos HTTP |
| Escenarios **Dado / Cuando / Entonces** | Tablas, columnas, índices |
| Reglas de negocio (RN-xx) y requisitos (FR-xxx) | Clases, paquetes, nombres en inglés |
| Estados del pedido y sus transiciones | Frameworks de pruebas |
| Criterios de éxito medibles (SC-xxx) | Docker, CI/CD |

> La regla de idioma de la constitución aplica aquí: la spec se escribe en **español y en lenguaje de negocio** (Jefe, Gerente, BORRADOR). Los nombres en inglés aparecen recién en el plan.

---

### 1.1 ¿Una feature o varias?

Registro y aprobación son **requerimientos diferentes**, y en la spec quedan separados. Pero en Spec Kit **una spec = una feature**, no un requerimiento:

- **Feature**: incremento de valor que se especifica, planifica y entrega como un todo (`specs/001-…/`).
- **Requerimiento**: cada uno es una **historia de usuario** con prioridad, prueba independiente y su propia fase en `tasks.md`.

| Historia | Requerimiento | Prioridad | Fase en tasks |
|---|---|---|---|
| US1 | Registrar pedido en borrador | P1 (MVP) | Fase 3 |
| US2 | Enviar y aprobar según monto | P1 | Fase 4 |
| US3 | Rechazar o anular | P2 | Fase 5 |
| US4 | Consultas, bandeja e historial | P3 | Fase 6 |

**Por qué registro y aprobación van en la misma feature**
1. **Mismo agregado**: ambos operan sobre el mismo pedido y su única máquina de estados (BORRADOR → PENDIENTE → APROBADO…).
2. **Reglas que cruzan ambos**: el límite de monto (RN-03) se valida al *registrar* pero existe por la *aprobación* (RN-04); la segregación de funciones (RN-05) relaciona a quien registra con quien aprueba.
3. **La aprobación sola no aporta valor**: depende de que exista el registro; no es probable de forma aislada.
4. **`/speckit-analyze` revisa una feature a la vez**: separadas, no detectaría contradicciones entre ambas specs.

**Cuándo SÍ conviene separarlos**

| Señal | Ejemplo |
|---|---|
| Se entregan en momentos distintos | Registro en enero, aprobación en marzo |
| Equipos distintos | Un equipo de compras y otro de finanzas |
| Agregados o modelos distintos | Registro crea pedidos; aprobación gestiona presupuestos |
| La spec crece demasiado | Más de 6–8 historias o difícil de revisar en un PR |

Si se separan, quedarían `001-registro-pedidos` y `002-aprobacion-pedidos`; la 002 declara la dependencia de la 001 y referencia su máquina de estados. Es lo que se hace con el **CR-001** de la Sesión 08 (`002-…` como *delta* de la 001).

> **Decisión del curso**: una sola feature (así lo plantea el temario). Lo que importa verificar es que **cada requerimiento sea una historia independiente con su propia prueba** (ver verificación 12 de la sección 4).

---

## 2. Ejecutar `/speckit-specify`

Asegúrate de estar en `master` y sin cambios pendientes:
```powershell
cd D:\AmbDesarrollo\Cursos\InTraining\Vive\CURS-000687-SDD-Spect-Kit-Dev\pedidos-aprobaciones
git status            # working tree clean
claude
```

### 2.1 Archivo de entrada
Igual que con la constitución, la necesidad de negocio se guarda como archivo versionado, en **una carpeta por feature** con un archivo por fase:
```text
docs/prompts/features/001-registro-aprobacion-pedidos/
└── 01-specify.md        ← entrada de /speckit-specify (luego 02-clarify.md, 03-checklist.md, 04-plan.md…)
```
```powershell
New-Item -ItemType Directory -Force docs/prompts/features/001-registro-aprobacion-pedidos
```
Copia el archivo `01-specify.md` entregado con esta guía en esa carpeta y revísalo: debe contener **solo necesidad de negocio** (sin tecnología).

### 2.2 Ejecutar la skill
```text
/speckit-specify Especifica la feature usando como entrada @docs/prompts/features/001-registro-aprobacion-pedidos/01-specify.md
```

### Qué hace la skill
1. Ejecuta `.specify/scripts/powershell/create-new-feature.ps1`: asigna el número `001`, crea la carpeta `specs/001-…/` y, si la integración con Git está activa, la rama `001-…`.
2. Llena `spec-template.md` con tu necesidad.
3. Marca con `[NEEDS CLARIFICATION: …]` lo que no puede deducir (normalmente solo los puntos más importantes; el resto lo resuelve con supuestos que debes revisar).
4. Puede generar un checklist de calidad en `specs/001-…/checklists/`.

Comprueba la rama y la carpeta:
```powershell
git branch --show-current     # p. ej. 001-registro-aprobacion-pedidos
Get-ChildItem specs -Recurse
```
> Si sigues en `master`, crea la rama tú mismo antes de hacer commit:
> `git switch -c 001-registro-aprobacion-pedidos`

---

## 3. Qué esperar en la primera versión

**Es normal que tenga vacíos.** Por ejemplo:
```markdown
- **RN-04**: Pedidos de monto bajo los aprueba el JEFE; los de monto alto, JEFE y GERENTE
  [NEEDS CLARIFICATION: ¿cuál es el umbral? ¿es inclusivo?]
- **RN-03**: total > 0 [NEEDS CLARIFICATION: ¿existe un monto máximo por pedido?]
- **RN-07**: El rechazo exige motivo [NEEDS CLARIFICATION: longitud mínima/máxima]
```
**No los respondas todavía**: se resuelven con `/speckit-clarify` en el paso 5. Esta separación es parte de lo que se evalúa en la sesión.

---

## 4. Revisar la especificación

Abre `specs/001-…/spec.md` y verifica:

| # | Verificación | ✔ |
|---|---|---|
| 1 | Historias **US1–US4** con prioridad (P1, P2, P3) y "Prueba independiente" | ☐ |
| 2 | Escenarios de aceptación en formato **Dado / Cuando / Entonces** | ☐ |
| 3 | Actores: Solicitante, Jefe, Gerente, Auditor | ☐ |
| 4 | Estados BORRADOR → PENDIENTE_APROBACION → APROBADO / RECHAZADO / ANULADO | ☐ |
| 5 | Requisitos **FR-xxx** con "El sistema DEBE…" | ☐ |
| 6 | Reglas de negocio **RN-xx** (líneas, total, niveles, segregación, edición, rechazo, estados finales) | ☐ |
| 7 | Casos límite (montos frontera, acciones sobre estados finales, concurrencia) | ☐ |
| 8 | Criterios de éxito **SC-xxx** medibles y sin tecnología | ☐ |
| 9 | Supuestos explícitos (p. ej. autenticación externa) y **Fuera de alcance** | ☐ |
| 10 | No aparecen "Spring", "endpoint", "tabla", "API REST", "JSON" | ☐ |
| 11 | Hay marcadores `[NEEDS CLARIFICATION]` para umbrales y límites (se verán en el paso 5) | ☐ |
| 12 | Cada requerimiento (registrar, aprobar, rechazar/anular, consultar) es una **historia independiente** con su propia prueba | ☐ |

Comprobaciones rápidas:
```powershell
$spec = Get-ChildItem specs -Recurse -Filter spec.md | Select-Object -First 1
Select-String -Path $spec -Pattern 'NEEDS CLARIFICATION'                       # lista los vacíos
Select-String -Path $spec -Pattern 'Spring|endpoint|PostgreSQL|JSON|tabla'     # debe estar vacío
```

### Pedir mejoras (sin resolver las clarificaciones)
Si falta algo que está en el archivo de entrada (diagrama de estados, Fuera de alcance, escenarios alternativos y límite):
```text
Revisa la spec contra @docs/prompts/features/001-registro-aprobacion-pedidos/01-specify.md y completa lo que falte de la sección "Indicaciones para la spec". No resuelvas los [NEEDS CLARIFICATION].
```
Si la necesidad de negocio cambia, **edita primero el archivo de entrada** y luego pide la actualización: así el cambio queda trazado en Git.

### Comparar con la referencia
En `pedidos-aprobaciones.zip`: `git show sesion-03:specs/001-registro-aprobacion-pedidos/spec.md`. Tu spec no tiene que ser idéntica, pero debe cubrir las mismas historias, reglas y casos límite.

---

## 5. Discusión de la sesión 03

1. ¿Por qué "Nadie puede aprobar su propio pedido" es una **regla de negocio** y no un detalle de seguridad técnica?
2. ¿Qué diferencia hay entre un **requisito funcional** (FR) y una **regla de negocio** (RN)?
3. ¿Cómo sabes que un criterio de éxito es medible? (`"La bandeja debe ser rápida"` vs `"< 1 s con 10,000 pedidos"`)
4. ¿Qué historia es el **MVP** y por qué puede probarse de forma independiente?
5. ¿Registro y aprobación deberían ser features distintas? ¿Qué señales lo justificarían? (ver 1.1)

---

## 6. Commit

```powershell
git add docs/prompts/features/001-registro-aprobacion-pedidos/01-specify.md specs
git commit -m "docs(spec): 001 especificación inicial de pedidos y aprobaciones"
git push -u origin (git branch --show-current)
git tag sesion-03
git push origin --tags
```

> El trabajo de la feature vive en la rama `001-…`. Se integra a `master` con un Pull Request al terminar la implementación (paso 9).

---

## ✅ Checklist del paso 4
- [ ] `docs/prompts/features/001-registro-aprobacion-pedidos/01-specify.md` creado
- [ ] Rama `001-…` creada y carpeta `specs/001-…/spec.md` generada
- [ ] Las 12 verificaciones de la sección 4 revisadas
- [ ] Los `[NEEDS CLARIFICATION]` siguen abiertos (se resuelven en el paso 5)
- [ ] Commit `docs(spec): 001 …` publicado en la rama de la feature

**Siguiente**: Paso 5 – Clarificar y validar la calidad con `/speckit-clarify` y `/speckit-checklist` (Sesión 04).
