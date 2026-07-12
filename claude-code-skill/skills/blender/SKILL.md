---
name: blender
description: Controla Blender en esta computadora para crear y editar escenas 3D a partir de prompts o imágenes de referencia. Usar SIEMPRE que el usuario mencione Blender o pida trabajo 3D, por ejemplo "hazlo en blender", "crea en blender", "ábreme blender", "modela X en 3D", "haz un render de". Abre Blender automáticamente si está cerrado, conecta por MCP (puerto local 9876) y ejecuta código bpy verificando el resultado visualmente.
---

# Blender: control total desde Claude Code

Requiere: add-on oficial "MCP" (Blender Lab) instalado en Blender ≥ 5.1 con auto-arranque en
`localhost:9876`, y el servidor MCP oficial de Blender registrado en Claude Code (ver el README del
proyecto: https://github.com/tormentionrex-hub/claude-blender).

## Paso 1 — Probar la conexión

Llamar la herramienta `get_objects_summary` del servidor MCP de Blender (el prefijo depende del
nombre con que se registró el servidor, p. ej. `mcp__blender__get_objects_summary`).

- **Responde con la escena** → listo, ir al Paso 3.
- **Error "Cannot connect to Blender at localhost:9876"** → Blender está cerrado. Ir al Paso 2.
- **Timeout persistente** → ver `references/solucion-problemas.md`.

## Paso 2 — Abrir Blender automáticamente

En Windows (ajustar la ruta a la versión instalada):

```bash
powershell -c "Start-Process 'C:\Program Files\Blender Foundation\Blender 5.1\blender-launcher.exe'"
```

En macOS: `open -a Blender` · En Linux: `blender &`

Esperar ~25 segundos (el add-on auto-arranca el servidor ~1 s tras cargar) y reintentar el Paso 1
hasta 3 veces. Para verificar el puerto sin MCP (Windows):

```bash
powershell -c "(Test-NetConnection localhost -Port 9876).TcpTestSucceeded"
```

## Paso 3 — Trabajar: crear/editar con bpy

1. Planear la escena a partir del prompt o imagen del usuario.
2. Ejecutar `execute_blender_code` con código Python (`bpy`): asignar un dict a `result`, código
   idempotente, bloques medianos. Recetas probadas en `references/recetas-bpy.md`.
3. Verificar SIEMPRE de forma visual: `get_screenshot_of_window_as_image` (poner antes el viewport en
   RENDERED, receta incluida) o `render_thumbnail_to_path`.
4. Iterar hasta cumplir lo pedido y guardar con `bpy.ops.wm.save_as_mainfile(filepath=...)` donde el
   usuario indique.

Para procesar archivos .blend sin abrir la interfaz usar las variantes `*_for_cli`
(`blender --background`).

## Reglas

- NUNCA cerrar Blender con `quit_blender` directo en una llamada MCP; usar el patrón de timer de
  `references/recetas-bpy.md`.
- NO activar add-ons MCP de terceros en paralelo (chocan con el oficial en el puerto 9876).
- NO desactivar "Allow Online Access" en las preferencias de Blender.
