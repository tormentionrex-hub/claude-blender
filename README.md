# claude-blender

**Dile a Claude "hazlo en Blender" — y Blender se abre, modela, ilumina y renderiza solo.**

Plugin de **Claude Cowork** + skill de **Claude Code** que le dan a Claude control total de Blender
en tu computadora: creas y editas escenas 3D con simples prompts de texto o imágenes de referencia.
Claude abre Blender si está cerrado, escribe el código `bpy`, verifica el resultado con capturas y
renders, e itera hasta que quede como pediste.

Creado por **Christopher González** · Licencia MIT · [English summary below ⬇](#english-summary)

---

## ¿Para qué se hizo?

Modelar en Blender tiene una curva de aprendizaje enorme. Este proyecto nació para poder **crear
contenido 3D hablando**: prototipos de modelos, escenas de prueba, renders rápidos, inspección de
archivos `.blend` — sin tocar la interfaz de Blender. El objetivo es que cualquier persona con
Claude y Blender instalados pueda decir *"crea un planeta con anillos y luz azul de fondo"* y ver el
resultado en su pantalla en segundos.

## ¿Cómo funciona?

Dos capas que trabajan juntas:

```
   Tú: "hazlo en Blender: una silla de madera estilo nórdico"
    │
    ▼
┌─────────────────────┐   herramientas MCP    ┌──────────────────────────┐
│  Claude             │ ────────────────────► │  Servidor MCP oficial    │
│  (Cowork o Code)    │   execute_blender_code│  de Blender (conector)   │
│  + este skill       │   screenshots, render │                          │
└─────────────────────┘                       └───────────┬──────────────┘
                                                          │ socket localhost:9876
                                              ┌───────────▼──────────────┐
                                              │  Add-on oficial "MCP"    │
                                              │  (Blender Lab) dentro de │
                                              │  Blender ≥ 5.1 → bpy     │
                                              └──────────────────────────┘
```

1. **La plomería** es 100 % oficial: el [proyecto blender_mcp de Blender Lab](https://projects.blender.org/lab/blender_mcp)
   (servidor MCP + add-on) conectado por un socket local en el puerto 9876.
2. **Este repositorio aporta la inteligencia de uso**: una *skill* que le enseña a Claude el flujo
   completo — probar la conexión, abrir Blender solo si hace falta, escribir `bpy` idempotente,
   **verificar visualmente cada cambio** (captura del viewport en modo render / thumbnails), iterar,
   guardar, y esquivar todas las trampas conocidas (puerto ocupado, cierres a mitad de llamada,
   add-ons conflictivos…). Todo eso está documentado en recetas y guía de problemas dentro del skill.

## Contenido del repositorio

| Carpeta | Qué es |
|---|---|
| [`cowork-plugin/`](cowork-plugin/) | Fuente del plugin para **Claude Cowork** (app de escritorio) |
| [`claude-code-skill/`](claude-code-skill/) | Skill para **Claude Code** (terminal) + `INSTALAR.md` |
| [`dist/blender.plugin`](dist/) | Plugin **empaquetado**, listo para instalar en Cowork con un clic |

## Requisitos

- **Blender 5.1 o superior** (el add-on oficial declara mínimo 5.1.0) — [blender.org/download](https://www.blender.org/download/)
- **Claude** en cualquiera de sus formas con MCP: app de escritorio (Cowork) o Claude Code
- Probado en **Windows 11**; en macOS/Linux cambian solo las rutas y la forma de abrir Blender

---

## Instalación · Parte 1: Blender (común a todo)

1. Descarga el add-on oficial: **[mcp-1.0.0.zip](https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip)**
   (página oficial: [blender.org/lab/mcp-server](https://www.blender.org/lab/mcp-server/)).
2. En Blender: **Edit → Preferences → Get Extensions → ⌄ (menú) → Install from Disk…** y elige el ZIP
   (o simplemente arrastra el ZIP a la ventana de Blender).
3. En **Preferences → System** activa **Allow Online Access** (el add-on lo exige).
4. En **Preferences → Add-ons → MCP**, verifica que esté habilitado y con **Auto Start** activado.
5. **Importante:** si tienes otros add-ons MCP de terceros (p. ej. el "Blender MCP" clásico de
   ahujasid), **desactívalos** — usan el mismo puerto 9876 con un protocolo incompatible y la
   conexión se cuelga.
6. Reinicia Blender. Desde ahora, cada vez que Blender abra, el servidor MCP escucha solo en
   `localhost:9876`.

<details>
<summary>⚡ Alternativa: hacer todo el paso 1 desde la consola Python de Blender (Scripting)</summary>

```python
import urllib.request, bpy
urllib.request.urlretrieve('https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip', r'C:\Users\<tu-usuario>\Downloads\mcp-1.0.0.zip')
bpy.ops.extensions.package_install_files(filepath=r'C:\Users\<tu-usuario>\Downloads\mcp-1.0.0.zip', repo='user_default', enable_on_install=True)
bpy.context.preferences.system.use_online_access = True
p = bpy.context.preferences.addons['bl_ext.user_default.mcp'].preferences; p.use_autostart = True
bpy.ops.wm.save_userpref()
```
Luego reinicia Blender (o arranca el servidor ya: `bpy.ops.blmcp.server_start()`).
</details>

## Instalación · Parte 2A: Claude Cowork (app de escritorio)

1. En el app de Claude, agrega el **conector oficial "Blender"** (Ajustes → Conectores).
2. Descarga [`dist/blender.plugin`](dist/blender.plugin) de este repo, envíalo a un chat de Cowork y
   pulsa **Instalar** en la vista previa del archivo.
3. Ya está. En cualquier chat escribe: *"hazlo en blender: …"*. La primera vez que Claude tenga que
   abrir Blender por ti, te pedirá aprobar el control de la computadora (una vez por sesión).

## Instalación · Parte 2B: Claude Code (terminal)

Manualmente: sigue [`claude-code-skill/INSTALAR.md`](claude-code-skill/INSTALAR.md)
(copiar el skill a `~/.claude/skills/` + registrar el servidor MCP oficial con `claude mcp add`
según la [guía oficial](https://projects.blender.org/lab/blender_mcp/wiki/Setup)).

**…o mejor: que Claude Code lo haga todo por ti ⬇**

## 🚀 Instálalo con un solo prompt (Claude Code)

Copia y pega esto en Claude Code:

```text
Instala el control de Blender para Claude desde https://github.com/tormentionrex-hub/claude-blender :

1. Clona el repositorio en una carpeta temporal.
2. Copia claude-code-skill/skills/blender a mi carpeta de skills de usuario
   (~/.claude/skills/blender — en Windows %USERPROFILE%\.claude\skills\blender), creando carpetas si faltan.
3. Localiza mi ejecutable de Blender (versión 5.1 o superior). Si no encuentras ninguno, dime dónde
   descargarlo y detente.
4. Descarga https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip
   a mi carpeta de descargas.
5. Instala y deja configurado el add-on SIN abrir la interfaz, ejecutando Blender en modo
   --background con --python-expr para: bpy.ops.extensions.package_install_files(filepath=<zip>,
   repo='user_default', enable_on_install=True); use_online_access=True;
   preferencias del add-on 'bl_ext.user_default.mcp' con use_autostart=True; y bpy.ops.wm.save_userpref().
6. Si existe algún add-on MCP de terceros activo (p. ej. 'blender_mcp_addon'), desactívalo con
   bpy.ops.preferences.addon_disable + save_userpref en esa misma sesión --background.
7. Registra el servidor MCP oficial de Blender en Claude Code siguiendo
   https://projects.blender.org/lab/blender_mcp/wiki/Setup (claude mcp add blender ...).
8. Verifica de punta a punta: abre Blender, espera ~25 segundos y llama get_objects_summary del MCP.
   Si responde con la escena, crea un cubo dorado sobre un piso oscuro con execute_blender_code,
   hazme un render de prueba y enséñamelo. Al final dime exactamente qué quedó instalado y dónde.
```

## Uso — ejemplos

- *"Hazlo en Blender: un planeta con anillos y una luz azul de fondo"*
- *"Crea en Blender esta silla (imagen adjunta) y hazme un render en Cycles"*
- *"Ábreme Blender y ponle un material metálico dorado al objeto Suzanne"*
- *"Modela un logo 3D con mi texto 'CLOUFT', extrúyelo y dale iluminación de estudio"*
- *"Abre mi archivo escena.blend y dime qué objetos tiene y cuántos polígonos"*

## Solución de problemas (resumen)

| Síntoma | Causa típica | Arreglo |
|---|---|---|
| `Cannot connect ... localhost:9876` | Blender cerrado / servidor no arrancó | Abrir Blender y esperar ~25 s; si persiste: `bpy.ops.blmcp.server_start()` en la consola |
| Timeout de 60 s en cada llamada | Add-on MCP de terceros en el mismo puerto, o conector atascado | Desactivar el add-on de terceros; reiniciar Blender; reiniciar el conector/app |
| `Online access must be enabled` | Acceso online desactivado | Preferences → System → Allow Online Access |
| Blender se cerró y todo murió | Se llamó `quit_blender()` dentro de una llamada MCP | Nunca hacerlo; usar el patrón de timer diferido (ver recetas) |

Guía completa: [`cowork-plugin/skills/blender/references/solucion-problemas.md`](cowork-plugin/skills/blender/references/solucion-problemas.md)
· Recetas `bpy` probadas: [`recetas-bpy.md`](cowork-plugin/skills/blender/references/recetas-bpy.md)

## Créditos y autoría

Este plugin y esta skill fueron **creados por Christopher González**, con la ayuda de la
inteligencia artificial **Claude Code** (Anthropic) como asistente de desarrollo, y construidos
sobre estas tecnologías:

- [Blender](https://www.blender.org/) y su API Python `bpy`
- [Proyecto oficial blender_mcp](https://projects.blender.org/lab/blender_mcp) (servidor MCP + add-on, Blender Lab)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
- Sistema de skills y plugins de Claude (Cowork / Claude Code)

## Licencia

[MIT](LICENSE) © 2026 Christopher González — úsalo, modifícalo y compártelo libremente.

---

## English summary

**Say "do it in Blender" and Claude opens Blender, models, lights and renders for you.**
This repo ships a Claude Cowork plugin (`dist/blender.plugin`) and a Claude Code skill
(`claude-code-skill/`) that teach Claude the full workflow on top of the
[official Blender MCP project](https://projects.blender.org/lab/blender_mcp): test the connection
(`localhost:9876`), auto-launch Blender if closed, write idempotent `bpy` code from your prompts or
reference images, visually verify every change via viewport screenshots/renders, iterate, and save.
Setup: install the official MCP add-on in Blender ≥ 5.1 (enable *Allow Online Access* + *Auto Start*,
disable third-party MCP add-ons that fight over port 9876), then install the plugin (Cowork) or copy
the skill + register the official MCP server (Claude Code). A copy-paste one-shot install prompt for
Claude Code is included above. Created by **Christopher González**, with Claude Code (Anthropic) as
an AI development assistant. MIT licensed.
