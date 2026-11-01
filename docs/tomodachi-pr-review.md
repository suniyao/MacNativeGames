# Tomodachi upstream contribution — approval pending

The fork push is authorized. Opening an upstream PR still awaits the user's
separate approval. No parent branch is pushed to directly.

## Concrete proposal

- Parent: `ly05010419/MacNativeGames:main`
- Verified parent base: `92ec79bd10c006e1cb7ca32d7ac1d8f66c758a15`
- Fork: `suniyao/MacNativeGames`
- New review branch: `codex/tomodachi-upstream`
- Proposed head: `d065d21347079267215c043dd10ee19b9791a05b`
- Proposed title/body: [tomodachi-pr-description.md](tomodachi-pr-description.md)
- Status: **draft**, with actual gameplay and remaining gaps stated explicitly.

The review branch is pushed to the user fork and its remote head was verified.
The earlier development and contribution branches are preserved. This new branch
was built on current parent main without force-pushing existing history.
The active project branch remains `codex/tomodachi-metal4`. The self-contained
review checkout is `/private/tmp/macnativegames-tomodachi-pr`; its branch is also
retained in this repository.

## Organization and rule checks

Reviewed the current parent CONTRIBUTING.md, source-test workflow, and new
`.githooks/pre-commit` content check. Enabled that hook in the review clone.
The parent requires native ARM64 execution, title-local copies of dependencies,
retained sources/licenses, verified original inputs, no distributed game content,
minimal outside changes, tested platforms, diagnostics off by default, and
conventional commit messages.

1. `b37c3ea` — dependency copies and retained licenses/input hashes.
2. `28737c9` — native Tomodachi port, build/package/input/UI integration.
3. `6c181c0` — SDK keyboard state and accepted Mii name lifecycle.
4. `1ef08e5` — immutable depth inputs for character lighting with GPU regression.
5. `d065d21` — local build paths, precise provenance/validation, honest limits.

The original two keyboard fixes are combined into one review topic. Temporary
dream-crash diagnostics remain on the backed-up development branches and are
excluded here. Inherited tool instructions now name this title's local paths;
inherited cache-equivalence claims have been removed.

Diff: 568 files, 117,673 added lines. Most are required dependency copies. The
only outside change is one root README line. Cross-game paths occur only in copy
provenance metadata, not executable build dependencies. Source scans found no
game data/keys, symlinks, generated outputs, or unexpected binaries; the necessary
translator checksum matches. Python parsing and `git diff --check` pass.

## Validation

- Fresh parent suite with desktop access: **1,313 tests, 169 skipped, no failures
  or errors**. The initial restricted run could not access display metrics and
  temporary preferences; the desktop rerun passed.
- Title Python/native tests: passed (four Python tests, clock/DST, gestures,
  keyboard lifecycle/Unicode, settings validation and simulated remapping).
- Both self-contained Metal 3 and Metal 4 builds/packages/signatures: passed.
- Both loader probes: 4,000 cross-module calls on four threads, passed.
- Both depth-feedback GPU regressions with validation: 3,117 checks each, passed.
- Existing Metal 4 gameplay evidence and remaining limits are in the proposed
  title-local validation notes. Neither iOS nor actual macOS 15 gameplay is claimed.

Logs and packaged test apps stay outside source (`/tmp/tomodachi-upstream-*`).
Regular installed apps and user saves were not replaced by this review build.

## Approval step

Approve opening a **draft** pull request from this exact fork branch/head to
`ly05010419/MacNativeGames:main`. The maintainer decides acceptance. The dream
bubble disappearing prevented reproduction and is not presented as a fix; broad
graphics/gameplay/save validation and the photo shader remain review gaps.

Later fixes can be committed to the same PR branch, tested, and pushed to update
the open PR. After merge, further fixes use a new branch and PR.
