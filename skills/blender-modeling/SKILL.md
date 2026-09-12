---
name: blender-modeling
last_reviewed: 2026-09-06
group: 3D assets
description: >-
  Model, retopologise, UV unwrap and bake game-ready assets with correct scale, pivots and
  polycount. Use when creating 3D hard-surface meshes, low-poly props, or UV layouts.
---

# blender-modeling

## Core Philosophy
3D modeling for real-time video games is not creating unconstrained digital sculptures. A high-poly model with 4,000,000 polygons and 12 un-optimized material slots will crash a game engine. Real-time 3D art is the rigorous craft of geometric efficiency: sculpting high-fidelity details, retopologizing into clean low-poly game meshes, baking high-to-low normal maps with zero shading artifacts, establishing consistent texel density, and enforcing strict transform hygiene.

---

## 4-Step Game-Ready 3D Modeling Pipeline

### Step 1: High-Poly to Low-Poly Modeling & Retopology
1. **High-Poly Sculpting / Hard-Surface**:
   - Model complete micro-details: bevels, screws, seams, welded joints, cloth folds.
2. **Low-Poly Retopology Discipline**:
   - Silhouette is King: Allocate polygon budget to the **outer silhouette** where curved edges meet the sky/background. Flat surfaces require minimal geometry.
   - Clean Edge Loops: Quad-dominant topology along deformation joints; triangulate flat surfaces before export to prevent engine-specific triangulation warping.
   - Polycount Budgets:
     - Hero Character: 30k–60k triangles.
     - First-Person Weapon: 15k–30k triangles.
     - Environment Prop: 1k–5k triangles.

### Step 2: UV Unwrapping & Texel Density Standardization
1. **The Texel Density Constant**:
   - Standardize pixel density per meter across all assets in the game world:
     - Hero First-Person Props: **2048 px/m** (4K texture for 2m object).
     - Standard Characters / Weapons: **1024 px/m** (2K texture for 2m object).
     - Environment Props / Architecture: **512 px/m**.
2. **UV Seam Placement & Packing Hygiene**:
   - Place UV seams in concealed areas (undersides, inner seams, sharp mechanical angles).
   - Packing Efficiency: Achieve **$\ge 75\%$ UV space utilization**. Pack UV islands with at least **16px padding** (at 2K resolution) to prevent mipmap texture bleeding.

### Step 3: Normal Map Baking & Shading Seam Prevention
1. **The 3 Golden Rules of Baking**:
   - *Rule 1: Hard Edges Require UV Seams*: Any sharp edge ($> 60^\circ$ angle) marked with a Hard/Sharp Edge **must** have a corresponding UV seam split to prevent dark shading gradients across normal maps.
   - *Rule 2: Triangulate Before Baking*: Triangulate the low-poly mesh prior to baking to guarantee the normal map is baked against the exact triangulation the engine renders.
   - *Rule 3: Use Average Normals & Cage*: Use an expanded cage mesh to prevent projection ray intersection clipping on tight mechanical crevices.

### Step 4: Transform Hygiene & Export Configuration
1. **Transform Normalization**:
   - Apply all transforms: Location `(0,0,0)`, Rotation `(0,0,0)`, Scale `(1.0, 1.0, 1.0)`.
   - Set Origin Point: Place the origin pivot at the logical physical base (e.g. bottom-center of props for easy ground snapping, or center-of-mass for physics debris).

---

## Deliverable Format: 3D Asset Specification (`ASSET-SPEC.md`)

```markdown
# Real-Time 3D Asset Specification: [Prop Name]

## 1. Geometric Budget & Dimensions
- **Asset Name**: Tactical First-Aid Station Prop
- **Physical Dimensions**: 0.6m (W) x 0.8m (H) x 0.25m (D)
- **Target Polygon Count**: 3,200 triangles (Budget: <= 4,000 triangles)
- **Topology**: 100% clean manifold geometry; zero ngons; zero loose vertices.

## 2. UV & Texel Density Standards
- **Texture Set Size**: 2048x2048px (PBR Metallic/Roughness workflow)
- **Target Texel Density**: 1024 px/meter
- **UV Island Padding**: 16px minimum gutter spacing
- **UV Space Efficiency**: 79.4% packed area

## 3. Texture Map Manifest
| Map Type | Channel Packing | Format | Bit Depth |
|---|---|---|---|
| Base Color / Albedo | RGB | PNG / TGA | 8-bit sRGB |
| Normal Map | RGB (OpenGL / DirectX) | PNG / TGA | 16-bit Linear |
| ORM Map | R: Occlusion, G: Roughness, B: Metallic | PNG / TGA | 8-bit Linear |

## 4. Transform & Pivot Setup
- Pivot Point: Bottom-center at `(0, 0, 0)`.
- Applied Scale: `(1.0, 1.0, 1.0)` confirmed.
```

---

## Worked Example: Eliminating Black Normal Map Seams

- **Problem**: Hard-surface weapon model exhibited ugly dark shading gradients along mechanical edges when imported into Unreal Engine.
- **Diagnosis**: 90-degree bevel edges were marked as smooth without UV seams, forcing the normal map to compensate for extreme vertex normal bending.
- **Fix**: Marked 90-degree edges as Sharp; split UV islands along those exact edges; re-baked with cage.
- **Result**: Shading rendered completely seamless and flat under real-time lighting.

---

## Verification Checklist

- [ ] All transforms applied (Scale is strictly 1.0; rotation is 0,0,0).
- [ ] Pivot origin positioned at logical attachment/ground point.
- [ ] No ngons (faces with $> 4$ vertices) or non-manifold geometry present.
- [ ] Texel density verified consistent with project standards.
- [ ] Hard/sharp edges correspond 1:1 with UV seam boundaries.

---

## Anti-Patterns

- **Leaving Unapplied Scales**: Exporting an asset scaled at 0.05, breaking real-time physics and distance culling.
- **Wasted UV Texture Space**: Packing UVs with 40% empty white space, wasting texture VRAM.
- **Baking Without UV Padding**: Setting 0px gutter margin between UV islands, causing colors to bleed across LOD mipmaps.
