# Paso 1 – Instalar `uv`

**Objetivo**: instalar `uv`, el gestor de paquetes de Python que se usa para instalar la CLI de GitHub Spec Kit (`specify`).
**Tiempo estimado**: 5 minutos
**Siguiente paso**: [Paso 2 – Instalar Spec Kit](#5-siguiente-paso-instalar-spec-kit)

---

## 1. ¿Qué es `uv`?

`uv` es un gestor de paquetes y entornos de Python creado por **Astral** y escrito en Rust. Reemplaza a `pip`, `pipx` y `virtualenv`, y es mucho más rápido que ellos.

En el curso lo usamos para un único propósito:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

| Parte del comando | Qué hace |
|---|---|
| `uv tool install` | Instala un programa de línea de comandos en su propio entorno aislado y lo agrega al `PATH` |
| `specify-cli` | Paquete que se instala; deja disponible el comando `specify` |
| `--from git+https://…/spec-kit.git` | Descarga la versión más reciente desde el repositorio de GitHub, no desde PyPI |

> No necesitas tener Python instalado: si falta, `uv` descarga la versión que necesita.

---

## 2. Instalación

Elige **una** opción según tu sistema operativo.

### Windows (PowerShell)
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### macOS / Linux
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Alternativas
| Método | Comando |
|---|---|
| Homebrew (macOS) | `brew install uv` |
| WinGet (Windows) | `winget install --id=astral-sh.uv -e` |
| Scoop (Windows) | `scoop install main/uv` |
| pip (si ya tienes Python) | `pip install uv` |

> **Importante**: al terminar, **cierra y vuelve a abrir la terminal** (o VS Code) para que se actualice el `PATH`.

---

## 3. Verificación

```bash
uv --version
```
Resultado esperado (el número puede variar):
```text
uv 0.x.y
```

Verifica también que el directorio de herramientas está en el `PATH`:
```bash
uv tool dir --bin
```

---

## 4. Solución de problemas

| Problema | Causa probable | Solución |
|---|---|---|
| `uv: command not found` / `"uv" no se reconoce como un comando` | La terminal no recargó el `PATH` | Cierra y abre la terminal. Si persiste, ejecuta `uv tool update-shell` y vuelve a abrirla |
| PowerShell bloquea el script | Política de ejecución restrictiva | Usa el comando con `-ExecutionPolicy ByPass` mostrado arriba |
| `specify` no se reconoce después de instalarlo | El directorio de herramientas de `uv` no está en el `PATH` | `uv tool update-shell` y reabre la terminal |
| Error de certificado SSL / proxy corporativo | Red de la empresa intercepta HTTPS | Configura `HTTPS_PROXY` o instala desde una red sin proxy; consulta a soporte técnico |
| `git` no encontrado al instalar `specify-cli` | `--from git+…` requiere Git | Instala Git y verifica con `git --version` |

---

## 5. Siguiente paso: instalar Spec Kit

Con `uv` funcionando:

```bash
# Instalar la CLI de Spec Kit
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# Verificar
specify check
```

### Comandos útiles de `uv` para el curso
| Comando | Uso |
|---|---|
| `uv tool list` | Ver herramientas instaladas con `uv` |
| `uv tool upgrade specify-cli` | Actualizar Spec Kit |
| `uv tool uninstall specify-cli` | Desinstalar Spec Kit |
| `uvx --from git+https://github.com/github/spec-kit.git specify init <proyecto> --ai claude` | Ejecutar `specify` una sola vez, sin instalarlo |
| `uv self update` | Actualizar `uv` |

---

## ✅ Checklist del paso 1
- [ ] `uv --version` muestra una versión
- [ ] La terminal reconoce `uv` después de reabrirla
- [ ] `git --version` funciona (requisito del paso 2)
