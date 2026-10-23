# ISSUE-006 — Compression: encode short repeated follow-up blocks as RLE

State: Implementing
Authorized-Work: Pull-Request-Implementation
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-23
Updated: 2026-10-24
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [S]: The early no-compression return for inputs shorter than seven bytes bypasses the existing RLE selection.

## Reach-and-Impact

Reach [O]: A default CLI stream containing 131,072 copies of `a` followed by six copies of `b` ends in a six-byte raw block.
Impact [O]: The original frame is 32 bytes; the implemented encoder produces a valid 27-byte frame.
The earlier equivalent-byte-stream experiment and the implemented encoder both preserve decoded content.
Impact [S]: Eligible two-to-six-byte blocks save one to five bytes each; aggregate workload benefit is unknown.

## Evidence

- [S] `lib/compress/zstd_compress.c:3302-3308` returns early and `:4430-4433` bypasses the RLE check at `:4452-4463`.
- [S] `lib/compress/zstd_compress_internal.h:675-694` defines raw size as `3 + srcSize` and RLE size as four bytes.
- [O] `LC_ALL=C awk 'BEGIN { for(i=0;i<131072;i++) printf "a"; printf "bbbbbb"; }' | programs/zstd -q -c | od -An -tx1` shows the raw tail with Bash pipefail enabled.
- [O] Replacing only `31 00 00 62 62 62 62 62 62` with `33 00 00 62` preserves decoded SHA-256 `2f644d8d40796bfea743c4a57d2d72d595fe932f7684c492aa9afb5e65066119`.

## Prior-Art

Coverage: Current source and targeted open/closed GitHub issue/PR searches; checked=2026-10-23.
Gaps: Exhaustive discussions and small-flush workload frequency remain unchecked.

- https://github.com/facebook/zstd/issues/4457 — Distinct; worker-job first-block state, not the short-block early return.
- https://github.com/facebook/zstd/pull/4851 — Distinct active implementation of the worker-job issue; do not duplicate it.

Contribution fit: A separate bounded short-block correction preserving the true first-frame-block compatibility guard.

## Proposed-Change

Allow RLE for identical two-to-six-byte follow-up frame blocks after required sequence-state bookkeeping.

## Scope-and-Constraints

- Preserve first-frame-block compatibility with old decoders, zero/one-byte behavior, block API behavior, and sequence-collector errors.
- Preserve LDM skips, repcodes, entropy-table ownership, last-block flags, and savings accounting.
- Exclude the independently owned MT fix and broad RLE refactoring.
- Authorization: The user's 2026-10-23 instruction explicitly requests implementation of this recommendation.

## Verification

- [O] The actual CLI reproducer changed from 32 to 27 bytes.
- [O] The existing `tests/fuzzer.c` RLE detection case now covers all 15 combinations of ordinary/post-split/target-size paths and tail lengths two through six.
- [O] `tests/fuzzer -i0 -s1`, `tests/zstreamtest -i100 -s1`, and three CLI determinism cases passed.
- [O] An ephemeral baseline/candidate matrix passed 68 cases each, covering equal/unequal tails, lengths zero through seven, true first blocks, flush-followed-by-data, and the block API.
- Test decision: update the existing RLE detection case because the silent size regression is not prevented by a type or static check.
- No test was weakened, skipped, or deleted; no new permanent test file or function was added.

## Publication-Blockers

Implementation is committed and pushed; shared combined-branch native, CLI, and fuzz-regression checks passed.
The user has not selected a publication target or approved an exact external draft.
Old CLI decoders were not executed; their first-block guard is preserved structurally.

## Next-Action

Summary: Select publication target
Action: Select the external contribution target before preparing an exact submission draft.
Done-When: The user selects a target and the exact current draft is ready for review.

## Pull-Request-Implementation

Branch: ratio/short-rle
Base: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`
Scope: RLE selection for identical two-to-six-byte follow-up frame blocks.
Commit: 597c90265e7e97b4194caf9cb3363f6cfa4597cf
Push: origin/ratio/short-rle
Checks:

- Actual CLI reproduction, the extended existing RLE case, 68 boundary cases, native streaming tests, and three CLI determinism cases passed.
- Shared combined-branch verification is recorded in `ISSUE-004.md/Shared Integration Verification`.
- The separate supplied-sequence API's tiny-raw path remains out of scope; no all-APIs correction is claimed.

## Performance-Evidence

The fixed-setting environment, corpus identities, and three-run protocol are recorded in `ISSUE-004.md/Performance-Evidence`.
Candidate CLI SHA-256: `7804f80fe42f10c8d2ba9e7afad8ad452ab38cfde94b1d89fbf1a049326720ad`.
All four corpus output sizes equaled baseline: source 738,793, documentation 632,922, Silesia subset 4,829,268, and random-256 4,194,410 bytes.
Median encode/decode MB/s were source 420.99/2160.8, documentation 303.17/2806.7, Silesia subset 267.28/1627.0, and random-256 7582.59/52113.9.
These short measurements support no general throughput improvement.

## Related upstream review

- https://github.com/facebook/zstd/pull/4851 at `e908b4bfdb0dc39cfd13491c144d09bcb492087c` fixes a distinct MT job-first-block bug with a guarded state assignment.
- The review found only a successful CLA check, not build/test CI evidence; the reported 104-to-83-byte result remains the author's measurement.
- The PR was reviewed but not adopted or published by this task.
