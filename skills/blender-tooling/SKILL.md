---
name: blender-tooling
group: 3D assets
description: >-
  Build Blender Python add-ons, asset validators, exporters and pipeline automation. Use when
  writing Python scripts, add-ons, or batch tools for Blender.
---

# blender-tooling

## Core Philosophy
Manual 3D asset preparation—clicking through 14 menus to apply transforms, rename 40 bones, fix inverted normals, and export FBX files one by one—is an enormous drain on game studio productivity. High-performance art pipelines rely on custom Python automation within Blender (`bpy`). Professional Blender tooling builds robust, headless CLI batch exporters, automated asset validation linters, custom UI panels, and asset post-processors that eliminate human error.

---

## 4-Step Blender Python Tooling Architecture

### Step 1: The Blender Python API Anatomy (`bpy`)
1. **Core Submodules**:
   - `bpy.context`: Active selection, active mode, active scene.
   - `bpy.data`: Datablocks (meshes, armatures, materials, textures, actions).
   - `bpy.ops`: Operator execution (e.g. `bpy.ops.object.transform_apply()`).
   - `bpy.types`: Base classes for creating custom Operators, Panels, and Property Groups.
2. **The Safe Operator Rule**:
   - Avoid calling `bpy.ops` inside heavy loops (it is slow and requires active 3D context). Mutate `bpy.data` and mesh attributes directly via Python data APIs where possible.

### Step 2: Automated Asset Validation Linters
1. **Pre-Export Linter Checks**:
   - Write automated scripts that assert:
     - Unapplied Scale: `obj.scale != Vector((1.0, 1.0, 1.0))`.
     - Ngon Count: `any(len(poly.vertices) > 4 for poly in mesh.polygons)`.
     - Missing Materials / Textures: Materials with missing image filepaths.
     - Origin Alignment: Origin not at `(0, 0, 0)` for static props.
2. **The Hard Blocker Gate**:
   - If a validation check fails, abort export and display an error modal listing exact offender object names.

### Step 3: Custom Add-On Development (`bl_info` Standard)
1. **Add-On Architecture**:
   - Structure add-ons with formal registration:
     ```python
     bl_info = {
         "name": "Studio Pipeline Exporter",
         "author": "Technical Art Team",
         "version": (1, 2, 0),
         "blender": (4, 1, 0),
         "category": "Pipeline",
     }
     ```
2. **Custom UI Panels in the 3D Viewport**:
   - Create a dedicated sidebar tab (`N-Panel`) exposing 1-click batch actions: "Sanitize Asset", "Bake Normal Map", "Export Game-Ready FBX".

### Step 4: Headless CLI Batch Processing
1. **Headless Execution**:
   - Run Blender without GUI in CI/CD or build servers to process hundreds of assets overnight:
     ```bash
     blender -b assets/character.blend -P scripts/batch_export.py -- --output-dir ./dist/fbx
     ```
2. **CLI Argument Parsing**:
   - Use `sys.argv[sys.argv.index("--") + 1:]` to parse custom command-line arguments passed after Blender's native flags.

---

## Deliverable Format: Blender Python Add-on Template (`studio_exporter.py`)

```python
# -*- coding: utf-8 -*-
bl_info = {
    "name": "Game Asset Validator & Exporter",
    "author": "Tech Art",
    "version": (1, 0, 0),
    "blender": (4, 0, 0),
    "location": "View3D > Sidebar > Game Pipeline",
    "category": "Pipeline",
}

import bpy
from mathutils import Vector

class OBJECT_OT_validate_game_asset(bpy.types.Operator):
    '''Validate active object for game-engine readiness'''
    bl_idname = "object.validate_game_asset"
    bl_label = "Validate Active Asset"

    def execute(self, context):
        obj = context.active_object
        if not obj or obj.type != 'MESH':
            self.report({'ERROR'}, "No mesh object selected!")
            return {'CANCELLED'}

        # 1. Check Unapplied Scale
        if obj.scale != Vector((1.0, 1.0, 1.0)):
            self.report({'ERROR'}, f"Unapplied scale: {obj.scale}. Press Ctrl+A to apply.")
            return {'CANCELLED'}

        # 2. Check for Ngons (> 4 vertices)
        mesh = obj.data
        ngon_count = sum(1 for poly in mesh.polygons if len(poly.vertices) > 4)
        if ngon_count > 0:
            self.report({'ERROR'}, f"Asset contains {ngon_count} ngons! Triangulate or retopologize.")
            return {'CANCELLED'}

        self.report({'INFO'}, "Asset passed all game-ready checks!")
        return {'FINISHED'}

class VIEW3D_PT_game_pipeline_panel(bpy.types.Panel):
    '''Custom sidebar panel in 3D Viewport'''
    bl_label = "Game Pipeline Tools"
    bl_idname = "VIEW3D_PT_game_pipeline_panel"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = "Game Pipeline"

    def draw(self, context):
        layout = self.layout
        layout.operator("object.validate_game_asset", icon='CHECKMARK')

def register():
    bpy.utils.register_class(OBJECT_OT_validate_game_asset)
    bpy.utils.register_class(VIEW3D_PT_game_pipeline_panel)

def unregister():
    bpy.utils.unregister_class(OBJECT_OT_validate_game_asset)
    bpy.utils.unregister_class(VIEW3D_PT_game_pipeline_panel)

if __name__ == "__main__":
    register()
```

---

## Worked Example: Automated 500-Prop Batch FBX Exporter

- **Challenge**: Artists spent 2 days manually opening 500 individual `.blend` files to re-export FBX files after an engine coordinate axis change.
- **Solution**: Wrote a 60-line headless Python script that opened Blender in background mode, sanitized scales, aligned origins, and exported FBX assets in 12 minutes.
- **Outcome**: Saved 16 hours of artist labor; eliminated 100% of human export coordinate mistakes.

---

## Verification Checklist

- [ ] Add-on defines compliant `bl_info` dictionary.
- [ ] Operators use `register()` and `unregister()` lifecycle hooks cleanly.
- [ ] Validation functions check for unapplied transforms, ngons, and missing textures.
- [ ] Scripts can execute headlessly in background mode (`blender -b -P script.py`).
- [ ] Custom script arguments parsed safely after the `--` delimiter.

---

## Anti-Patterns

- **Hardcoding Absolute Filepaths**: Hardcoding `C:/Users/artist/Desktop` inside Blender Python tools.
- **Ignoring Context**: Invoking `bpy.ops` commands that require active 3D Viewport selections inside background headless runs.
- **Destructive Batch Operations**: Overwriting original `.blend` source files without automated backup checkpoints.
