# ISSUE-010 — FSE: precompute standard sequence compression tables

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-23
Updated: 2026-10-23
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [S]: `ZSTD_buildCTable()` rebuilds constant default FSE tables whenever sequence encoding selects `set_basic`.
This is review item P1, retained as VALID; it is distinct from ISSUE-005's encoding-selection decision.

## Reach-and-Impact

Reach [O]: Independent small messages repeatedly reach the standard-table builder through `ZSTD_buildSequencesStatistics()`.
Impact [O]: The builder accounts for 8.27% of the combined compression/decompression CPU profile on 500 GitHub JSON files.
No candidate implementation, net throughput improvement, or general workload prevalence has been measured.

## Evidence

- [S] `lib/compress/zstd_compress_sequences.c:266` calls `FSE_buildCTable_wksp()` with default normalization, symbol limit, and table log.
- [S] `lib/common/fse.h:423-435` consumes a native-layout header, U16 state table, and symbol transformations; raw host byte dumps are not portable constants.
- [O] All 516 sampled `ZSTD_buildCTable` caller frames in the JSON profile resolve to the basic-table call at `+0x14e`, not the dynamic-table call at `+0x180`.

## Prior-Art

Coverage: Local records and GitHub issue/PR searches for `"default" "CTable"` and PRs for `"precomputed" "FSE"`; checked=2026-10-23.
Gaps: Exhaustive discussions, releases, and differently named implementations remain unchecked.

- https://github.com/facebook/zstd/pull/1144 — Related; table-selection costs, not elimination of constant-table construction.
- ISSUE-005 — Distinct; changes when Basic is selected rather than how an already selected Basic table is obtained.

Contribution fit: A bounded CPU optimization after authorization, portable representation review, and paired benchmarks.

## Proposed-Change

Replace only `set_basic` construction with correctly aligned, endian-correct `static const` tables copied into existing state.
Preserve dynamic, repeated, and RLE table paths without a mutable global cache or initialization lifecycle.

## Scope-and-Constraints

- Preserve compressed bytes, context ownership, dictionaries, table transitions, and supported architectures.
- Account for constant-data footprint and residual copying cost.
- Exclude parameter tuning, encoding-selection changes, and interpreting a hotspot percentage as a speedup.

## Performance-Evidence

Environment: Ryzen 9 5950X, Ubuntu 24.04, Linux 6.8.0-107-generic, GCC 13.3.0, release `-O3`, perf 6.8.12.
Build: `make -C programs -j8 zstd-release`; binary SHA-256 `8ce9b1f28c8e3b4703fa7a83f8a91d44c86b313e3af0e14f412535c5ee044066`.
Corpus: https://github.com/facebook/zstd/releases/download/regression-data/github.tar.zst contains 500 JSON files totaling 407,963 bytes.
Decompressed archive SHA-256: `e21aa81d1cd9b5c0a66658e58097677f8f2ea573b6e23b624ac8f614590a4d8e`.
Command: `perf record -F 999 -e cycles:u --call-graph dwarf,8192 -N -o /tmp/zstd-core-perf-UmzQCg/github.data programs/zstd -b1 -i3 /tmp/zstd-core-perf-UmzQCg/github/*.json`.
Result [O]: 8.27% standard sequence-table construction, 1.44% Huffman-related FSE construction, 9.71% total FSE construction; lost samples=0.
Control [O]: `programs/zstd -b1 --split=256 -i3 /tmp/zstd-core-perf-UmzQCg/source.tar` produced 9.28% standard-table construction in its profile.
Limits: Each workload had one profiling invocation with repeated timed iterations; the denominator combines compression and decompression, not production CPU or expected savings.
Profile paths are temporary research artifacts, not durable fixtures.

## Verification

- Alternate baseline/candidate runs on the same JSON files at identical settings; compare throughput and byte-identical output.
- Check dictionary use, table transitions, constant-data size, alignment, and endian-correct representation.

## Publication-Blockers

Implementation authorization, candidate measurements, portability verification, completed prior-art research, target, draft, and publication approval remain unresolved.

## Next-Action

Summary: Select scoped implementation authorization
Action: Obtain the user's work-mode selection for the standard-table optimization before creating a candidate build.
Done-When: Authorized-Work and any implementation boundary are explicitly selected and recorded.
