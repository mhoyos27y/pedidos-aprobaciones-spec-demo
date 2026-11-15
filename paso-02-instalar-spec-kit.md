# Paso 2 – Instalar GitHub Spec Kit e inicializar el repositorio

**Sesión**: 01 – Introduction to GitHub Spec Kit
**Objetivo**: instalar la CLI `specify` y crear el repositorio del caso **Sistema de Pedidos y Aprobaciones** con la estructura de Spec Kit para Claude Code.
**Requisito previo**: [Paso 1 – Instalar `uv`](paso-01-instalar-uv.md) ✅
**Tiempo estimado**: 10 minutos

> ⚠️ **Versión de Spec Kit**: esta guía usa la sintaxis actual (v1.x): `--integration claude` y comandos con guion (`/speckit-constitution`). Muchos tutoriales antiguos usan `--ai claude` y `/speckit.constitution`; con la versión actual `--ai` da el error `No such option: --ai`.

---

## 1. Verificar prerrequisitos

```bash
uv --version        # Paso 1
git --version       # para versionar el proyecto (y la extensión git de Spec Kit)
claude --version    # Claude Code (suscripción activa)
```

Si `git` no tiene identidad configurada, configúrala (la usarán los commits del curso):
```bash
git config --global user.name  "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"
```

---

## 2. Instalar la CLI de Spec Kit

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

Verifica:
```bash
specify check
```
`specify check` revisa qué herramientas detecta (git, claude, VS Code, etc.). Lo importante es que **git** y **claude** aparezcan como disponibles.

Comprueba también la versión instalada:
```bash
specify version
```
Si tienes una versión antigua (0.x), actualízala:
```bash
uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git
```

> Si `specify` no se reconoce: `uv tool update-shell`, cierra y abre la terminal.

---

## 3. Inicializar el repositorio del caso

Ubícate en tu carpeta de trabajo y ejecuta:

### macOS / Linux
```bash
specify init pedidos-aprobaciones --integration claude
```

### Windows (PowerShell)
```powershell
specify init pedidos-aprobaciones --integration claude --script ps
```

Opciones útiles de `specify init`:
| Opción | Uso |
|---|---|
| `--integration claude` | Instala los comandos de Spec Kit como *skills* de Claude Code (`/speckit-*`) |
| `--script sh` / `--script ps` | Scripts en Bash o PowerShell |
| `--here` (o `.`) | Inicializar en la carpeta actual en vez de crear una nueva |
| `--force` | Combinar con una carpeta que ya tiene archivos, sin confirmar |
| `--no-git` | No inicializar Git (no recomendado en el curso) |
| `--non-interactive` | No hacer preguntas (útil en scripts) |
| `--debug` | Ver detalle si algo falla |

Si omites `--integration`, la terminal te pedirá elegir el agente: selecciona **Claude Code**.

---

## 4. Revisar lo que se generó

```bash
cd pedidos-aprobaciones
code .              # abre VS Code con el proyecto como carpeta raíz
```
> Abre en VS Code la carpeta `pedidos-aprobaciones` (no la carpeta padre del curso): Claude Code y Git trabajan desde la raíz del proyecto.

Estructura esperada (Spec Kit v1, Windows con `--script ps`):
```text
pedidos-aprobaciones/
├── .claude/
│   └── skills/                         # una skill de Claude Code por comando
│       ├── speckit-analyze/SKILL.md
│       ├── speckit-checklist/SKILL.md
│       ├── speckit-clarify/SKILL.md
│       ├── speckit-constitution/SKILL.md
│       ├── speckit-converge/SKILL.md
│       ├── speckit-implement/SKILL.md
│       ├── speckit-plan/SKILL.md
│       ├── speckit-specify/SKILL.md
│       └── speckit-tasks/SKILL.md
└── .specify/
    ├── integrations/
    │   ├── claude.manifest.json        # qué instaló la integración de Claude
    │   └── speckit.manifest.json
    ├── memory/
    │   ├── .constitution-template.json
    │   └── constitution.md             # plantilla (se completa en el Paso 3)
    ├── scripts/powershell/
    │   ├── check-prerequisites.ps1
    │   ├── common.ps1
    │   ├── create-new-feature.ps1      # crea la rama NNN-feature y specs/NNN-feature/
    │   ├── resolve-template.ps1
    │   ├── setup-plan.ps1
    │   └── setup-tasks.ps1
    └── templates/
        ├── checklist-template.md
        ├── constitution-template.md
        ├── plan-template.md
        ├── spec-template.md
        └── tasks-template.md
```
> Los nombres exactos pueden variar ligeramente según la versión de Spec Kit.

