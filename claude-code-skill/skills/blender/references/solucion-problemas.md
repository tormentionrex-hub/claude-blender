# Solución de problemas y mapa del setup

## Mapa de la instalación

| Pieza | Detalle |
|---|---|
| Blender | ≥ 5.1 (el add-on oficial declara mínimo 5.1.0). En Windows el lanzador es `blender-launcher.exe` y la ventana pertenece a `blender.exe` |
| Conector (lado Claude) | Conector oficial "Blender" del app de escritorio → herramientas `mcp__remote-devices__Blender__*` (en Claude Code: el servidor MCP oficial registrado con `claude mcp add`) |
| Add-on (lado Blender) | Oficial "MCP" v1.0.0 (Blender Lab), instalado como extensión en `%APPDATA%\Blender Foundation\Blender\<versión>\extensions\user_default\mcp\` |
| Protocolo | Socket local `localhost:9876` |
| Auto-arranque | Preferencia `use_autostart=True` del add-on (arranca ~1 s tras abrir Blender) |
| Requisito | "Allow Online Access" activado (Preferences → System) |
| Conflicto conocido | Add-ons MCP de terceros (p. ej. "Blender MCP" de ahujasid) usan el MISMO puerto 9876 con protocolo incompatible → mantenerlos desactivados |
| ZIP del add-on oficial | `https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip` |

## Diagnóstico por síntoma

### "Cannot connect to Blender at localhost:9876" (falla rápida)

Blender está cerrado, recién abriendo, o el servidor no arrancó.

1. Abrir Blender (por computer use o manualmente), esperar ~25 s, reintentar.
2. Si Blender YA está abierto: el auto-arranque pudo fallar. En la consola Python de Blender
   (workspace Scripting) ejecutar:
   `bpy.ops.blmcp.server_start()`
   Si responde "Online access must be enabled": ejecutar antes
   `bpy.context.preferences.system.use_online_access = True; bpy.ops.wm.save_userpref()`
3. Verificar que el add-on esté habilitado:
   `bpy.ops.preferences.addon_enable(module='bl_ext.user_default.mcp'); bpy.ops.wm.save_userpref()`
4. Activar el auto-arranque para el futuro:
   `p = bpy.context.preferences.addons['bl_ext.user_default.mcp'].preferences; p.use_autostart = True; bpy.ops.wm.save_userpref()`

### Timeout de 60 s en cada llamada (cuelgue)

Causa típica: algo incompatible escucha en el 9876 (un add-on MCP de terceros) o el proceso del
conector quedó atascado en cola por intentos previos fallidos.

1. Confirmar que ningún add-on MCP de terceros esté activo con su servidor corriendo (panel lateral N
   del viewport → pestaña del add-on → detener/desactivar), y desactivarlo:
   `bpy.ops.preferences.addon_disable(module='<modulo_del_addon>'); bpy.ops.wm.save_userpref()`
2. Reiniciar Blender — el add-on oficial auto-arranca al abrir.
3. Si sigue igual: el proceso conector está atascado → desactivar y reactivar el conector "Blender"
   en los ajustes del app de escritorio de Claude, o cerrar y reabrir el app. Luego reintentar.

### Los permisos de computer use "desaparecen"

Si aparece "No applications are granted for this session", volver a ejecutar el par
`computer_resolve_access` → `computer_request_access` (el usuario aprueba de nuevo). Resolver SIEMPRE
tanto el lanzador (p. ej. `"Blender 5.1"`) como `"blender.exe"` (dueño real de la ventana; sin él las
capturas salen enmascaradas).

### La ventana de Blender no responde a clics de computer use

El primer clic tras un cambio de foco solo enfoca la ventana; repetir el clic. Verificar el resultado
con zoom antes de asumir que un botón se pulsó.

### Escribir en la consola Python de Blender

Workspace "Scripting" → clic en la línea `>>>` → comandos de UNA línea (usar `;` para encadenar).
Tras un fallo de foco, hacer clic en la consola y reintentar.

## Instalar / rehacer el setup desde cero (consola de Blender)

1. Descargar el ZIP oficial (URL en la tabla) — o directamente:
   `import urllib.request; urllib.request.urlretrieve('https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip', r'<ruta>\mcp-1.0.0.zip')`
2. `bpy.ops.extensions.package_install_files(filepath=r'<ruta>\mcp-1.0.0.zip', repo='user_default', enable_on_install=True)`
3. `bpy.context.preferences.system.use_online_access = True`
4. `p = bpy.context.preferences.addons['bl_ext.user_default.mcp'].preferences; p.use_autostart = True`
5. Desactivar add-ons MCP de terceros si existen: `bpy.ops.preferences.addon_disable(module='blender_mcp_addon')`
6. `bpy.ops.wm.save_userpref()` y reiniciar Blender.
