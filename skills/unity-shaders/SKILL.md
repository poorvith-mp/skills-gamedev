---
name: unity-shaders
group: Unity
description: >-
  Author materials and VFX with Shader Graph, HLSL and URP/HDRP custom passes. Use when authoring
  URP/HDRP shaders, Shader Graph, or visual effects.
---

# unity-shaders

## Core Philosophy
Real-time shaders are programs executed concurrently across thousands of GPU cores for every single vertex and pixel on screen. In modern Unity (URP and HDRP), authoring game visual effects requires mastering both node-based Shader Graph and text-based HLSL. Writing efficient shaders is an exercise in computational frugality: every trigonometric function, dependent texture read, or unnecessary full-precision float in the fragment shader directly burns GPU fill-rate and tanks mobile/console frame rates.

---

## 4-Step Unity Shader & VFX Pipeline

### Step 1: Pipeline Architecture: URP vs HDRP Custom Passes
1. **Universal Render Pipeline (URP)**:
   - Optimized for mobile, Nintendo Switch, VR, and PC.
   - Single-pass forward and deferred rendering paths.
   - Extensible via `ScriptableRendererFeature` and custom Render Passes.
2. **High Definition Render Pipeline (HDRP)**:
   - High-end PC and current-gen consoles (PS5, Xbox Series X).
   - Compute-shader-driven deferred clustered lighting, volumetric fog, and ray-tracing.
   - Extensible via `CustomPassVolume`.

### Step 2: HLSL Structure & Precision Optimization
1. **Precision Standard**:
   - `float` (32-bit): World-space coordinates, UV coordinates, depth calculations.
   - `half` (16-bit): Colors, normals, vector directions, ambient lighting calculations (saves 50% ALU register pressure on mobile GPUs).
2. **Core Shader Pass Layout**:
   - `Vertex Shader`: Transforms object-space vertices to clip space:
     $$P_{\text{clip}} = M_{\text{projection}} \times M_{\text{view}} \times M_{\text{model}} \times P_{\text{local}}$$
   - `Fragment / Pixel Shader`: Calculates final RGBA color per rasterized pixel.

### Step 3: Math & Procedural Effects (Dissolve, Rim Light, Fresnel)
1. **Schlick's Fresnel Approximation (Rim Lighting)**:
   $$F \approx F_0 + (1 - F_0)(1 - \vec{N} \cdot \vec{V})^5$$
   - Where $\vec{N}$ is surface normal and $\vec{V}$ is normalized view direction vector.
2. **Procedural Dissolve with Alpha Clipping**:
   - Sample a Simplex/Voronoi noise texture, subtract a `_Cutoff` property, and execute `clip(noise - cutoff)`.

### Step 4: URP Custom Render Pass Architecture
1. **Creating Custom Passes (`ScriptableRendererFeature`)**:
   - Inject custom rendering logic into the URP frame: e.g. custom outline passes, thermal vision, or full-screen blit effects:
     - Enqueue pass at `RenderPassEvent.AfterRenderingTransparents`.
     - Allocate temporary render textures via `RTHandles`.

---

## Deliverable Format: Optimized URP Unlit Dissolve Shader (HLSL)

```hlsl
Shader "Custom/URP_Dissolve"
{
    Properties
    {
        _MainTex ("Albedo Texture", 2D) = "white" {}
        _NoiseTex ("Noise Texture", 2D) = "white" {}
        _Cutoff ("Dissolve Cutoff", Range(0, 1)) = 0.0
        [HDR] _EdgeColor ("Edge Burn Color", Color) = (2.0, 0.5, 0.0, 1.0)
        _EdgeWidth ("Edge Width", Range(0, 0.2)) = 0.05
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" }
        LOD 100

        Pass
        {
            Name "ForwardLit"
            Tags { "LightMode"="UniversalForward" }

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct Attributes {
                float4 positionOS : POSITION;
                float2 uv         : TEXCOORD0;
            };

            struct Varyings {
                float4 positionCS : SV_POSITION;
                float2 uv         : TEXCOORD0;
            };

            TEXTURE2D(_MainTex);    SAMPLER(sampler_MainTex);
            TEXTURE2D(_NoiseTex);   SAMPLER(sampler_NoiseTex);

            CBUFFER_START(UnityPerMaterial)
                float4 _MainTex_ST;
                half4  _EdgeColor;
                half   _Cutoff;
                half   _EdgeWidth;
            CBUFFER_END

            Varyings vert(Attributes input) {
                Varyings output;
                output.positionCS = TransformObjectToHClip(input.positionOS.xyz);
                output.uv = TRANSFORM_TEX(input.uv, _MainTex);
                return output;
            }

            half4 frag(Varyings input) : SV_Target {
                half noise = SAMPLE_TEXTURE2D(_NoiseTex, sampler_NoiseTex, input.uv).r;
                clip(noise - _Cutoff); // Discard pixel if below dissolve threshold

                half4 albedo = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, input.uv);
                
                // Calculate glowing burn edge
                half edgeFactor = step(noise - _Cutoff, _EdgeWidth);
                half4 finalColor = lerp(albedo, _EdgeColor, edgeFactor);
                
                return finalColor;
            }
            ENDHLSL
        }
    }
}
```

---

## Worked Example: 60 FPS Mobile GPU Optimization for Anime Water Shader

- **Problem**: A stylized water shader caused the game to drop from 60 FPS to 24 FPS on iPhone 11 and Snapdragon mobile devices.
- **Diagnosis**: Profiling in RenderDoc showed 8 dependent texture lookups in the fragment shader and universal 32-bit `float` precision causing high register spilling.
- **Optimization**:
  1. Converted all non-coordinate variables from `float` to `half`.
  2. Pre-baked 3 procedural caustic noise octaves into a single RGB channel-packed lookup texture, replacing 3 runtime procedural noise calculations with 1 sample.
- **Outcome**: Frame time on target mobile GPU dropped from 41ms to 14.8ms, locking the game at a stable 60 FPS.

---

## Verification Checklist

- [ ] Properties wrapped inside `CBUFFER_START(UnityPerMaterial)` for SRP Batcher compatibility.
- [ ] Mobile shaders utilize `half` precision for colors, directions, and lighting math.
- [ ] Alpha test / `clip()` avoided on mobile tiled GPUs where Early-Z rejection is critical.
- [ ] Textures channel-packed (e.g. Roughness in R, Metallic in G, AO in B) to minimize texture samplers.
- [ ] Custom passes integrated via `ScriptableRendererFeature` in URP.

---

## Anti-Patterns

- **Breaking SRP Batcher**: Declaring shader uniforms outside of `CBUFFER_START(UnityPerMaterial)`, forcing Unity to issue unique draw calls per material.
- **Over-Using Complex Math in Fragment Pass**: Running sin/cos/pow loops per pixel that could be calculated once in vertex shader and interpolated.
- **Neglecting Depth Texture Performance**: Indiscriminately enabling `_CameraDepthTexture` on mobile hardware without verifying bandwidth overhead.