### Qué observar (discusión de la sesión 01)
| Archivo | Para qué sirve |
|---|---|
| `.claude/skills/speckit-*/SKILL.md` | Cada skill es un *prompt* versionado: define cómo Claude ejecuta cada fase |
| `.specify/templates/spec-template.md` | Secciones obligatorias de una spec: escenarios, requisitos, criterios de éxito |
| `.specify/templates/plan-template.md` | Incluye el **Constitution Check** |
| `.specify/memory/constitution.md` | Plantilla vacía con marcadores `[PRINCIPLE_1_NAME]`… |
| `.specify/scripts/powershell/` | Crean ramas `NNN-feature` y carpetas `specs/NNN-feature/` |
| `.specify/integrations/*.manifest.json` | Registro de lo que instaló cada integración (permite `specify integration switch`) |

---

## 5. Probar Spec Kit dentro de Claude Code

```bash
claude
```
En Claude Code escribe `/speckit` y verifica que aparecen los comandos (con **guion**):

```text
/speckit-constitution   /speckit-specify   /speckit-clarify   /speckit-checklist
/speckit-plan           /speckit-tasks     /speckit-analyze   /speckit-implement
```

> Abre `claude` **desde la carpeta del proyecto**; si no, las skills no se cargan.

Sal con `/exit` (todavía no ejecutes ninguno; la constitución es el Paso 3).

---

## 6. Primer commit y publicación en GitHub

### ⚠️ El repositorio Git va dentro de `pedidos-aprobaciones`
Los scripts de Spec Kit usan la raíz del repositorio Git (`git rev-parse --show-toplevel`) para crear `specs/NNN-feature/` y las ramas de cada feature. Por eso `git init` se ejecuta **dentro de `pedidos-aprobaciones`**, no en la carpeta del curso. Si lo haces en la carpeta padre, `specs/` quedará fuera del proyecto.

```text
CURS-000687-SDD-Spect-Kit-Dev\        ← carpeta del curso (SIN .git)
└── pedidos-aprobaciones\             ← raíz del repositorio Git
    ├── .git    ├── .claude    ├── .specify    └── README.md
```

### 6.1 Crear el repositorio local y el primer commit (PowerShell)
```powershell
cd D:\AmbDesarrollo\Cursos\InTraining\Vive\CURS-000687-SDD-Spect-Kit-Dev\pedidos-aprobaciones

# ¿specify init ya creó el repositorio?
Test-Path .git                      # True → omite la siguiente línea
git init                            # solo si salió False

# README en UTF-8
"# CURS-000687-SDD-Spect-Kit-Dev" | Set-Content -Encoding utf8 README.md

# Primer commit con todo lo generado por Spec Kit
git add .
git commit -m "chore: inicializar repositorio con GitHub Spec Kit"
```

> **No uses** `echo "…" >> README.md` en Windows PowerShell 5.1: guarda el archivo en UTF-16 y GitHub lo muestra con caracteres extraños.
> **No uses** `git add README.md` solo: hay que versionar también `.claude/` y `.specify/`.

### Convención de mensajes de commit (Conventional Commits)
La constitución (Principio IV) exige mensajes con el formato:
```text
<tipo>(<ámbito opcional>): <descripción>
```
Este primer commit usa `chore` porque inicializar el repositorio con Spec Kit es **mantenimiento/configuración**: no agrega funcionalidad, no corrige errores ni cambia el comportamiento del sistema.

**Tipos que usaremos en el curso**

| Tipo | Cuándo | Ejemplo del caso |
|---|---|---|
| `feat` | Nueva funcionalidad para el usuario | `feat(orders): T021 aprobar pedido nivel 1` |
| `fix` | Corrección de un error | `fix(orders): T030 rechazo sin motivo devolvía 500` |
| `docs` | Solo documentación o artefactos SDD | `docs(spec): 001 clarificaciones y checklist` |
| `test` | Agregar o corregir pruebas | `test(orders): T022 pruebas de aprobación por nivel` |
| `refactor` | Cambio interno sin cambiar el comportamiento | `refactor(orders): extraer ApprovalPolicy` |
| `build` | Dependencias, Maven, Docker | `build: agregar Testcontainers al pom` |
| `ci` | Pipelines (GitHub Actions) | `ci: ejecutar mvn verify en cada PR` |
| `chore` | Mantenimiento que no encaja en los anteriores | `chore: inicializar repositorio con GitHub Spec Kit` |

