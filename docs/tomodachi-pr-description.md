# feat: add experimental native Tomodachi Life Mac port

Adds a self-contained, Mac-only port of *Tomodachi Life: Living the Dream*
1.0.0 (`010051F0207B2000`). The original ARM64 game code is rewritten ahead
of time and runs natively on Apple silicon with the host service layer and
NVN on Metal. Original inputs and patch sites are verified.

Runtime, graphics, rewriter, translator sources/prebuilt, scripts, tests, and
licenses live in `games/tomodachi/`; builds do not read another game's folder.
The only outside change is one root README entry. No game content, keys,
firmware/fonts, saves, translated game shaders, or game apps are distributed.
The included NativeAOT translator is a checked build/runtime tool with source
and retained licenses.

Includes native preparation/transactional packaging, shared-font validation,
clock/timezone support, touch input, inline keyboard handling, and the upstream
PR #1210 controller/keyboard drawings adapted to this title. Keyboard fixes
preserve SDK state and submitted names. Immutable depth snapshots correct the
reproduced flickering highlights on Mii skin, hair, and clothing.

The five commits separate dependency copies, title integration, keyboard fixes,
lighting fixes, and build/validation documentation. Temporary dream-crash
diagnostics and unrelated TOTK/iOS work are excluded.

## Validation

- Apple silicon, macOS 27.2: both Metal 3 and Metal 4 built and packaged; both
  packages passed strict signature verification.
- Both binaries passed the loader probe (4,000 calls, four threads) and 3,117
  synthetic depth-feedback GPU checks with Metal validation.
- Parent suite: **1,313 tests passed, 169 skipped**, with desktop access.
  Title-local Python/native tests also passed.
- Metal 4 development play covered setup/menus, island/building/Mii interactions,
  settings diagrams, and accepted name entry using native save copies. Highlight
  on/off/on comparisons isolated the reproduced lighting defect.
- No iOS build or gameplay claim. Metal 3 gameplay on an actual macOS 15 machine
  is unverified. Tests/probes do not establish complete gameplay correctness.

## Remaining validation / review focus

Keep this PR a **draft**: broad picture accuracy, long gameplay sessions, save
round-trips, physical controller/reconnect coverage, and partial services/audio
remain. Photo capture previously hit an unsupported geometry shader. One dream
bubble triggered a null shader-entry abort; the bubble disappeared before the
instrumented retry, so this is unreproduced rather than a verified fix.
Guarded Mii/cloth heap growth and NVN memory alignment need review. No performance
improvement is claimed.

Build instructions, provenance, and detailed platform evidence are in
`games/tomodachi/README.md`, `third_party/COPIES.md`, and `docs/validation.md`.
