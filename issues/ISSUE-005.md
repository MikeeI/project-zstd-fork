# ISSUE-005 — FSE: avoid basic encoding for two identical sequence codes

State: Implementing
Authorized-Work: Pull-Request-Implementation
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-23
Updated: 2026-10-24
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [S]: The monomorphic two-sequence exception prefers Basic without accounting for the final FSE state bits.

## Reach-and-Impact

Reach [S]: `ZSTD_selectEncodingType()` applies the exception before strategy-specific selection for literal-length, match-length, and offset codes.
Impact [O]: The public supplied-sequence witness shrank from 44 to 42 frame bytes, with sequence payload six to four bytes and flags `0x00` to `0x54`.
The one-sequence control remained 28 frame bytes; all decoded bytes matched.
Real-corpus size benefits remain unproven.

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

- [O] The controlled public-API reproducer passed against baseline and candidate with the predicted two-byte saving.
- [O] A normal-parser matrix of 1,000 deterministic inputs in ordinary mode and target-size 64 mode passed all 2,000 roundtrips.
- [O] Each mode's total output changed from 118,991 to 118,000 bytes; 1,982 individual frames improved, 18 were equal, and none grew.
- [O] `tests/fuzzer -i0 -s1` and four CLI literal/determinism cases passed.
- No permanent test was added, changed, weakened, skipped, or deleted for this finding.

## Publication-Blockers

Implementation is committed and pushed; shared combined-branch native, CLI, and fuzz-regression checks passed.
The user has not selected a publication target or approved an exact external draft.
The local bit-cost proof does not guarantee globally smaller streams: superblock repeat minimums, split decisions, and table history can change outcomes.

## Next-Action

Summary: Select publication target
Action: Select the external contribution target before preparing an exact submission draft.
Done-When: The user selects a target and the exact current draft is ready for review.

## Pull-Request-Implementation

Branch: ratio/fse-two-sequences
Base: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`
Scope: Restrict unconditional Basic preference to one sequence.
Commit: ba7585af86141cac6e589a6dfdac4db7e498afbe
Push: origin/ratio/fse-two-sequences
Checks:

- Public-API reproduction, 2,000 ordinary/target-size frames, native basic tests, and four CLI cases passed.
- Shared combined-branch verification is recorded in `ISSUE-004.md/Shared Integration Verification`.

## Bug-Reproduction

Use `ZSTD_compressSequences()` with explicit block delimiters and an empty delimiter after the sequences.
Set compression level 3, literal compression to `ZSTD_lcm_uncompressed`, repcode resolution to `ZSTD_ps_disable`, and sequence validation to 1.
For each sequence index `i`, append 13 literal bytes with values `32 + i * 13 + j`, where `j` runs from 0 through 12.
Append 12 matching bytes by repeatedly copying the byte eight positions behind the current output position.
Each sequence has `offset=8`, `litLength=13`, `matchLength=12`, and `rep=0`.
Run first with one sequence and then with two, leaving all other defaults unchanged.
Require `ZSTD_decompress()` to reconstruct every input byte.

| Sequences | Source bytes | Frame bytes, base/new | Payload bytes, base/new |
| --- | ---: | ---: | ---: |
| 1 | 25 | 28 / 28 | 3 / 3 |
| 2 | 50 | 44 / 42 | 6 / 4 |

Compile the ephemeral reproducer with `cc -std=c90 -O2 -Wall -Wextra -I<tree>/lib fse-repro.c <tree>/lib/libzstd.a -pthread`.
The temporary source was removed after verification; the complete input-generation and API contract above is retained for reproduction.

## Performance-Evidence

The fixed-setting environment, corpus identities, and three-run protocol are recorded in `ISSUE-004.md/Performance-Evidence`.
Candidate CLI SHA-256: `6180e8adfaf5eaeade176154460785cb6c893882f5f04462c97f5afc43b8252a`.
All four corpus output sizes equaled baseline: source 738,793, documentation 632,922, Silesia subset 4,829,268, and random-256 4,194,410 bytes.
Median encode/decode MB/s were source 416.51/2172.3, documentation 303.31/2837.8, Silesia subset 266.63/1644.5, and random-256 7550.49/52429.8.
These short measurements support no general throughput improvement.
