# TOTK local update investigation (2026-10-30)

Upstream: `ly05010419/MacNativeGames`, commit
`933b6a0d1438de160208a0a13eb5e8d7a2cb8094`.
The existing installed game is TOTK 1.4.3. The local Tomodachi branch and
uncommitted changes were retained; the app update uses a separate upstream checkout.

## Immediate Play exit

Reproduced with `TOTK_DEBUG_LOG=1` on the installed launcher. The game loaded
its modules and verified RomFS, then stopped with:

```text
RomFS overlay extracted/v143/romfs-patches-out/1920x1080-scene-1920x1080 missing
FAIL: HeapDef overlay for this output / scene size missing
```

The packaged app contained only the older `2560x1440` overlay. Basic launcher
`--check` and loader `probe` both passed: neither exercises boot-time overlay
mounting. A successful loader check alone therefore does not verify this fix.

Generate the current script's four canvas/scene combinations (1920x1080 and
2560x1440) from the existing verified local RomFS with
`make_output_res_overlay.py <canvas> --scene <scene> --version v143`, and package
their manifests and files alongside the runtime update. Keep all generated game
data outside this source project.

## Local verification and artifacts

The upstream runtime passes its native loader probe from the external build
folder. Launching the validation executable inside the temporary `.app.partial`
bundle stalled before runtime entry on this machine. The repair validates an
identical cloned executable outside the bundle, with the staged app as its
working directory, and uses the same approach for pipeline preparation. The
normal updater still signs and verifies the completed app before installation.

Existing external build workspace:
`~/Documents/Codex/2026-10-24/can-you-help-me-check-my-2/work/TOTK_build`.
Its `logs` folder contains `runtime-probe.log`, `repair-overlay-*.log`,
`mac-pipelines.log`, and the updater's signing logs.
Gameplay validation uses a save copy under
`~/Library/Application Support/MacNativeGames/TOTK/tests/2026-10-30-upstream-933b6a0`.

## Completed validation

- Updater completed installation and the post-install launcher check.
- Graphics preparation produced 27,498 render and 325 compute pipelines for
  `applegpu_g16g`, with zero failures and zero write failures.
- Installed runtime logged commit `933b6a0` and successfully verified/mounted
  `2560x1440-scene-2560x1440`.
- Visually verified the title screen and Continue save menu using the engine's
  diagnostic frame capture. The user also confirmed the title window was visible.
- Loaded the copied save into rendered gameplay at Rabella Wetlands Skyview Tower.
- Test artifacts include `title-window.png`, `continue-menu.png`, `gameplay.png`,
  and `verified-launch.log` under the tests folder above.
- Closed the test session normally and reopened the existing desktop shortcut
  without the test-profile override.

The `/Volumes/mac-APFS` volume referenced by AGENTS.md was unavailable. The
connected external volume was `/Volumes/SD`, FAT32, approximately 494 GB.
This update uses the existing internal artifact workspace outside the source
project instead; no drive was reformatted.
