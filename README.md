# Umbra Admission — GitHub Action

> **Copyright (c) 2026 Binay Dalai. All rights reserved.**
> This repository is strictly for viewing and contributing to the original project. You may not use, copy, modify, distribute, or commercialize this code for your own personal or commercial projects without explicit written permission. Only the original author retains the right to use and monetize this project.


[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Umbra%20Admission-purple?logo=github)](https://github.com/marketplace/actions/umbra-admission)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Latest release](https://img.shields.io/github/v/release/bkd-dotcom/umbra-action?sort=semver)](https://github.com/bkd-dotcom/umbra-action/releases)

**Govern any coding agent's change to your repository, and attach a signed receipt.**

Every pull request — no matter which agent opened it (Claude Code, Codex, Cursor,
Copilot, Devin, or a human) — is run through the [umbra-core](https://github.com/bkd-dotcom/umbra-core)
admission pipeline:

```
executable contract  →  untrusted-text quarantine  →  required checks  →
independent verifier  →  earned authority (0/1/2)  →  Ed25519-signed receipt
```

The action posts the verdict as a PR comment, uploads the signed receipt as an
artifact, and **fails the check** unless the change earns the authority you
require. Make it a *required status check* and nothing merges without a receipt.
`auto_merge` is always false — Umbra governs the agent; a human merges.

## Usage

```yaml
name: Umbra Admission
on:
  pull_request:
permissions:
  contents: read
  pull-requests: write
jobs:
  admit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 0                       # base must be reachable for the diff
      - uses: bkd-dotcom/umbra-action@v1
        with:
          min-authority: "1"                   # 0 observe · 1 analyze · 2 branch-PR
          signing-key: ${{ secrets.UMBRA_SIGNING_KEY }}   # optional: stable signed receipts
```

Add a `.umbra/admission.yaml` to your repo to declare the contract (allowed and
forbidden paths, diff budget, required checks). Without one, a conservative
default applies. See the [umbra-core docs](https://github.com/bkd-dotcom/umbra-core).

### Also scan for vulnerabilities (SARIF → code scanning)

Turn on `scan` to run the SAST detection engine over the PR and upload SARIF to
GitHub code scanning alongside the admission verdict — deterministic, offline, and
free (7 languages, cross-file taint):

```yaml
permissions:
  contents: read
  pull-requests: write
  security-events: write        # to upload SARIF to code scanning
jobs:
  admit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with: { ref: ${{ github.event.pull_request.head.sha }}, fetch-depth: 0 }
      - uses: bkd-dotcom/umbra-action@v1
        with:
          scan: "true"
          scan-fail-on: "high"    # optional: fail the check on high+ findings
          min-authority: "1"
```


## Inputs

| Input | Default | Description |
|---|---|---|
| `mission` | review prompt | The bounded task the change claims to perform. |
| `min-authority` | `1` | Fail unless the change earns at least this level (0/1/2). |
| `agent` | `""` | Force `codex-cli` or `claude-code` to *re-run* the change. Blank governs the existing PR diff without invoking an agent. |
| `signing-key` | `""` | Base64 Ed25519 key (32+ bytes) for stable receipts. Falls back to a dev key (honestly flagged). |
| `require-sandbox` | `false` | Fail closed if code-executing checks (npm/pip install, go/cargo build) can't run in a real filesystem/network sandbox. |
| `scan` | `false` | Also run the **SAST detection engine** over the checkout and upload SARIF to code scanning (7 languages, cross-file taint, deterministic/offline; needs `umbra-core >= 0.5.0`). |
| `scan-fail-on` | `""` | With `scan`, fail the check if any finding is at/above this severity (`critical`/`high`/`medium`/`low`/`info`). Blank = report-only. |
| `umbra-version` | latest | Pin a specific `umbra-core` PyPI version (blank installs the latest hardened release). |
| `python-version` | `3.12` | Python to run on. |

## Outputs

| Output | Description |
|---|---|
| `authority-level` | The level the change earned (0/1/2). |
| `receipt-hash` | Canonical hash of the signed receipt. |
| `sarif-file` | Path to the SARIF file when `scan` is enabled (else empty). |
| `findings-count` | Number of detection findings when `scan` is enabled (else 0). |

## How it works

This action is a thin wrapper over the `umbra-core` PyPI package. It stages the
PR's change as a working-tree diff, runs `umbra admit`, and enforces the earned
authority. On Linux runners it installs bubblewrap so required checks run under a
real filesystem/network **sandbox** (the tier is recorded truthfully in every
receipt; it falls back to a lower tier only if the sandbox can't initialize). The
governance logic, contract, verifier, and receipts all live in
[umbra-core](https://github.com/bkd-dotcom/umbra-core).

Part of the [Umbra platform](https://github.com/bkd-dotcom/umbra-umbrella) — see the umbrella for the full integration catalog and compatibility matrix.

## License

**Copyright (c) 2026 Binay Dalai. All rights reserved.** This code is not open source. You may not use, copy, modify, distribute, or commercialize it for your own personal or commercial purposes without explicit written permission from the author, who alone retains the right to use and monetize this project. See [CONTRIBUTING.md](CONTRIBUTING.md).
