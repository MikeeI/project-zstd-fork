# ISSUE-005 — FSE: avoid basic encoding for two identical sequence codes

State: Implementing
Authorized-Work: Pull-Request-Implementation
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-23
Updated: 2026-10-23
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [S]: The monomorphic two-sequence exception prefers Basic without accounting for the final FSE state bits.

## Reach-and-Impact

Reach [S]: `ZSTD_selectEncodingType()` applies the exception before strategy-specific selection for literal-length, match-length, and offset codes.
Impact [S]: Two sequences with literal length 13, match length 12, and raw offset 8 require six sequence-stream bytes with Basic versus four bytes including RLE tables.
Actual API reproduction and corpus frequency remain unmeasured.

## Evidence

- [S] `lib/compress/zstd_compress_sequences.c:167-178` prefers Basic when `nbSeq <= 2`.
- [S] `lib/compress/zstd_compress_sequences.c:373-378` flushes the final states.
- [S] `lib/common/zstd_internal.h:126-164` defines the default distributions and table logs.
- [S] The witness uses LL code 13, ML code 9, and OF code 3, each with default normalized count one.

## Prior-Art

Coverage: Current source, local history, and targeted open/closed GitHub issue/PR searches; checked=2026-10-23.
Gaps: Exhaustive discussions and naturally generated sequence frequency remain unchecked.

- https://github.com/facebook/zstd/pull/1144 — Related; existing approximate FSE table selection, not this monomorphic exception.

Contribution fit: A minimal encoder selection correction after an executed public-API reproduction.

## Proposed-Change

Limit the unconditional Basic preference to one sequence and retain existing RLE handling for two identical codes.

## Scope-and-Constraints

- Preserve one-sequence behavior, table-state contracts, all callers, and public APIs.
- Exclude broader FSE cost-model changes and level tuning.
- Authorization: The user's 2026-10-23 instruction explicitly requests implementation of this recommendation.

## Verification

- Run the supplied-sequence witness before and after the isolated correction, preserving literals and disabling repcode rewriting only to control the experiment.
- Require the predicted two-byte difference and bytewise roundtrip.
- Cover one/two/three sequences, existing fuzzer cases, and final-frame sizes rather than estimated costs alone.

## Publication-Blockers

Executed reproduction, implementation, checks, publication target, exact draft, and publication approval remain pending.

## Next-Action

Summary: Verify scoped encoder correction
Action: Reproduce and correct the monomorphic two-sequence decision.
Done-When: The isolated contribution is verified, committed, and pushed to the fork.

## Pull-Request-Implementation

Branch: ratio/fse-two-sequences
Base: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`
Scope: Restrict unconditional Basic preference to one sequence.
Commit: Pending.
Push: Pending.
Checks:

- Baseline and candidate verification pending.
