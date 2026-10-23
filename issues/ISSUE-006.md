# ISSUE-006 — Compression: encode short repeated follow-up blocks as RLE

State: Implementing
Authorized-Work: Pull-Request-Implementation
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-23
Updated: 2026-10-23
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [S]: The early no-compression return for inputs shorter than seven bytes bypasses the existing RLE selection.

## Reach-and-Impact

Reach [O]: A default CLI stream containing 131,072 copies of `a` followed by six copies of `b` ends in a six-byte raw block.
Impact [O]: The original frame is 32 bytes; an equivalent RLE tail makes it 27 bytes and is accepted by the unchanged decoder with the same checksum and output.
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

- Reproduce the 32-byte baseline and require the implemented encoder to produce the valid 27-byte result.
- Extend the existing `tests/fuzzer.c` RLE detection case for short follow-up blocks and roundtrips.
- Exercise ordinary, split, target-size, first-block, unequal-tail, and streaming continuation cases.

## Publication-Blockers

Encoder implementation, checks, publication target, exact draft, and publication approval remain pending.

## Next-Action

Summary: Verify scoped encoder correction
Action: Correct short repeated follow-up block selection and verify all frame dispatch paths.
Done-When: The isolated contribution is verified, committed, and pushed to the fork.

## Pull-Request-Implementation

Branch: ratio/short-rle
Base: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`
Scope: RLE selection for identical two-to-six-byte follow-up frame blocks.
Commit: Pending.
Push: Pending.
Checks:

- Baseline and candidate verification pending.
