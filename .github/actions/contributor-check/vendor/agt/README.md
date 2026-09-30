# Vendored AGT contributor-check scripts

These files are copied from
[microsoft/agent-governance-toolkit](https://github.com/microsoft/agent-governance-toolkit),
MIT License, at commit `359a6b8cf453f95d6bc9caf932057e1e78ccffd9`
(`main` as of 2026-07-31), from the `scripts/` directory. Each file keeps its
original copyright and licence header.

They were previously fetched at action runtime by a pinned `actions/checkout`
with `sparse-checkout: scripts`. That coupled contributor gating to an upstream
repository's internal script layout: a layout change upstream becomes a runtime
failure here, and a runtime failure used to be reported as a contributor-risk
level. See agentrust-io/.github#27.

## Contents

| File | Upstream path | SHA-256 |
|---|---|---|
| `contributor_check.py` | `scripts/contributor_check.py` | `f1c1179c921f59f836e85355b2efdf8641f65b763516f05853f4d216e70ae164` |
| `contributor_check_allowlist.json` | `scripts/contributor_check_allowlist.json` | `d8a5ddbb0695c21b3eae88e6bc85edcda6cbd9721cad3ca9d10fbeeed4fc0636` |
| `credential_audit.py` | `scripts/credential_audit.py` | `9650675038e7720a6d5cdd8be058fa38b2e004cc1e6f7bc81c48ed100e4680e1` |
| `cluster_detect.py` | `scripts/cluster_detect.py` | `da36c10074aa638bf61c5873fbe6cff975533563553689eba5e90329e3d8d7cf` |
| `contributor_check_action.py` | `scripts/contributor_check_action.py` | modified, see below |

The first four are byte-identical to upstream at that commit. The checksums
above are the upstream bytes, so `sha256sum` here is a one-line verification.

## The one local modification

`contributor_check_action.py` differs from upstream in `_run_check` and in the
block of `main()` that calls it. `_run_check` now reads the subprocess return
code. A check that could not be executed raises `CheckExecutionError` and the
action exits non-zero with an operational error, instead of returning the
contributor-risk level `UNKNOWN`.

`UNKNOWN` still means what its upstream comment says it means: a check that ran
and could not determine an answer. That case is unchanged and still labels.

## Why more than two files

`contributor_check_action.py` resolves the scripts it runs as siblings of
itself, through `Path(__file__).resolve().parent`. The composite action runs
`profile,credential` by default and `profile,credential,cluster` on dispatch,
so `credential_audit.py` is invoked on every run and `cluster_detect.py` on
some. `contributor_check.py` reads `contributor_check_allowlist.json` from its
own directory and silently grants no exemptions if it is absent.

Vendoring only the two files named in #27 would leave those siblings missing,
and with the return-code change that is now a hard failure on every run rather
than a silent `UNKNOWN`. All five move together or none of them do.

## Updating

Re-copy from the same upstream paths at a newer commit, update the commit SHA
and the checksums above, and re-apply the `_run_check` modification. Upstream
has a known divergence between `scripts/` and the packaged
`agent_compliance/cli/` copies (AGT#3571), so check which copy carries the
retry behaviour before advancing.
