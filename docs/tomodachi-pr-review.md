# Tomodachi contribution review — upstream PR approval pending

The user authorized pushing the development and contribution branches to their
fork on 2026-10-31. Opening an upstream PR still requires separate approval.

## Proposed destination

- Fork: https://github.com/suniyao/MacNativeGames
- Branch: `codex/tomodachi-contribution`
- Parent/base: `ly05010419/MacNativeGames:main`
- Base verified from upstream: `933b6a0d1438de160208a0a13eb5e8d7a2cb8094`
- Proposed head: `23d4e0d9435f1bd01177ef0a76c963fedb237252`
- PR status: **draft**, pending broader graphics and gameplay validation
- Proposed title/body: [tomodachi-pr-description.md](tomodachi-pr-description.md)

The branch is saved in this local repository. The active working branch remains
`codex/tomodachi-metal4`; its unfinished work and the installed apps are preserved.
A clean review checkout is at `/private/tmp/macnativegames-tomodachi-pr`.
The branch's commits are durable in this repository even if that temporary
checkout disappears.

## Rules reviewed

[Parent CONTRIBUTING.md](https://github.com/ly05010419/MacNativeGames/blob/main/CONTRIBUTING.md)
requires new games to carry all their own dependencies and scripts, avoid
reading other games' folders, limit outside changes to one root README update,
retain licenses, keep generated/game data out of source, build touched versions,
run the parent/title tests, report played vs probe-only validation, and use
`type: description` commit messages.

The original local implementation depended on the TOTK folder and bundled
unrelated shared/TOTK/iOS changes in its history. That history is not used as
the proposed PR base.

## Improvements made for review

1. Built a new branch on current parent main, excluding unrelated TOTK and iOS
   changes from the proposed diff.
2. Made the title self-contained: runtime, graphics layer, rewriter, translator
   sources/prebuilt, tools, notices, docs, and tests are under `games/tomodachi/`.
3. Added exact copy-input hashes and retained dependency licenses. Omitted an
   unused Unicorn-based diagnostic; no CPU-emulation dependency is needed by
   the port or title tests.
4. Added title-local test commands and documented actual support/limitations,
   including remaining visual/gameplay limits and the guarded compatibility patches.
5. Improved packaging so icon/signing failures leave the installed app intact;
   added synthetic tests for failure preservation and final relative paths.
6. Made packaged diagnostics opt-in and required an explicit external font
   extraction output; corrected build instructions against real CLI help.
7. Recorded preparation-tool hashes in the existing preparation report.
8. Built, packaged, and loader-tested both renderer backends in an external
   validation workspace. Kept the full-suite failure report honest and reran
   the affected unchanged test separately.
9. Separated dependency copies from title integration into two conventionally
   named commits for easier review.
10. Fixed `InlineKeyboard::Calc` to return its SDK keyboard state. Previously the
    undefined return value could make the title repeatedly reopen its text field.
    Regression tests check lifecycle states; a restarted installed Metal 4 app
    accepted typing and updated the in-game Mii name preview. The name was left
    unconfirmed for the user. Both isolated backends rebuilt after the fix.
11. Corrected the submit lifecycle: `FinishedKeyboard` is applet exit, not
    Enter/Cancel/Disappear. The false notification reset the title keyboard
    state and discarded the submitted name. Native tests now assert no applet
    exit on submit/cancel/hide. After installing and restarting, the name
    remained on the game name screen with OK enabled. Both backends rebuilt
    and all title tests passed; persistence across restarts remains unverified.

12. Fixed the reproduced flickering character highlights by snapshotting sampled
    depth attachments. On/off/on comparison isolates the cause; Metal 3 and
    Metal 4 each pass 3,117 GPU checks with Metal validation. The negative control
    fails as expected. Both backends rebuilt, title tests and installed signature
    verification passed. All-scene correctness and performance remain unclaimed.

## Exact scope and checks

- `26413e0`: `chore: vendor isolated Tomodachi runtime dependencies`
- `f1e3005`: `feat: add experimental native Tomodachi Life Mac port`
- `3e8a525`: `fix: return native inline keyboard state to the game`
- `58401f4`: `fix: preserve submitted names without signaling keyboard exit`
- `ee44acc`: `fix: snapshot depth inputs used by character lighting`
- `23d4e0d`: `chore: trace missing shader entries during dream crash diagnosis`
  (diagnostics only; the dream-bubble crash remains unresolved)
- 567 added/changed files, 117,584 added lines; most are required local copies of
  runtime and licensed shader dependencies under the parent's self-contained
  layout rule. The only outside change is one added README line.
- Parent suite: 1,302 tests, 169 skipped, one untouched BOTW settings test failed
  once. Isolated reruns passed both on upstream and on the proposed branch.
- Title Python/native/window checks: passed.
- Metal 3 / Metal 4 builds, signatures, and native loader probes: passed.
- Source scan: no prohibited game/artifact extensions or symlinks; only the
  necessary shader translator is a native tool binary. Its pinned hash matches.
- `git diff --check`: passed. Imported vendor formatting and unified-patch
  context are preserved through narrowly scoped whitespace attributes.

External validation logs are `/tmp/tomodachi-pr-*.log`. The source-only clean
checkout contains no copied saves, generated headers, images, shader caches,
or compiled app products.

## Upstream PR approval pending

The fork push is authorized. Opening a **draft** PR against the parent remains
a separate approval. Acceptance is the maintainer's decision; broader visual/gameplay
validation and the unsupported photo shader still prevent a ready-for-merge claim.

Face fixes can continue after the PR opens: commits pushed to the same branch
update that PR. After it merges, later fixes would go in a new branch/PR.
