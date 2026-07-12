---
name: blender
description: Controla Blender en la computadora del usuario para crear y editar escenas 3D a partir de prompts o imágenes de referencia. Usar SIEMPRE que el usuario mencione Blender o pida trabajo 3D, por ejemplo "hazlo en blender", "crea en blender", "ábreme blender", "modela X en 3D", "haz un render de", "do it in Blender", "3D model of". Abre Blender automáticamente si está cerrado, conecta por MCP y ejecuta código bpy verificando el resultado visualmente.
---

# Blender: control total desde Claude

Flujo para crear o editar cualquier cosa en el Blender del usuario. Requiere el setup descrito en el
README del proyecto (conector oficial "Blender" del app de escritorio de Claude + add-on oficial
"MCP" de Blender Lab con auto-arranque en `localhost:9876`). Si el setup ya existe, NO reinstalar
nada: solo conectar, y si Blender está cerrado, abrirlo.

## Paso 1 — Cargar herramientas

Cargar en UNA sola llamada de ToolSearch las herramientas necesarias:

```
select:mcp__remote-devices__Blender__get_objects_summary,mcp__remote-devices__Blender__execute_blender_code,mcp__remote-devices__Blender__get_screenshot_of_window_as_image,mcp__remote-devices__Blender__render_thumbnail_to_path,mcp__remote-devices__Blender__render_viewport_to_path,mcp__remote-devices__Blender__get_object_detail_summary,mcp__remote-devices__Blender__search_api_docs,mcp__remote-devices__computer_resolve_access,mcp__remote-devices__computer_request_access,mcp__remote-devices__computer_open_application,mcp__remote-devices__computer_wait
```

(Si los nombres difieren en la sesión, buscar "Blender" con ToolSearch y usar los equivalentes.)

## Paso 2 — Probar la conexión

Llamar `get_objects_summary`.

- **Responde con la escena** → Blender está listo. Ir al Paso 4.
- **Error "Cannot connect to Blender at localhost:9876"** → Blender está cerrado (o recién abriendo). Ir al Paso 3.
- **Timeout de 60s** → ver `references/solucion-problemas.md`.

## Paso 3 — Abrir Blender automáticamente

1. `computer_resolve_access` con `["Blender"]`; si devuelve una sugerencia tipo `"Blender 5.1"` en
   `didYouMean`, volver a resolver con ese nombre exacto. Pasar el parámetro `device` con el nombre
   del dispositivo conectado del usuario (verlo con `get_device_info`).
2. `computer_request_access` con las entradas devueltas VERBATIM (el usuario aprueba una vez por
   sesión). Si al capturar pantalla la ventana sale enmascarada, resolver y pedir acceso también para
   `"blender.exe"` (el proceso dueño de la ventana).
3. `computer_open_application` con el nombre resuelto (p. ej. `"Blender 5.1"`).
4. `computer_wait` de 25 segundos (Blender carga y el add-on auto-arranca el servidor ~1 s después).
5. Reintentar `get_objects_summary`. Si falla, esperar 10 s más y reintentar (hasta 3 veces) — luego
   ver `references/solucion-problemas.md`.

## Paso 4 — Trabajar: crear/editar con bpy

Ciclo de trabajo:

1. **Planear** la escena a partir del prompt o imagen del usuario (si hay imagen de referencia,
   analizar formas, materiales, colores e iluminación y recrearlos con geometría, materiales
   Principled BSDF y luces).
2. **Ejecutar** `execute_blender_code` con código Python (`bpy`). Reglas:
   - Asignar un dict JSON-serializable a la variable `result` para devolver datos.
   - Escribir código idempotente (verificar `if "X" in bpy.data.objects` antes de crear/borrar).
   - Trabajar en bloques medianos (geometría → materiales → luces → cámara), no todo en una llamada gigante.
   - Suavizar mallas con `for p in obj.data.polygons: p.use_smooth = True` (evitar ops dependientes de contexto cuando sea posible).
   - Ante dudas de API, usar `search_api_docs` / `get_python_api_docs`.
3. **Verificar visualmente** SIEMPRE después de cambios significativos:
   - `get_screenshot_of_window_as_image` para ver el viewport (ponerlo antes en modo RENDERED y vista de cámara por código — ver `references/recetas-bpy.md`).
   - `render_thumbnail_to_path` para un render rápido de prueba.
   - `render_viewport_to_path` para el render final con la configuración de la escena.
4. **Iterar** hasta que coincida con lo pedido. Mostrar avances al usuario.
5. **Guardar** cuando el usuario lo pida o al terminar algo valioso:
   `bpy.ops.wm.save_as_mainfile(filepath=...)` en la carpeta que el usuario indique (p. ej. Documentos).

## Reglas importantes

- NUNCA cerrar Blender con `bpy.ops.wm.quit_blender()` directo dentro de una llamada MCP (mata la
  conexión a mitad de respuesta y puede atascar el conector). Si hay que cerrarlo, usar el patrón de
  timer diferido de `references/recetas-bpy.md`.
- NO activar add-ons MCP de terceros en paralelo (p. ej. el "Blender MCP" clásico de ahujasid):
  pelean por el puerto 9876 con protocolos incompatibles.
- NO desactivar "Allow Online Access" en las preferencias de Blender: el add-on oficial lo requiere.
- Para trabajar sobre archivos .blend SIN abrir la interfaz (procesos por lotes), existen las
  variantes `*_for_cli` (p. ej. `execute_blender_code_for_cli`) que corren `blender --background`.
- Comunicar rutas de archivos creados al usuario en lenguaje simple ("lo guardé en Documentos como X.blend").

## Referencias

- `references/recetas-bpy.md` — fragmentos bpy probados: escena base, materiales, luces, cámara, viewport en RENDERED, guardado, cierre seguro.
- `references/solucion-problemas.md` — diagnóstico de timeouts, puerto ocupado, reinicio del conector y reinstalación desde cero.
