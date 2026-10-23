# ISSUE-004 — Huffman: preserve compression opportunities for small alphabets

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

Root-Cause [S]: The largest-symbol-frequency rejection ignores alphabet cardinality and rejects balanced 128-symbol literals that admit seven-bit Huffman codes.

## Reach-and-Impact

Reach [O]: A 16,384-byte order-two de Bruijn sequence over bytes 0–127 reaches the ordinary CLI as match-free literals.
Impact [O]: The unchanged CLI emits a 16,397-byte raw frame.
Impact [S]: Existing Huffman representations permit a 14,430-byte frame for that witness; candidate output and real-corpus impact remain unmeasured.

## Evidence

- [S] `lib/compress/huf_compress.c:1384-1387` rejects after constructing the complete histogram.
- [S] `lib/compress/huf_compress.c:1368-1381` has an earlier sampling rejection for inputs of at least 40,960 bytes.
- [S] `lib/compress/zstd_compress.c:3714-3718` repeats the full-histogram rejection in the superblock metadata path.
- [O] `LC_ALL=C awk 'BEGIN { for(i=0;i<128;i++) { printf "%c",i; for(j=i+1;j<128;j++) printf "%c%c",i,j; } }' | programs/zstd -q -c | wc -c` returns `16397` with Bash pipefail enabled.
- [O] The input and decoded output SHA-256 are `458a93f8aa777c8dab7621702d64e379f108c17f541198357514dc1b7d8554fd`.

## Prior-Art

Coverage: Current upstream source, local history, and targeted open/closed GitHub issue/PR searches; checked=2026-10-23.
Gaps: Exhaustive discussions and workload prevalence remain unchecked.

- https://github.com/facebook/zstd/pull/1202 — Related; protects small random inputs from expensive unsuccessful Huffman work.
- https://github.com/facebook/zstd/pull/2717 — Related; sampling avoids full histograms for incompressible data.

Contribution fit: A bounded encoder correction, contingent on preserving random-data performance.

## Proposed-Change

Make the full-histogram and sampling rejection alphabet-aware while retaining final size checks and raw fallback.
Use existing histogram information and avoid new APIs or search parameters.

## Scope-and-Constraints

- Preserve lossless decoding, table lifecycle, format compatibility, and fast rejection of 256-symbol noise.
- Cover both Huffman callers and the earlier sampling gate.
- Exclude level tuning, exhaustive alternative encoding, and claims of free CPU savings.
- Authorization: The user's 2026-10-23 instruction explicitly requests implementation of this recommendation.

## Verification

- Reproduce the unchanged witness, then compare candidate bytes and roundtrip at identical settings.
- Exercise sparse alphabets, large sampled inputs, mixed samples, random controls, and the superblock path.
- Compare repeated encode/decode timings and memory at fixed settings on synthetic controls and real available corpora.

## Publication-Blockers

Implementation, verification, publication target, exact draft, and publication approval remain pending.

## Next-Action

Summary: Verify scoped encoder correction
Action: Implement and verify alphabet-aware rejection against the baseline and random-data controls.
Done-When: The isolated contribution is verified, committed, and pushed to the fork.

## Pull-Request-Implementation

Branch: ratio/huffman-alphabet
Base: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`
Scope: Alphabet-aware Huffman rejection in full histograms, sampling, and superblock metadata.
Commit: Pending.
Push: Pending.
Checks:

- Baseline and candidate verification pending.
