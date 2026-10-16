# UEVR-Foveated: R0 Verified Baseline and Source Build

## Baseline, not an experiment

This branch was created directly from the exact tested **remleo UEVR 01143-r6** commit:
`4dd185f94b72f8cd385000d4ed06d2648155ffda`.

The only changes in this branch are GitHub workflows and this document. No renderer code, graphics algorithms, VR configuration, or engine hooks were modified. **Do not rebase this baseline to the moving remleo branch.**

- Fork: https://github.com/liviutrn/UEVR-foveated
- Upstream source: https://github.com/Remleo/UEVR
- Upstream release: https://github.com/Remleo/UEVR/releases/tag/remleo-01143-r6
- Original binary SHA256: `824709c32adfb37264f50a5b79d9dfbc132eaf0ad8eaa9853c770ca534b646e2`
- Exact SDK submodule gitlink: `remleo/UESDK@b1c6534652bd7381ad16e25f0507e3162e4b4691`
- Matching frontend: `Remleo/uevr-frontend@b2fac6086a87714f5bb87637e65ff0b3c4a3fad7` (latest frontend commit preceding r6 release)

## Artifacts

**R0 verified remleo r6 binary**: downloads the byte-identical original UEVR ZIP, confirms SHA256, verifies UEVRBackend.dll and UEVRInjector.exe exist, then uploads ZIP + provenance. This is *not* a source compilation. Workflow runs automatically on push to this branch.

**R0 source compilation**: compiles from this branch with its exact UESDK submodule revision, compiles remleo's matching frontend, overlays those new compiled files into an original-release package, and uploads a new Windows ZIP. It must pass in GitHub Actions and then be tested on PSVR2 before being called equivalent to r6.

## Private SDK access required for source build

GitHub's `GITHUB_TOKEN` for this repository cannot automatically clone private repositories owned by other users. The owner should configure repository secret **UESDK_PULLER** at:

`Settings -> Secrets and variables -> Actions -> New repository secret`.

Set the value to an authorized access token with **read-only Contents access** to `remleo/UESDK` and any nested private repositories required for its submodules. Prefer a limited-scope fine-grained token where the repository owner allows it; otherwise an appropriately restricted classic PAT may be necessary. Do **not** post tokens in issues, pull requests, chat or logs. Once configured, rerun the failed source-build job; or make this branch default and run its manual workflow. **Access in your GitHub browser alone is not enough for Actions.**

Note: EpicGames/UnrealEngine is *not* a drop-in replacement for remleo/UESDK. Also, the connected GitHub app currently gets 404 for both `remleo/UESDK` and `praydog/UESDK` despite user browser access.

## Regression test to pass before experiments

1. Extract built ZIP into a separate directory; keep the original stable r6 package untouched.
2. Same S.T.A.L.K.E.R. 2 installation, SteamVR/OpenXR, PSVR2, game profile and settings.
3. Normal UEVR injection succeeds, no UE crash, no compositor black screen.
4. Menus/UI, HMD tracking, controllers, depth and both-eye shading match working r6.
5. Canted/Raw projection and aim transition remain correct; no new hitches.
6. Save logs and frametime comparison. Only then mark baseline verified.

Next experimental branches (do not contaminate baseline): `experiments/duoview-fixed` (fixed-center high-res fovea plus low-res periphery) and later `experiments/quadviews`. Eye gaze and temporal-history correction are **not** part of R0.
