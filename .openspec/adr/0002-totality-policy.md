# ADR-0002: Totality Policy — Explicit `total fn` for All Terminating Functions

**Status:** Accepted
**Date:** 2026-06-18
**Context:** MVL infers totality for functions that have no unbounded loops and no calls to `partial fn`. These appear as `total*` (implicit) in `mvl assurance`. pkg-rest must decide whether to rely on inference or annotate explicitly, consistent with the policy established in pkg-http (ADR-0002 there).

## Decision

**All functions that terminate must carry the explicit `total fn` keyword.** Implicit totality (`total*`) is not permitted in source files.

Rationale:
- Explicit annotation is a contract: the author asserts termination and exhaustiveness, the checker verifies it.
- Implicit totality is a silent default that can be silently broken by adding a call to a `partial fn` — the compiler will catch it, but only if the caller was already marked `total fn`.
- `partial fn` is already explicit; totality must be equally visible.
- `mvl assurance` target: `0 implicit total fn`.

## Application in pkg-rest

| Function | Keyword | Reason |
|---|---|---|
| `rest_error_msg` | `total fn` | Exhaustive enum match; `https_error_msg` is `total fn` in pkg.tls.https |
| `json_headers` | `total fn` | Pure map builder; no loops, no partial callees |
| `map_http_err` | `total fn` | Trivial enum constructor wrapper |
| `is_success` | `total fn` | Pure arithmetic predicate |
| `merge_headers` | `total fn` | Bounded `while` loop with `decreases keys.len() - ki` |
| `parse_json_body` | `partial fn` | Calls `std.json.decode` which is `partial fn` |
| `rest_post_json` | `partial fn` | Net effect; calls `https_post` which is `partial fn` |
| `rest_get_json` | `partial fn` | Net effect; calls `https_request` which is `partial fn` |

### Termination proof for `merge_headers`

`merge_headers` uses a `while ki < keys.len()` loop with `ki` incremented by 1 each iteration. The termination measure is `keys.len() - ki`, which:
- Starts at `keys.len()` (non-negative, since list lengths are `>= 0`).
- Decreases by exactly 1 each iteration.
- The loop guard `ki < keys.len()` ensures the measure is positive while the loop runs.

The `decreases keys.len() - ki` annotation makes this proof obligation explicit and machine-checkable.

## Test file mirrors

Mirror functions in `*_test.mvl` intentionally omit `total` — they are not separately verified by the assurance tool and exist only to satisfy the test runner's scoping rules (see the test-file mirror pattern workaround for issue #96).

## Consequences

- `make assurance` must report `0 implicit total` — use this as the merge gate.
- Any new pure helper must be annotated `total fn` before it can be called from an existing `total fn`.
- Bounded loops require a `decreases` expression; unbounded loops require `partial fn`.
- Reviewers must reject PRs that introduce implicit totality.

## Connected to

- MVL Req 3 (Totality): `mvl assurance` REQ3 verifies this
- MVL Req 8 (Termination): `total fn` functions must have provable termination
- ADR-0001: REST client design — identifies which functions are total vs partial
