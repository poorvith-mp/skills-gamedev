---
name: unity-tooling
group: Unity
description: >-
  Build editor tooling: EditorWindows, PropertyDrawers, AssetPostprocessors and ScriptedImporters.
  Use when writing custom Unity editor windows, inspectors, or gizmos.
---

# unity-tooling

## Core Philosophy
Game development speed is bottlenecked by editor friction. When level designers, game writers, and technical artists have to manually navigate deep inspector hierarchies, input coordinate numbers by hand, or edit JSON files in Notepad, development slows to a crawl and errors multiply. Professional Unity tooling extends the Unity Editor using custom `EditorWindow`, `PropertyDrawer`, `AssetPostprocessor`, and `ScriptedImporter` classes to build bespoke internal tools that make design workflows foolproof and instant.

---

## 4-Step Unity Editor Tooling Architecture

### Step 1: Custom PropertyDrawers & Decorators
1. **Property Drawers**:
   - Customize how specific data structures render in the default Inspector.
   - Use `EditorGUI` and `EditorGUILayout` (or modern UI Toolkit `PropertyField`).
2. **Validation Decorators**:
   - Create custom attributes like `[ShowIf("hasWeapon")]` or `[RequiredReference]` that visually warn designers with red error boxes if an asset slot is left empty.

### Step 2: Custom EditorWindows (UI Toolkit vs IMGUI)
1. **Modern UI Toolkit Standard**:
   - Build dockable editor windows using UXML and USS stylesheets:
     - Native flexbox styling, reactive data binding, and high performance.
2. **Dockable Editor Utilities**:
   - Level distribution tools, dialogue node graph editors, audio sound bank auditioners, and localization key managers.

### Step 3: Automated Asset Postprocessors (`AssetPostprocessor`)
1. **Enforcing Art Pipeline Standards**:
   - Automatically configure imported textures, models, and audio clips without human intervention:
     - Set all UI sprites to Sprite (2D and UI) with Power-of-Two compression.
     - Strip animation tracks and blendshapes from rigid static props.
     - Clamp 4K texture imports to 2048px maximum size.
     - Convert audio imports to Vorbis/ADPCM based on file duration.

### Step 4: Custom Asset Importers (`ScriptedImporter`)
1. **Bespoke File Format Parsing**:
   - Create native importers for custom studio file extensions (e.g. `.dialogue`, `.voxel`, `.yarn`):
     - Maps raw external text or binary files directly into native Unity assets (ScriptableObjects or Prefabs) upon dragging into the Project view.

---

## Deliverable Format: Production Automated Art Asset Postprocessor

```csharp
#if UNITY_EDITOR
using UnityEditor;
using UnityEngine;

public class TextureAndModelPostprocessor : AssetPostprocessor {
    // Automatically sanitize texture imports
    private void OnPreprocessTexture() {
        TextureImporter importer = (TextureImporter)assetImporter;

        // Auto-configure UI icons
        if (assetPath.Contains("Assets/Art/UI/")) {
            importer.textureType = TextureImporterType.Sprite;
            importer.mipmapEnabled = false;
        }

        // Auto-configure 3D model textures
        if (assetPath.Contains("Assets/Art/Textures/")) {
            importer.maxTextureSize = 2048;
            TextureImporterPlatformSettings settings = importer.GetDefaultPlatformTextureSettings();
            settings.format = TextureImporterFormat.Automatic;
            settings.textureCompression = TextureImporterCompression.CompressedHQ;
            importer.SetPlatformTextureSettings(settings);
        }
    }

    // Automatically sanitize 3D FBX imports
    private void OnPreprocessModel() {
        ModelImporter importer = (ModelImporter)assetImporter;

        // Strip animations from environment props
        if (assetPath.Contains("Assets/Art/Environment/")) {
            importer.importAnimation = false;
            importer.materialImportMode = ModelImporterMaterialImportMode.None;
            importer.isReadable = false; // Free CPU memory
        }
    }
}
#endif
```

---

## Worked Example: Custom Level Palette Painter Window

- **Problem**: Level designers spent 4 hours per room manually dragging prefab assets from the Project folder, placing them, and zeroing out their rotation coordinates.
- **Solution**: Built an `EditorWindow` called `PaletteSpawnerWindow`. Designers press a hotkey, select a category ("Props - Furniture"), and click directly on the terrain geometry in the Scene view to spawn randomly rotated props snapped to the surface normal.
- **Outcome**: Level dressing time per room decreased from 4 hours to 35 minutes.

---

## Verification Checklist

- [ ] All editor-specific scripts placed inside an `Editor/` folder or wrapped in `#if UNITY_EDITOR` to prevent build compilation failures.
- [ ] `AssetPostprocessor` rules enforce compression, mipmapping, and size limits automatically.
- [ ] Custom `PropertyDrawer` scripts implement proper `GetPropertyHeight()` calculations.
- [ ] Heavy editor processes use `EditorUtility.DisplayProgressBar()` and `EditorUtility.ClearProgressBar()`.
- [ ] All editor-modified scene objects properly register undo actions via `Undo.RecordObject()`.

---

## Anti-Patterns

- **Leaking Editor Code into Standalone Builds**: Forgetting `#if UNITY_EDITOR` on scripts importing `UnityEditor`, causing the entire production CI/CD build to fail.
- **Modifying Scene State Without `Undo`**: Mutating GameObjects programmatically without `Undo.RecordObject()`, preventing designers from using Ctrl+Z.
- **Heavy Asset Scans in `OnGUI()`**: Executing file I/O or `AssetDatabase.FindAssets()` inside high-frequency `OnGUI()` loops, freezing the editor.