> Con el tipo y el ID de tarea (`feat(orders): T021 …`) se mantiene la trazabilidad Necesidad → US → FR → Tarea → Commit → Prueba, y `git log --oneline` separa el trabajo funcional (`feat`/`fix`) del de soporte (`docs`/`chore`).

Si Git no tiene tu identidad configurada:
```powershell
git config --global user.name  "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"
```

### 6.2 Conectar con el repositorio del curso en GitHub
El repositorio remoto ya existe en la organización **GalaxyTraining** (vacío):

```powershell
git branch -M master
git remote add origin https://github.com/GalaxyTraining/CURS-000687-SDD-Spect-Kit-Dev.git
git push -u origin master
```

| Detalle | Nota |
|---|---|
| Rama principal | `master` (convención del repositorio del curso). En las guías, donde diga `main`, usa `master` |
| Autenticación | El primer `push` por HTTPS abre la ventana de inicio de sesión de GitHub (Git Credential Manager). Tu cuenta necesita permiso de escritura en **GalaxyTraining** |
| Nombre distinto | El repositorio se llama `CURS-000687-SDD-Spect-Kit-Dev` y la carpeta local `pedidos-aprobaciones`: es válido |

### 6.3 Verificar
```powershell
git log --oneline       # 1 commit: chore: inicializar repositorio con GitHub Spec Kit
git remote -v           # origin → GalaxyTraining/CURS-000687-SDD-Spect-Kit-Dev
git status              # nothing to commit, working tree clean
```
En GitHub deben verse en la raíz: `.claude/`, `.specify/` y `README.md`.

### 6.4 Marcar el fin de la sesión (opcional)
```powershell
git tag sesion-01
git push origin --tags
```

---

## 7. Solución de problemas

| Problema | Solución |
|---|---|
| `specify: command not found` | `uv tool update-shell` y reabrir la terminal |
| Error de rate limit / descarga desde GitHub | `specify init … --github-token <TOKEN>` o define `GH_TOKEN` |
| La carpeta ya existe | Usa otro nombre, o `specify init --here --force --integration claude --script ps` dentro de ella |
| `No such option: --ai` | Versión actual de Spec Kit: usa `--integration claude` |
| No aparecen los `/speckit-*` en Claude Code | Abre `claude` **desde la raíz del proyecto**; verifica que exista `.claude/skills/` |
| En Windows los scripts `.sh` fallan | Reinicializa con `--script ps` |
| Proxy corporativo | Configura `HTTPS_PROXY` antes de instalar/inicializar |
| `specs/` se crea fuera de `pedidos-aprobaciones` | El `.git` está en la carpeta padre: el repositorio debe estar dentro del proyecto |
| `remote origin already exists` | `git remote set-url origin https://github.com/GalaxyTraining/CURS-000687-SDD-Spect-Kit-Dev.git` |
| `push` rechazado (403 / permission denied) | Pide acceso de escritura a la organización GalaxyTraining o verifica la cuenta en Git Credential Manager |
| `push` rechazado (fetch first) | El remoto ya tiene commits: `git pull origin master --allow-unrelated-histories` y vuelve a hacer `push` |
| README con caracteres raros en GitHub | Se guardó en UTF-16: `Get-Content README.md \| Set-Content -Encoding utf8 README.md` |

---

## ✅ Checklist del paso 2
- [ ] `specify check` detecta git y claude
- [ ] Existe la carpeta `pedidos-aprobaciones` con `.specify/` y `.claude/skills/`
- [ ] Claude Code muestra los comandos `/speckit-*`
- [ ] `.git` dentro de `pedidos-aprobaciones` (no en la carpeta del curso)
- [ ] Primer commit publicado en `GalaxyTraining/CURS-000687-SDD-Spect-Kit-Dev` (rama `master`)

**Siguiente**: Paso 3 – Crear la Project Constitution con `/speckit-constitution` (Sesión 02).
