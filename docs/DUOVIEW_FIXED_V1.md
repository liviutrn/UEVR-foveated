# DuoView Fixed-Center Mono — Sequential proof of concept v1

Source branch: `experiments/duoview-fixed-sequential-v1`. Baseline remains `baseline/remleo-01143-r6`.
UEVR upstream commit: `4dd185f94b72f8cd385000d4ed06d2648155ffda`.
UESDK: `remleo/UESDK@b1c6534652bd7381ad16e25f0507e3162e4b4691`.

## Rendering architecture

Select `DuoView Fixed Mono (Sequential DX12+OpenXR)` in UEVR > Unreal > Rendering Method.

- Scheduler is the existing synchronized-sequential two-draw pipeline, **not** native stereo.
- First UE view: **one monoscopic camera**, central 50% of FOV rendered at `0.5W × 0.5H` pixels. Projection's focal terms and off-axis offsets are scaled by 2.
- Second UE view: **same monoscopic camera**, full FOV rendered at `0.3W × 0.3H` pixels.
- GPU copies the two render-target regions to dedicated persistent D3D12 textures.
- GPU scales the peripheral region into the full per-eye image and draws the centre fovea, **opaque**, into the centred 50% rectangle.
- Same composed mono texture is copied to both OpenXR AFR eye swapchains **only after the second sequential view**. No eye gaze, no temporal reprojection, no external hooks.
- Unreal game world still uses two scene views, not four. This is monoscopic DuoView and naturally has no stereo depth perception.

Illustrative pixel count for W=H=3000: fovea 1500×1500 (2.25 MP), periphery 900×900 (0.81 MP); **3.06 MP total scene viewport area** vs 18 MP for full 3000×3000 stereo, excluding compositor/engine overhead and the extra GPU samples for filters. Viewport area is not guaranteed to equal actual shader work under every Unreal pipeline.

## Deliberate capability limits and safeguards

- **DX12 + OpenXR only**. DX11 or OpenVR cannot silently produce a fake DuoView image. An unsupported runtime blocks VR image submission and logs a reason.
- Engine render target must be non-multisampled BGRA8, 2D, even-width, and large enough; incompatible layouts are explicitly rejected before copy operations.
- First and second captures must be consecutive render frames, otherwise the pair is rejected. No silent normal-stereo fallback.
- Both outputs use one eye's head-centred pose, with no IPD offset and no per-eye cant; out-of-range periphery or fovea resources are recreated together when dimensions change.
- Depth-layer compositing is disabled in this mode: stereo depth is invalid for a two-FOV mono image.
- Original stereo/AFR behavior is not changed unless the new method is selected. Switching modes rebuilds D3D12 resources.
- The experimental layer does **not** implement eye tracking, adaptive crop, stereo quad views, or image-history correction.
- The first pair and any incomplete pair may produce a missing/old compositor frame; don't describe that as successful DuoView. Look for the `[DuoView]` logs.
- **No hardware validation has been performed** on PSVR2. CI pass verifies compilation and package structure, not in-game correctness or performance.

## Headset validation (STALKER 2, remleo r6 baseline held separate)

1. Fully extract to a NEW folder and run the included `UEVRInjector.exe`. Do not overwrite the working remleo package.
2. Select SteamVR OpenXR runtime; STALKER 2 must run in **DX12**. Disable DLSS Frame Generation.
3. Set Rendering Method to `DuoView Fixed Mono (Sequential DX12+OpenXR)`.
4. Confirm the logs print `[DuoView] Initialized fixed mono GPU views` with nonzero W/H. Confirm both eyes show identical content, with a crisp centre and softer periphery.
5. Rotate head while standing still and compare scene edge alignment, shadows/Lumen, transparency, UI, hitches, frametime against remleo r6 Synchronized Sequential. If image flickers or the fovea is offset, keep the logs and switch back to baseline.
6. Confirm toggling back to Synchronized Sequential restores its original stereo image.

Success criterion: complete mono image, central high-density region with no geometry seam, correct UI and shading, no injected crash or runtime submission errors, meaningful frametime reduction. Any failure is a failed experiment, not a reason to silently show baseline stereo.
