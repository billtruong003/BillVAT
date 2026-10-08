# Bill VAT — Vertex Animation Textures

Play skinned animations on a plain `MeshRenderer`. The baker samples every clip of a character's
`Animator` into a position texture; a shader reads it on the GPU. No `Animator`, no skinning, and
renderers batch with GPU instancing, so hundreds of animated enemies cost little more than static
meshes.

## Install

Package Manager → **Add package from git URL…**

```text
https://github.com/billtruong003/BillVAT.git#v1.0.0
```

Requires Unity 6000.0+ and URP 17.

## Bake

1. Open **Tools ▸ BillTheDev ▸ VAT Baker (Batch)**.
2. Drag prefabs or scene objects into the window. Each needs a `SkinnedMeshRenderer` and an
   `Animator` with a controller; every non-legacy clip in that controller is baked.
3. Press **Bake All Selected Animations** and choose an output folder.

For each source you get:

| Asset | |
|---|---|
| baked `Mesh` | the skinned mesh in bind pose, with vertex IDs for the texture lookup |
| position texture | one row per frame, positions packed between the bounds below |
| `VAT_AnimationData` | mesh, texture, `positionMinBounds`/`positionMaxBounds`, clip list (name, start frame, frame count, duration, wrap mode) |
| material | `BillTheDev/VAT/Optimized_VAT` with the texture and bounds already set |

Automation: `VAT_BakerEditorWindow.BakeObjects(sources, "Assets/Art/VAT")` bakes without the window
and returns how many succeeded.

## Play

Put the baked mesh on a `MeshFilter` + `MeshRenderer` with the baked material, then add
`VAT_Animator`:

```csharp
var vat = enemy.GetComponent<VAT_Animator>();
vat.Play("Run");
vat.CrossFade("Attack", 0.15f);
vat.Play("Idle", normalizedStartPhase: Random.value);   // de-sync a crowd
string clip = vat.CurrentClipName;
float t = vat.CurrentNormalizedPhase;
```

| Field | |
|---|---|
| Animation Data | the baked `VAT_AnimationData` |
| Playback Speed | multiplier |
| Default Clip Index | played on start |
| Show Animation Bounds | gizmo |

Both animators set per-renderer values through a `MaterialPropertyBlock`, so renderers sharing one
material still batch. `VAT_Animator_Instanced` is a lighter variant with `Play(clipName)` only.

`VAT_Animator` checks, and never writes, the material: if the material's position texture or bounds
do not match the `VAT_AnimationData`, it logs an error naming the mismatch. Re-bake, or copy the
values from the data asset onto the material.

## Shaders

| Shader | Use |
|---|---|
| `BillTheDev/VAT/Optimized_VAT` | unlit albedo, cheapest (CG, not SRP-Batcher compatible) |
| `BillTheDev/VAT/URP_ToonLit_VAT` | toon-lit with shadows |
| `BillTheDev/VAT/Ghost_VAT` | transparent ghost look |

To write your own, sample `_PositionTexture` with the vertex ID and the clip's frame, and remap with
`_PositionMin`/`_PositionMax`; `VAT_Toonlit.shader` is the reference.

## Limits

- Positions only: normals stay in bind pose, so lighting does not follow the deformation exactly.
  Toon shading hides most of it.
- Texture height = total frames of all clips; very long clips on high-vertex meshes make large
  textures. Bake at 30 fps (the default) and keep crowd meshes light.
- No blending between more than two clips; no root motion.

## License

MIT, see [LICENSE.md](LICENSE.md).
