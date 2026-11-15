# Paso 3 – Crear la Project Constitution

**Sesión**: 02 – Project Constitution & Spec-Driven Foundations
**Objetivo**: definir las reglas no negociables del proyecto que Claude Code y el equipo deben respetar en todas las fases: arquitectura, dominio, pruebas, trazabilidad y uso controlado de la IA.
**Requisito previo**: [Paso 2 – Instalar Spec Kit](paso-02-instalar-spec-kit.md) ✅
**Comando**: `/speckit-constitution`
**Artefacto generado**: `.specify/memory/constitution.md`
**Tiempo estimado**: 30–45 minutos

---

## 1. ¿Qué es la constitución?

Es el documento de mayor jerarquía del proyecto. Todas las skills de Spec Kit la leen:

| Fase | Cómo usa la constitución |
|---|---|
| `/speckit-specify` | Mantiene la spec dentro de los límites del proyecto |
| `/speckit-plan` | Incluye una sección **Constitution Check** que debe pasar antes y después del diseño |
| `/speckit-tasks` | Ordena las tareas (p. ej. prueba antes que implementación si exiges Test-First) |
| `/speckit-analyze` | Marca como **CRITICAL** cualquier violación de un principio |
| `/speckit-implement` | Genera código respetando las restricciones técnicas |

Un buen principio es **verificable**: se puede comprobar con una prueba, una revisión o un comando.

| ❌ Vago | ✅ Verificable |
|---|---|
| "El código debe ser de calidad" | "Cobertura de líneas del paquete `domain` ≥ 90 %" |
| "Usar buenas prácticas" | "El paquete `domain` no importa `org.springframework` ni `jakarta`" |
| "Manejar bien el dinero" | "Montos en `BigDecimal`, 2 decimales, `HALF_UP`; prohibido `double`" |

---

## 2. Revisar la plantilla actual

Abre `.specify/memory/constitution.md`: contiene marcadores como `[PROJECT_NAME]`, `[PRINCIPLE_1_NAME]`, `[PRINCIPLE_1_DESCRIPTION]`… La skill los reemplaza con tu contenido.

---

## 3. Ejecutar `/speckit-constitution`

Desde la raíz del proyecto:
```powershell
cd pedidos-aprobaciones
claude
```

### 3.1 Guardar el prompt como archivo versionado
El contenido de la constitución **no se escribe directamente en la terminal**: se guarda en un archivo de entrada dentro del repositorio. Así queda versionado, se revisa en los PR y se reutiliza para enmiendas.

```text
pedidos-aprobaciones/
└── docs/
    └── prompts/
        ├── README.md                 ← índice de prompts
        ├── constitution.md           ← entrada de /speckit-constitution
        └── features/                 ← una carpeta por feature (paso 4 en adelante)
```

Crea la carpeta y copia `constitution.md` (y el `README.md` de prompts) entregados con esta guía:
```powershell
New-Item -ItemType Directory -Force docs/prompts
# copia constitution.md dentro de docs/prompts/
```
Revísalo y ajústalo antes de ejecutar (por ejemplo, la rama principal `master`).

### 3.2 Ejecutar la skill referenciando el archivo
En Claude Code, `@ruta` adjunta el contenido del archivo al prompt:

```text
/speckit-constitution Crea la constitución usando como entrada @docs/prompts/constitution.md
```

> Al escribir `@docs/` Claude Code autocompleta la ruta; selecciona el archivo con `Tab`.

> Si Claude Code pide permiso para editar archivos o ejecutar scripts, revisa lo que propone y acéptalo.

---

## 4. Revisar el resultado

Abre `.specify/memory/constitution.md` y verifica:

| # | Verificación | ✔ |
|---|---|---|
| 1 | No quedan marcadores entre corchetes (`[PRINCIPLE_…]`, `[PROJECT_NAME]`) | ☐ |
| 2 | Hay 7 principios con nombre y reglas en forma **DEBE / NUNCA** | ☐ |
| 3 | Cada principio se puede verificar con una prueba, revisión o comando | ☐ |
| 4 | Están las restricciones técnicas (Java 25, Spring Boot, PostgreSQL, Flyway…) | ☐ |
| 5 | Está la convención de idioma (código en inglés, artefactos en español) | ☐ |
| 6 | Están el flujo y las compuertas (no plan con `[NEEDS CLARIFICATION]`, no implement con CRITICAL) | ☐ |
| 7 | Gobernanza con versión `1.0.0` y fecha de ratificación | ☐ |
| 8 | El **Sync Impact Report** (comentario al inicio) indica qué plantillas revisó | ☐ |

Comprobación rápida en PowerShell:
```powershell
Select-String -Path .specify/memory/constitution.md -Pattern '\[[A-Z_]+\]'   # no debe devolver nada
Select-String -Path .specify/memory/constitution.md -Pattern 'Versión|Version'
```

### Si algo no te convence: edita el archivo y vuelve a ejecutar
1. Modifica `docs/prompts/constitution.md` (p. ej. en el principio 3 agrega "las pruebas de integración usan PostgreSQL real con Testcontainers, nunca H2").
2. Ejecuta indicando el tipo de cambio:
```text
/speckit-constitution Actualiza la constitución con los cambios de @docs/prompts/constitution.md. Es un cambio de redacción (PATCH): sube la versión a 1.0.1.
```

Referencia: compara con la constitución del repositorio de referencia (`.specify/memory/constitution.md` en `pedidos-aprobaciones.zip`, tag `sesion-02`). No tiene que ser idéntica; sí debe cubrir los mismos puntos.

---

## 5. Discusión de la sesión 02

1. ¿Qué principio evitaría que la IA ponga reglas de negocio en el controlador? → **II**
2. ¿Qué pasa si `/speckit-analyze` detecta que una tarea implementa antes de su prueba? → hallazgo **CRITICAL** por el principio **III**
3. ¿Por qué la versión semántica? → para saber si un cambio de la constitución obliga a revisar planes existentes (MAJOR) o no (PATCH)
4. ¿Por qué la regla de idioma va en la constitución y no en el plan? → porque aplica a **todas** las features

---

## 6. Commit

```powershell
git add docs/prompts/constitution.md .specify/memory/constitution.md
git commit -m "docs: ratificar constitución v1.0.0"
git push origin master
git tag sesion-02
git push origin --tags
```

---

## ✅ Checklist del paso 3
- [ ] `docs/prompts/constitution.md` creado (entrada versionada)
- [ ] `constitution.md` sin marcadores y con versión 1.0.0
- [ ] 7 principios verificables + restricciones técnicas + idioma + flujo + gobernanza
- [ ] Commit `docs: ratificar constitución v1.0.0` publicado

**Siguiente**: Paso 4 – Especificar la feature con `/speckit-specify` (Sesión 03).
