# ISSUE-007 — Compression: include Raw/RLE literal headers in block-size estimates

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

Root-Cause [S]: The Raw and RLE branches of both literal-size estimators omit the one-to-three-byte literal header.
The actual literal writers include that header; the ordinary estimator explicitly promises header plus content.
Confidence is high for the accounting mismatch, not for a resulting compression improvement.

## Reach-and-Impact

Reach [S]: The ordinary estimator feeds post-sequence split decisions through `ZSTD_deriveBlockSplitsHelper()`.
Reach [S]: The separate superblock estimator feeds target-size subblock planning and its early incompressibility decision.
Impact [S]: Each affected literal-section estimate is short by `1 + (litSize > 31) + (litSize > 4095)` bytes.
For 64 literals, the Raw estimate is 64 versus 66 serialized bytes; the RLE estimate is one versus three bytes.
These are source-derived section sizes, not an executed end-to-end reproduction.
No changed split decision, smaller frame, CPU saving, or workload prevalence has been measured.

## Evidence

- [S] `lib/compress/zstd_compress.c:3857-3878` documents header-plus-content estimation but returns only `litSize` or one.
- [S] `lib/compress/zstd_compress_superblock.c:307-324` has the same Raw/RLE omission in a separate estimator.
- [S] `lib/compress/zstd_compress_literals.c:39-65,81-106` computes and writes the missing literal headers.
- [S] `lib/compress/zstd_compress.c:3951-3994` adds sequence costs and the block header, not the omitted literal header.
- [S] `lib/compress/zstd_compress.c:4243-4259` compares the whole estimate with the sum of the two halves.
- [S] `lib/compress/zstd_compress_superblock.c:397-415,513-532` consumes the second estimator for sizing and fallback.
- [S] `lib/compress/zstd_compress.c:269-272,4209-4239,4665-4671` defines post-splitter activation, the 300-sequence gate, and target-size dispatch.
- [S] `lib/compress/zstd_compress.c:3691-3712` selects Raw for sufficiently small literal ranges before testing RLE.
- [O] `git ls-remote upstream refs/heads/dev` matched the recorded revision on 2026-10-23.

## Prior-Art

Coverage: Current source and callers; local ledger; GitHub PR searches `ZSTD_estimateBlockSize_literal` and `ZSTD_estimateSubBlockSize_literal`; issue search `splitter literal header`; checked=2026-10-23.
Both symbol-specific PR searches returned no matches; this does not establish novelty.
Gaps: No exhaustive discussion, release, historical-diff, or alternate-term search; no public reproduction.

- Related: Splitter architecture and expansion safeguards, not evidence of this accounting correction: https://github.com/facebook/zstd/pull/4136
- Distinct: Parser/repcode corner case and splitting experiments, not the omitted literal headers: https://github.com/facebook/zstd/issues/2460
- Distinct local records: ISSUE-004 concerns Huffman rejection; ISSUE-005 concerns monomorphic FSE selection; ISSUE-006 concerns short-block RLE dispatch.

Contribution fit: A bounded estimate correction only if controlled experiments establish useful behavior without unacceptable regressions.

## Proposed-Change

Include the actual Raw/RLE literal-header width in both corresponding estimator branches.
Evaluate the ordinary post-splitter first, then the separate target-size path; do not combine their measurements.
Do not change compressed-literal estimates, parser search, split thresholds, or global compression presets.

## Scope-and-Constraints

- Preserve bitstream validity, decoder compatibility, expansion safeguards, repcodes, and entropy-table lifecycle.
- Preserve the distinction between literal-section RLE and an entire RLE frame block.
- No new API, additional match search, diagnostic framework, or claimed constant-budget gain is justified by this record.
- The proposed arithmetic needs no additional tables; actual CPU and peak-memory effects remain unmeasured.
- Test decision: none for this documentation task; any implementation needs separately selected authorization.

## Verification

- After explicit experiment authorization, start with the smallest isolated A/B size comparison before extensive instrumentation.
- Confirm post-splitter activation, no target-size diversion, eligible sequence counts, and the actual Raw/RLE metadata branches.
- Probe literal counts around 31/32 and 4095/4096 in reachable states; file length alone does not establish these conditions.
- Distinguish an unvisited path from unchanged decisions, unchanged frame bytes, regressions, and genuine gains.
- If diagnosis is needed, capture whole/left/right estimates and metadata before shared scratch is overwritten.
- Require instrumented and uninstrumented baselines to emit identical frames; do not invoke extra entropy builds for passive logging.
- Validate complete frames and subsequent table reuse, not just a locally smaller estimated block.
- Measure release CPU, separate peak allocations, and decoder behavior against the fixed-budget protocol in `AGENTS.md`.

## Publication-Blockers

Work authorization and publication target are unselected.
No instrumented witness, experimental patch, before/after frame comparison, resource measurements, complete prior-art review, or approved publication draft exists.
An accounting mismatch alone does not establish a user-visible compression regression.

## Next-Action

Summary: Establish affected sizing decisions
Action: Establish an end-to-end witness for the omitted-header accounting after explicit authorization for any required instrumentation or experimental patch.
Done-When: Path coverage and a complete-frame comparison establish a useful correction or a documented negative result for the tested scope.
