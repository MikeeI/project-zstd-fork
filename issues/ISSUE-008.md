# ISSUE-008 — Optimal parser: reuse invariant code-price components

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: https://github.com/facebook/zstd/pull/4732
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-23
Updated: 2026-10-23
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [S]: Match-length loops recompute offset and frequency-derived code prices while the relevant statistics remain unchanged.
This is review item P2, retained as VALID for reuse of existing upstream work, not a new discovery.

## Reach-and-Impact

Reach [O]: The optimal parser is a major compression hotspot on source text and independent JSON messages.
Impact [S]: Hoisting offset costs and retaining per-code frequency terms can eliminate repeated work without changing match prices.
The whole parser profile share is not removable work, and upstream-reported speedups have not been reproduced locally.

## Evidence

- [S] `lib/compress/zstd_opt.c:1184` and `:1316` call `ZSTD_getMatchPrice()` inside candidate-length loops.
- [S] `ZSTD_updateStats()` is called at `lib/compress/zstd_opt.c:1420` after path selection; frequency terms are invariant within the preceding length loops.
- [O] The local GCC `-O3` disassembly repeats offset-frequency loads, `bsr`, and shifts inside `ZSTD_compressBlock_opt2` around addresses `0x87a42-0x87a73`.
- [S] PR #4732 adds per-code arrays and preserves intentional U32 wraparound; changing arithmetic types would require an equivalence proof.

## Prior-Art

Coverage: GitHub PR search for `"match" "price"`, complete PR description, and both changed-file diffs; checked=2026-10-23.
Gaps: Local patch validation and architecture coverage remain incomplete.

- https://github.com/facebook/zstd/pull/4732 — Duplicate root cause; existing open implementation by matevz-kovacic.

Contribution fit: Independently validate the existing PR and, if authorized, contribute new evidence there rather than open a competing implementation.
External-Reference identifies that existing contribution, not a publication performed by this session.

## Proposed-Change

Use PR #4732 as the candidate basis: hoist offset prices and cache per-code frequency terms with exact U32 arithmetic.
Refresh touched entries after statistics changes and rebuild after rescaling or initialization.

## Scope-and-Constraints

- Preserve parse choices, compressed bytes, dictionary initialization, and context-local ownership.
- Account for 121 U32 entries, totaling 484 bytes of added price-array storage per `optState_t`, plus maintenance work.
- Exclude duplicate upstream publication, parameter tuning, and blanket gains across strategies or architectures.

## Performance-Evidence

Environment and binary provenance: ISSUE-010/Performance-Evidence.
Source corpus: `git archive --format=tar --output=/tmp/zstd-core-perf-UmzQCg/source.tar upstream/dev lib programs`.
Source corpus size=4,014,080 bytes; SHA-256=`62eadc1f97ac850ff4d1889a51e39fb2a98870dc5cef9ba508f9a876ff01a0bc`.
Command: `perf record -F 999 -e cycles:u --call-graph dwarf,8192 -N -o /tmp/zstd-core-perf-UmzQCg/level19.data programs/zstd -b19 -i3 /tmp/zstd-core-perf-UmzQCg/source.tar`.
Result [O]: `ZSTD_compressBlock_opt2` accounts for 28.83% of the combined compression/decompression profile.
JSON command: `perf record -F 999 -e cycles:u --call-graph dwarf,8192 -N -o /tmp/zstd-core-perf-UmzQCg/github19.data programs/zstd -q -b19 -i3 /tmp/zstd-core-perf-UmzQCg/github/*.json`.
JSON result [O]: The same parser accounts for 33.42%; neither share isolates the removable arithmetic.
External report: PR #4732 reports approximately 6-12% higher level-19 compression throughput on full Silesia using Ryzen 9700X and i5-13400 with GCC/Clang.
Those are the author's measurements, not local results; the downloaded 12 MB Silesia regression subset is not that full corpus.
Each local workload had one profiling invocation with repeated timed iterations, not an alternating candidate comparison.

## Verification

- Alternate baseline and existing-PR candidate runs with identical data and parameters; compare throughput, instructions, and byte-identical output.
- Cover dictionaries, rescaling, initialization, different compilers, and price-array maintenance costs.

## Publication-Blockers

Work-mode selection, local candidate validation, contribution value beyond the existing PR, target, exact draft, and publication approval remain unresolved.

## Next-Action

Summary: Select existing PR validation scope
Action: Obtain the user's work-mode and scope selection for independent validation of PR #4732.
Done-When: The permitted validation effects and work mode are explicitly recorded without claiming a new upstream implementation.
