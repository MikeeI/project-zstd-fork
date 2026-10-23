# ISSUE-009 — Huffman: evaluate omitted minimum-depth candidates

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-23
Updated: 2026-10-23
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [S]: `HUF_minTableLog()` uses `highbit(cardinality) + 1`, omitting a valid depth bound for power-of-two alphabets.
The larger bound can already yield a shallower tree, so omission does not cause a loss for every such alphabet.
This is review item P3, classified CHANGED because the correction must preserve the existing search and tie decisions.
It is distinct from ISSUE-004's incompressibility rejection before tree-depth selection.

## Reach-and-Impact

Reach [S]: Optimal-depth literal coding reaches `HUF_optimalTableLog()` through ordinary Huffman compression and superblock metadata construction.
Impact [O]: An isolated 97-byte literal input produces a 45-byte Huffman result; an omitted depth-three candidate produces 43 bytes.
Both results decode exactly, but this is not a complete zstd-frame improvement or a demonstrated real-corpus benefit.

## Evidence

- [S] `lib/compress/huf_compress.c:1269-1272` computes the lower bound; `:1295-1325` probes depths with early termination.
- [S] `lib/compress/huf_compress.c:1318` stops when `newSize > optSize + 1`; merely lowering the starting depth does not establish preservation of the old search trajectory.
- [S] `lib/compress/zstd_compress_literals.c:166` enables optimal depth by strategy; `lib/compress/zstd_compress.c:3729` is the other table-depth caller.
- [O] Original reproduction output: `input=97 current=45 candidate=43 saved=2 current_roundtrip=ok candidate_roundtrip=ok`.
- [O] A separate review probe compared the naive lower-bound search across 120,000 generated distributions and printed `regression_witnesses=0`; no regression was observed, and monotonicity was not proved.

## Prior-Art

Coverage: GitHub issue/PR searches for `"HUF_minTableLog"`, PR search for `"HUF_optimalTableLog"`, and the original optimal-depth discussion; checked=2026-10-23.
Gaps: Real literal-histogram frequency, complete-frame behavior, and exhaustive discussions remain unchecked.

- https://github.com/facebook/zstd/pull/3285 — Related; introduced optimal-depth search and documents local versus later table-reuse effects.
- ISSUE-004 — Distinct; alphabet-aware incompressibility rejection, not the candidate-depth lower bound.

Contribution fit: A small ratio correction only after bounding added CPU cost and validating complete frames.

## Proposed-Change

Preserve the original depth search, then separately evaluate the omitted minimum-depth candidate for eligible power-of-two alphabets.
Select it only when its header-plus-payload cost is strictly lower than the original winner; retain old ties and one-symbol handling.
Do not present the revised proposal as a proven one-line bound change or claim that a naive change has an observed regression.

## Scope-and-Constraints

- Preserve lossless decoding, format compatibility, raw/RLE fallback, and valid workspace/table lifetimes.
- Separate actual tree depth from the requested maximum depth.
- Account for the extra candidate's CPU cost and possible effects on later Huffman-table reuse.
- Exclude alphabet-rejection changes owned by ISSUE-004 and generic compression-ratio promises.

## Bug-Reproduction

Environment: Unchanged upstream objects from the GCC 13.3.0 release build on Ryzen 9 5950X; provenance in ISSUE-010.
Input: Concatenate byte values `0,2,4,6,8,10,12,14` with respective occurrence counts `5,8,13,15,13,15,15,13`.
Baseline: `HUF_compress1X_repeat()` with `maxSymbolValue=14`, `HUF_TABLELOG_DEFAULT`, `HUF_repeat_none`, and `HUF_flags_optimalDepth` returns 45 bytes.
Candidate: Build the same histogram using `HUF_buildCTable_wksp()` with maximum depth 3, serialize with `HUF_writeCTable_wksp()`, and encode with `HUF_compress1X_usingCTable()`; total=43 bytes.
Decode both with `HUF_decompress1X_DCtx_wksp()` and compare all 97 bytes with the input; both comparisons passed.
The temporary probes were removed; no permanent test or encoder implementation was added.

## Verification

- Recreate the 97-byte witness and power-of-two boundary cases; require exact roundtrips and preservation of original search/tie behavior.
- Compare complete zstd frames, later table reuse, and paired encode timings at identical settings before claiming a practical gain.

## Publication-Blockers

Implementation authorization, complete-frame reproduction, real-corpus prevalence, CPU-cost validation, target, draft, and publication approval remain unresolved.

## Next-Action

Summary: Select minimum-depth validation scope
Action: Obtain the user's work-mode and implementation boundary for the separately evaluated Huffman candidate.
Done-When: Authorized effects and the original-search-preserving correction are explicitly selected and recorded.
