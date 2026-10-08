# Changelog

## [1.0.0] - 2026-10-08

First release as a standalone UPM package.

- Layout: `Runtime/` (`VAT.Runtime`), `Editor/` (`VAT.Editor`), `Shaders/`.
- `VAT_Animator` validates the material against its `VAT_AnimationData` and logs a clear error on a
  mismatch instead of silently patching the shared material at runtime.
- Error messages in English; removed references to one game's menus.
- Removed `BuildSizeOptimizer` and `TextureOptimizer` (general editor tools, not part of VAT).
