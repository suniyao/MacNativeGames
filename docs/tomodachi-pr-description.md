# feat: add experimental native Tomodachi Life Mac port

**Draft — broader graphics, gameplay, and save validation remain.**

Adds a self-contained experimental Mac port of *Tomodachi Life: Living the Dream*
1.0.0 (`010051F0207B2000`). The player's ARM64 code is rewritten ahead of time,
loaded natively on Apple silicon, and connected to the native service host and
NVN-on-Metal graphics layer. Builds verify original input identities and guarded
patch sites and keep generated data outside source.

All runtime, graphics, rewriting, shader translation, packaging, documentation,
and title tests live under `games/tomodachi/`. The only change outside that
folder is one dated root README entry. Copied dependencies are separated into
the first commit, with per-file provenance and retained licenses, so the second
commit contains the title integration.

The port includes local-data preparation and packaging, shared-font validation,
native text/inline keyboard handling, touch gestures, clock/timezone support,
and upstream PR #1210's keyboard/controller drawings and mapping helpers adapted
to this title. Controller settings navigation follows PR #977/#1210. Existing
native key/swap preferences migrate. Diagnostics are off by default; packaging
finishes and verifies the new app before archiving the installed copy.

## Validation

- Apple silicon Mac, macOS 27.2: self-contained Metal 3 and Metal 4 builds
  completed; both app packages passed code-signature verification.
- Both binaries passed the native cross-module loader probe (4,000 calls,
  four threads) using locally verified player modules. This does not establish
  gameplay correctness.
- Metal 4 development sessions exercised native setup/menus, text entry,
  settings/controller drawings, and an existing island/native save copy,
  including Mii/room interactions. The final isolation refactor has fresh build,
  package, and loader checks; it has not had another full gameplay sweep.
- Title-local tests pass: four Python tests (rewriter and packaging), timezone/DST,
  gesture ABI/event ordering, inline keyboard lifecycle/Unicode, saved settings,
  guest remapping, and simulated controller settings navigation. The inline
  keyboard regression also checks the SDK state return contract; after fixing
  that return value, a live Metal 4 Mii name prompt accepted typing and updated
  the guest preview without repeatedly reopening the field. A further fix
  separates name submission from applet exit; after restarting, the submitted
  name remained on the game's name screen with OK enabled. Submit/cancel/hide
  regression tests ensure no false applet-exit callback is emitted.
- Parent suite: 1,302 tests, 169 skipped, one failure in the untouched
  `test_settings_panel.SettingsPanel.test_current_modes_and_refresh`
  (`GAME_BOTW160`, compact, not overridden). That test passed when run separately
  on unchanged upstream and again on this branch. Reporting the original
  failure rather than claiming an entirely green full-suite run.
- Python sources parse, the prebuilt translator hash matches, source-only artifact
  checks pass, and the diff is limited to this game plus the README entry.

## Known blockers / review focus

- The reproduced block-shaped highlights on faces, hair, and clothing are fixed
  by immutable depth-feedback snapshots. Same-process on/off/on testing removed,
  restored, then removed the defect (8/16/24 captured frames). Metal 3 and Metal 4
  each pass 3,117 synthetic GPU checks with validation; the negative control fails.
  Broader picture correctness remains unverified. Photo capture has separately
  hit an unsupported geometry-stage shader; keep this contribution a draft.
- Opening a Mii dream bubble has triggered a null shader-entry abort. Additional
  diagnostics are built; a fix is not yet established.
- Full island gameplay, long sessions, and save round-trips need more validation.
- Guarded Mii/cloth heap growth and physical NVN backing alignment need review.
- Some services/audio effects are incomplete.
- No iPhone/iPad support or validation is claimed. Metal 3 builds on the test Mac,
  but this draft does not claim gameplay validation on an actual macOS 15 device.
- Controller navigation has synthetic-event coverage; no physical-controller
  compatibility sweep across families/reconnect scenarios is claimed.

No game files, keys, firmware, fonts, extracted icons, saves, translated game
shaders, or game apps are distributed in this PR. The native shader translator
is the verified build-tool prebuilt required by this title; its source and
licenses are included locally.

Related upstream controls work: https://github.com/ly05010419/MacNativeGames/pull/1210
and https://github.com/ly05010419/MacNativeGames/pull/977.
