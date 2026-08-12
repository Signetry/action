# Changelog — Umbra Admission (GitHub Action)

Follows [Keep a Changelog](https://keepachangelog.com/) / [SemVer](https://semver.org/).
Pin `@v1` (moving) or an exact `@v0.1.3+` tag.

## [Unreleased]

## [0.3.1] — 2026-08-03

### Changed

- Default `umbra-core` install pinned to `git+https://github.com/Signetry/core@v0.5.4`
  (was `@v0.5.3`) following the umbra-core v0.5.4 source-available release.
- The `umbra-version` input is documented as a **source version tag** (umbra-core
  is source-available and installed from its source repo, not PyPI).
- The advisory reviewer workflow installs `umbra-reviewer@v0.1.1` from source.
- `@v1` moved to this release. No functional change to the admission pipeline.

## [0.3.0] — 2026-07-30

### Changed — licensing & distribution

- **All Rights Reserved.** The MIT `LICENSE` was removed; this Action is no longer
  open source. See the notice in the README and `CONTRIBUTING.md` (contributions are
  made under a copyright-assignment agreement).
- **Installs `umbra-core` from its source repo, not PyPI** — `umbra-core` was
  removed from PyPI, so the Action now installs it via
  `git+https://github.com/Signetry/core@v0.5.3` (default) or the tag given in
  the `umbra-version` input. Fixes workflows that would otherwise fail after the PyPI
  removal.

## [0.2.0] — 2026-07-30

### Added

- **Detection scan mode** (`scan: "true"`): runs the umbra-core SAST detection
  engine over the checkout and uploads **SARIF** to GitHub code scanning alongside
  the admission verdict — 7 languages, cross-file taint, deterministic and offline.
  Optional `scan-fail-on` gates the check on a severity threshold; new outputs
  `sarif-file` and `findings-count`. Requires `umbra-core >= 0.5.0` (older versions
  skip scan with a warning). SARIF upload needs `security-events: write`.

### Changed

- Default `umbra-core` floor raised to `>= 0.5.0` (detection engine, `--fix`
  fusion, bring-your-own-key secret redaction).
- The PR comment is now rendered by **umbra-core** (`umbra comment`) from the
  Admission Decision Pack, so the Action posts the exact canonical template the
  architecture freezes — identical to the hosted UI and CLI (Executor · Contract ·
  Trust boundary · Checks · Verifier · Proof gates · Receipt · Auto-merge, machine-
  readable reasons, and the L2/L1/L0 conditional line). No more Action-specific
  comment format that could drift from the receipt.

## [0.1.3] — 2026-07-22

### Security

- **Fixed a script-injection sink.** Action inputs (`mission`, `agent`,
  `min-authority`, `umbra-version`) are passed via `env:` and validated, never
  interpolated into a shell body.
- **Fail-closed PR staging.** The action errors if the base commit can't be
  fetched/reset, instead of silently admitting an empty diff.

### Added

- Installs **bubblewrap** on Linux and relaxes the unprivileged-userns clamp so
  required checks run **`sandboxed`** by default.
- `require-sandbox` input → `UMBRA_REQUIRE_SANDBOX` (fail closed on code-executing
  checks without a real sandbox).
- Defaults to installing the hardened `umbra-core>=0.1.3`.

## [0.1.0] — 2026-07-22

### Added

- Initial composite action: stages the PR diff, runs `umbra admit`, posts the
  verdict comment, uploads the signed receipt, and fails the check below the
  required authority.

> `v0.1.0`–`v0.1.2` (exact pins) are superseded — upgrade to `@v1`. See
> [SECURITY.md](SECURITY.md).

[0.1.3]: https://github.com/Signetry/action/releases/tag/v0.1.3
[0.1.0]: https://github.com/Signetry/action/releases/tag/v0.1.0
