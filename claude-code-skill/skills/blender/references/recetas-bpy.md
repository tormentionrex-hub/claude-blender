# Recetas bpy probadas (Blender 5.1)

Fragmentos verificados en la máquina del usuario. Ejecutar vía `execute_blender_code`; devolver datos asignando un dict a `result`.

## Limpiar la escena por defecto

```python
import bpy
for name in ("Cube",):
    if name in bpy.data.objects:
        bpy.data.objects.remove(bpy.data.objects[name], do_unlink=True)
result = {"objetos": [o.name for o in bpy.data.objects]}
```

## Material metálico / de color (Principled BSDF)

```python
import bpy
m = bpy.data.materials.new("Oro")
m.use_nodes = True
n = m.node_tree.nodes["Principled BSDF"]
n.inputs["Base Color"].default_value = (1.0, 0.68, 0.16, 1)
n.inputs["Metallic"].default_value = 1.0
n.inputs["Roughness"].default_value = 0.16
obj = bpy.data.objects["MiObjeto"]
obj.data.materials.append(m)
```

## Suavizar malla sin operadores de contexto

```python
for p in obj.data.polygons:
    p.use_smooth = True
sub = obj.modifiers.new("Subd", "SUBSURF")
sub.levels = 2
sub.render_levels = 3
```

## Luces de estudio (key + rim)

```python
import bpy
def lamp(name, kind, loc, energy, color=(1, 1, 1), size=None):
    d = bpy.data.lights.new(name, kind)
    d.energy = energy
    d.color = color
    if size and kind == 'AREA':
        d.size = size
    o = bpy.data.objects.new(name, d)
    bpy.context.collection.objects.link(o)
    o.location = loc
    return o

lamp("Key", "AREA", (4, -4, 6), 900, (1, 0.95, 0.88), 5)
lamp("Rim", "AREA", (-5, 4, 4), 500, (0.25, 0.5, 1.0), 4)
```

## Cámara mirando a un objeto

```python
cam = bpy.data.objects["Camera"]
cam.location = (6.2, -6.2, 3.4)
bpy.context.scene.camera = cam
c = cam.constraints.new('TRACK_TO')
c.target = bpy.data.objects["MiObjeto"]
c.track_axis = 'TRACK_NEGATIVE_Z'
c.up_axis = 'UP_Y'
```

## Poner el viewport en modo RENDERED y vista de cámara

(Hacer esto antes de `get_screenshot_of_window_as_image` para que la captura muestre el resultado real.)

```python
try:
    for win in bpy.context.window_manager.windows:
        for a in win.screen.areas:
            if a.type == 'VIEW_3D':
                a.spaces[0].shading.type = 'RENDERED'
                a.spaces[0].region_3d.view_perspective = 'CAMERA'
except Exception:
    pass
```

## Fondo del mundo

```python
w = bpy.context.scene.world
w.use_nodes = True
w.node_tree.nodes["Background"].inputs[0].default_value = (0.008, 0.008, 0.014, 1)
```

## Guardar el archivo

```python
import bpy
bpy.ops.wm.save_as_mainfile(filepath=r'C:\Users\<tu-usuario>\Documents\mi_escena.blend')
result = {"saved": bpy.data.filepath}
```

## Cerrar Blender de forma SEGURA (nunca directo)

El cierre directo dentro de una llamada MCP corta la conexión a mitad de respuesta. Diferirlo con un timer para que la respuesta regrese primero:

```python
import bpy
bpy.app.timers.register(lambda: bpy.ops.wm.quit_blender() and None, first_interval=1.5)
result = {"quitting": True}
```

## Notas de renderizado

- `render_thumbnail_to_path`: render chico y rápido; el add-on lo escribe en su carpeta temporal y devuelve la ruta real en la respuesta.
- `render_viewport_to_path`: usa la configuración actual de la escena (más lento, calidad real).
- Motor por defecto en 5.1: EEVEE. Para fotorealismo pedir explícitamente Cycles: `bpy.context.scene.render.engine = 'CYCLES'` (más lento).
