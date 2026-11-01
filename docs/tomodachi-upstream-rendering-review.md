# Tomodachi upstream rendering review — 2026-10-29

User observation: Mii shading currently looks normal; the control/cursor still
projects several rectangular patches onto grass. Green flickering persists.
The new screenshot is not evidence that every Mii shadow is fixed.

Reviewed the parent repository's public open and closed PR listing and searched
issues for shadow, flicker, green, rectangle and Tomodachi. No Tomodachi-specific
report or exact green-horizontal-flicker fix was found. Similar artifacts occur
in other ports, but that does not establish a single bug affecting every game.

## Relevant merged fixes and local comparison

| Upstream change | Evidence and limits | Local status / applicability |
| --- | --- | --- |
| [#67](https://github.com/ly05010419/MacNativeGames/pull/67), [#155](https://github.com/ly05010419/MacNativeGames/pull/155) | TOTK ground decals read stale attachment contents; initial pass splitting was followed by tile reads for overlapping decals. In-game before/after evidence. #155 also has a Metal 3 synthetic GPU test. | Local renderer lacks `att_unstored_read` / `att_fb_take`. Existing explicit barrier handling does not establish correctness for games that omit barriers between attachment-sampling draws. Strong investigation candidate for stale UI shapes, not a proven Tomodachi diagnosis. |
| [#547](https://github.com/ly05010419/MacNativeGames/pull/547) | Argument-cache invalidation now includes texture serial replacement and retired uniform snapshots. Synthetic production-path GPU regression; no gameplay claim. | Local legacy `args_sync_dirty` checks only registry and dirty generations. The newer optional cache key already includes serial/snapshot generations, but that does not fix the legacy path. Concrete shared-renderer correctness gap. |
| [#104](https://github.com/ly05010419/MacNativeGames/pull/104) | D16 shadow compare-reference rounding and nearest non-compare depth reads fixed TOTK dark patches/cascade blocks. Tested TOTK 1.0.0. | Local `totk_msl_fixup.h` absent. Do not apply its assumption that array comparisons are D16 to another game without checking the actual texture format. |
| [#222](https://github.com/ly05010419/MacNativeGames/pull/222) | Preserves the NVN fragment control section's early-depth-test flag; fixed dotted lattice surfaces in TOTK. | Local translator/runtime do not forward `early_fragment_tests`. Candidate for depth artifacts; not proof about pure-green pixels or cursor shadows. Requires shader-control layout validation and cache migration. |
| [#193](https://github.com/ly05010419/MacNativeGames/pull/193) | Larger byte snapshots distinguish writes to neighboring data from writes to terrain normal maps; fixes skipped write-backs. | Relevant resource-alias/write-back mechanism, but current grass texture recovery came from the separate full bindless-table fix. Avoid treating the old flat terrain and current cursor patches as identical. |
| [#842](https://github.com/ly05010419/MacNativeGames/pull/842) | Smash gained shader-emulated fixed-function alpha testing. | Local code still reports alpha testing as unimplemented. No matching alpha-test warning was found in the inspected normal Tomodachi profile log; not established as the cursor cause. |

These are software correctness fixes used by the Metal 4 parent, not evidence
that switching API versions automatically solves the symptoms. In particular,
#155's standalone test explicitly runs on Metal 3. The user's Metal 3 PR
[#6](https://github.com/ly05010419/MacNativeGames/pull/6) is closed and unmerged;
the local branch does not automatically inherit later parent fixes.

## Follow-up order

1. Port and exercise #547's cache invalidation regression on Metal 3.
2. Trace the cursor's attachment reads and resource identities, then adapt the
   attachment-read ordering/tile-read fixes with their GPU tests. Preserve full
   8192-entry bindless handling and the verified Mii mip fixes.
3. Verify depth formats and shader flags before applying TOTK-specific shader
   transformations. Investigate green pixels separately unless a capture ties
   them to the same draw/resource failure.

No runtime or packaged app was modified by this review; none of the candidate
changes has yet been validated against the remaining Tomodachi artifacts.
Public PR metadata and diffs were read into `/tmp/mng-prs-*.json` and
`/tmp/mng-pr-*-files.json`, outside the source tree.

## Cache invalidation backport and Metal 3 result

Ported #547's production fix and its synthetic regression into the shared
renderer. Before the fix, the regression aborted at mode 0, snapshot 0,
stripe 8: red channel 96 instead of 191 (the replaced texture stayed cached).
After tracking both serial and snapshot-retirement generations, the complete
`probe-gpu-arg-epoch` passed 23,354 checks with `MTL_DEBUG_LAYER=1` and
`TOTK_RENDER_STREAM=0`. The new regression covers five cache configurations.
The packaged Desktop app was replaced after archiving the preceding app and
passed strict codesign verification.

Visual check: green horizontal specks remain visible in the yellow loading
transition with this fix. A room's Mii face still renders normally. This is a
confirmed correctness fix, not a claim that the reported graphics symptoms
are resolved. Build/test logs: `/tmp/tomodachi-cache-{before,after}.log` and
`/tmp/tomodachi-cache-build-fixed.log`. Session evidence is outside the source
under `Development/Tomodachi_build/host/tests/upstream-cache-fix-qj3mireg`.

## Green-speck follow-up — 2026-10-29

Re-read the public upstream PR listing (all four API pages, open and closed).
Metadata evidence is external: `/tmp/tomodachi-pr-review-{1,2,3,4}.json`.
Two additional precedents matter:

- [#693 (Ori)](https://github.com/ly05010419/MacNativeGames/pull/693), merged:
  bloom kernels read garbage at tile edges, producing bands every 75 rows.
  The fix forwards compute shared-memory bytes from NVN control+0x754 through
  `totk_shader_rt.m` and `Service.cs`, includes the value in variant identity,
  and rejects cached variants missing it. Read-only comparison with the separate
  updated-main checkout confirms those changes in Ori. Tomodachi's shared runtime
  forwards local workgroup dimensions but omits shared-memory bytes; its service
  also omits them, while Program.cs already accepts a `shared` job field and
  defaults it to zero. This is a concrete translation gap, but applicability
  requires showing that an affected Tomodachi compute shader requests nonzero
  shared memory. Repeated rows make it a useful candidate, not a proven cause.
- [#87 (TOTK)](https://github.com/ly05010419/MacNativeGames/pull/87), merged:
  forcing anisotropy onto clamped data samplers created bright green pixels on
  Link's inventory outline. The fix restricts the override to repeating,
  linearly filtered, mipmapped material samplers. Local sampler creation simply
  uses the guest's requested anisotropy (clamped to 1..16); it has no TOTK_ANISO
  override, so the specific forced-filtering defect is not present here.

Other useful limits: [#1282 (XCX)](https://github.com/ly05010419/MacNativeGames/pull/1282)
reports grass flicker reduced by an opt-in immutable wind-input snapshot, but
explicitly leaves the producer/versioning cause unresolved. It is game-specific.
[#157](https://github.com/ly05010419/MacNativeGames/pull/157) traced purple bloom
and pulsing lights to temporal viewport jitter on attachment-reading decals;
its per-draw NaN probe is a useful diagnostic precedent, not a green-speck fix.

Next discriminating check: record nonzero compute shared-memory requests and
identify the first dispatch/draw introducing the green rows. If shared memory
is used, adapt Ori's forwarding plus selective cache migration, and compare
consecutive game frames before/after with the same scene. If not, prioritize
attachment read/order and alias/write-back evidence rather than importing the
Ori change as an asserted visual fix. No runtime/app was changed by this review.

## Attachment-feedback isolation and green-speck fix (2026-10-29)

The actual game reproduced the green rows after loading the copied profile.
Compute shared-memory forwarding regenerated the 384-byte `787155ba` kernel
with 96 uints instead of the old 1024; specks persisted, so Ori's missing-field
bug was not their cause here. The forwarding, cache identity and matching AOT
service are retained as a separate correctness change, preserving full handles.

At frame 2930, pass 471297's fragment `149900e651b25f93` sampled texture serial
6998 in register 0 while rendering to that same 1920x1080 RG11B10 texture.
The final image had pure-green corruption concentrated on row 1024 (and other
128-row boundaries). Skipping that shader reduced but did not eliminate specks.
The new `TOTK_ATT_SNAPSHOT=1` path ends the earlier pass, blits sampled
single-level, single-sample 2D colour attachments to immutable textures, and
binds those inputs for the draw. Resources remain resident through completion;
argument caches are invalidated both on substitution and on the following draw.
Storage images remain bound to their writable originals. Full indirect texture
handle arrays participate in detection. Other view/texture shapes remain outside
this implementation's scope. Tomodachi's app launcher enables it; other titles
retain their defaults. Runtime `flag att_snapshot 0|1` permits comparison.

Same-process on/off/on evidence in the actual island:
- On: 8 consecutive frames, zero pixels with G>180, R<30, B<30.
- Off: 8 frames, 61..250 such pixels per frame.
- On again: 16 frames, zero such pixels.
- Optimized build (copies only sampled colour targets, excludes depth-only
  dependencies): another 8 consecutive frames with zero such pixels.
The user also confirmed green specks gone. Snapshots incur extra copies/pass
boundaries; these runs do not establish unchanged performance or every scene.

Evidence: user-data tests `green-shared-1vq817do`,
`green-feedback-b_xd0cdp`, `green-feedback-fnskmc_w`; logs, pass dumps and PNGs
remain there. Build/package logs are `/tmp/tomodachi-green-release-*.log`.
The Desktop app was replaced with this change and strict codesign verified.

Remaining findings:
- Mii hair/hat block patches remain visible in the actual portrait panel.
  The early-depth flag experiment produced no enabled variants in the inspected
  scene and was removed; it is not a portrait fix.
- Building entry is corrected by the 2026-10-30 readback follow-up below.
- The supplied RenderTest PID 15415 photo crash matches the external session log:
  `unsupported NVN stage 2`, shader `99d4fa6451ac93c4...`, aborting in prog_get
  through pso_predict_rest. Geometry-stage translation/execution is absent.
  No photo-completion or crash-fix claim is made.
- TOTK's existing updater completed at 19:06:53; no restart was needed.

### Building picking and readback, 2026-10-30

Controller A and keyboard J both reached the guest as button bit 0. Stable-cursor
tests still could not select buildings, so remapping was not the cause. GPU
tracing identified CPU-visible 32x128 RG8 readback surfaces (serials 6308/6309).
Serial 6309 started at page offset 0xe00 and was invalidated over 0x4000 bytes. It was skipped
on alternating frames because ordinary game writes dirtied its shared final host
page (writers main+0x21289c, +0x2315dc, +0x2ccf3c). Comparing retained pre-render
texture bytes showed those bytes had not changed.

`wb_note` now retains pre-render bytes for CPU-cached textures up to 64 KiB.
Invalidation may recover GPU readback when page dirtiness is a false positive
and those bytes still match. Actual changed bytes retain the CPU result. Pending
records are also removed with their memory pool, preventing old texture results
from overwriting a newly allocated pool at the same address. The existing GPU
linear-render-target probe exercises clean readback, writes before/after the
texture on shared pages, preservation of a CPU edit inside the texture, and
reuse after pool destruction. It passes 459,152 checks with Metal validation;
`TOTK_PROBE_READBACK_NEGATIVE=1` restores the old decision and fails at neighbour
write mode 1, texel 0. `flag writeback_exact 0|1` is available for diagnostics.

Actual game validation: an isolated save-copy session showed the restaurant
name on hover, opened its cutaway interior, then selected the pawnshop and
entered its inventory screen using J/A. This establishes building interaction
in these scenes, not a fix for Mii faces or unsupported photo geometry. The
physical controller still needs user confirmation. The Desktop app was replaced
and its installed executable passed the same GPU probe; codesign verification
passed. No persistent saves were edited by this test.

External evidence: user-data `tests/green-feedback-f7bqv0jk/runtime.log` and
`building-pawnshop-verified.png`; `/tmp/tomodachi-building-{build,package}.log`,
`/tmp/tomodachi-readback-{probe,negative,installed}.log`.

### Character lighting feedback, 2026-10-31

The user's observation that bright mesh regions break up matched the capture:
lighting shader `94793852...` samples depth register 19 while that image remains
the current depth attachment. The face palette/normal inputs were intact.
Position-invariance and cache-disabling experiments did not fix this artifact
and are not shipped as fixes.

The renderer now snapshots sampled depth after landing pending clears. Copies
are reused only inside the same unchanged read-only pass; depth writes, pass
changes, pending clears, or a different backing object invalidate reuse. Support
is single-level, single-sample 2D textures. The title ID gates the default; other
games retain their defaults. `TOMODACHI_DEPTH_SNAPSHOT=0` and runtime
`flag depth_snapshot 0|1` support negative comparison.

Same-process lineup comparison: on 8 frames clean; off 16 frames restored
rectangular patches; on again 24 frames clean. Evidence remains outside source:
`~/Library/Application Support/MacNativeGames/Tomodachi/tests/character-flicker-zlyzebs8/`
(`captures/depth-on-*`, `depth-off-*`, `depth-on-again-*`, `runtime.log`).

Synthetic production-path GPU coverage passed 3,117 checks on Metal 3 and Metal
4 with Metal validation: D16, D32, depth-stencil, queued clears, copy substitution,
read-only reuse, pass changes, depth writes, backing replacement, and sampled
pixel readback. Disabling depth snapshots fails as expected. Logs:
`/tmp/tomodachi-depth-probe-{metal3,metal4,negative}.log`. Both self-contained
backends rebuilt, title tests passed with the required Python environment, and
the regular Desktop app was replaced and its strict signature verified. It
opened successfully at the host-time-changed notice; further navigation was
left to the user.

This resolves the reproduced highlight patches, not every rendering defect.
Broader gameplay/save validation and unsupported photo geometry remain open.
Extra copies/pass boundaries may affect performance; no performance claim.
