# Instalar el skill de Blender en Claude Code

> Atajo: en el README del repositorio hay un **prompt de un solo copy-paste** que le pide a Claude
> Code hacer todo esto por ti. Esto es la versión manual.

Tres pasos, una sola vez:

## 1. Copiar el skill

Copia la carpeta `skills/blender` de este directorio a tu carpeta de skills de usuario de Claude Code:

**Windows (PowerShell):**
```powershell
xcopy /E /I skills\blender "%USERPROFILE%\.claude\skills\blender"
```

**macOS / Linux:**
```bash
mkdir -p ~/.claude/skills && cp -r skills/blender ~/.claude/skills/blender
```

Resultado esperado: `~/.claude/skills/blender/SKILL.md` (+ carpeta `references`).

## 2. Preparar Blender (si aún no lo hiciste)

Instala el add-on oficial "MCP" (Blender Lab) en Blender ≥ 5.1 con auto-arranque y acceso online —
pasos completos en el README del repositorio (sección "Instalación · Parte 1").

## 3. Registrar el servidor MCP oficial de Blender en Claude Code

Sigue la guía oficial del proyecto blender_mcp y registra el servidor:

https://projects.blender.org/lab/blender_mcp/wiki/Setup

```bash
claude mcp add blender -- <comando del servidor según la guía oficial>
```

## Probar

Abre cualquier proyecto con `claude` y escribe:
**"hazlo en blender: un cubo dorado sobre un piso oscuro y hazme un render"**.
Claude debe abrir Blender solo (si está cerrado), crear la escena y enseñarte el resultado.
