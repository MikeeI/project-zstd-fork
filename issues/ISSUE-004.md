# ISSUE-004 — Huffman: preserve compression opportunities for small alphabets

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

Root-Cause [S]: The largest-symbol-frequency rejection ignores alphabet cardinality and rejects balanced 128-symbol literals that admit seven-bit Huffman codes.

## Reach-and-Impact

Reach [O]: A 16,384-byte order-two de Bruijn sequence over bytes 0–127 reaches the ordinary CLI as match-free literals.
Impact [O]: The unchanged CLI emits a 16,397-byte raw frame.
Impact [O]: The isolated candidate emits 14,430 bytes for that CLI witness, saving 1,967 bytes.
Impact [O]: Fixed-setting benchmarks on source, documentation, and the official 12-file Silesia regression subset showed no size change.

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

Use existing cardinality information at all three rejection gates.
Keep the old shortcut through 128 input bytes and require at least two input bytes per distinct symbol on average before overriding the full-histogram rejection.
This density condition avoids the measured severe random-input slowdown at 129 and 160 bytes.
Final size checks, table validation, and raw fallback remain unchanged.

## Scope-and-Constraints

- Preserve lossless decoding, table lifecycle, format compatibility, and fast rejection of 256-symbol noise.
- Cover both Huffman callers and the earlier sampling gate.
- Exclude level tuning, exhaustive alternative encoding, and claims of free CPU savings.
- Authorization: The user's 2026-10-23 instruction explicitly requests implementation of this recommendation.

## Verification

- [O] The final direct-Huffman smoke covered 65,536-byte balanced alphabets with and without sampling, shifted/sparse alphabets, and disjoint samples.
- [O] Direct Huffman output changed from raw fallback to 57,419 bytes for low 128 symbols, or 57,390 bytes for shifted/sparse 128 symbols; every encoded case roundtripped.
- [O] Ordinary, post-split, and target-size public frame paths passed bytewise roundtrips; balanced 256-symbol and disjoint-sample controls remained raw.
- [O] `tests/fuzzer -i0 -s1` passed on the final candidate.
- [O] CLI cases `compression/compress-literals.sh`, `determinism/basic.sh`, `determinism/reuse.sh`, and `determinism/multithread.sh` passed without expectation changes.
- No permanent test was added or changed for this finding; temporary probes cover the new heuristic decision.

## Publication-Blockers

Implementation is committed and pushed; shared combined-branch native, CLI, and fuzz-regression checks passed.
The user has not selected a publication target or approved an exact external draft.
Representative workload prevalence, short-random-input CPU cost, and non-x86 coverage remain publication considerations.

## Next-Action

Summary: Select publication target
Action: Select the external contribution target and evaluate the documented random-input CPU trade-off before drafting.
Done-When: The user selects a target and the exact current draft is ready for review.

## Pull-Request-Implementation

Branch: ratio/huffman-alphabet
Base: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`
Scope: Alphabet-aware Huffman rejection in full histograms, sampling, and superblock metadata.
Commit: 9afff0c12c381e59204e85ef87038e858943cbcf
Push: origin/ratio/huffman-alphabet
Checks:

- Individual release build, native basic tests, direct entropy probes, and four CLI cases passed.
- Shared integration `ratio/internal-compression@64b5c89c6661e78fd90f65d680e96fb264dbb3da` passed native and corpus regression.

## Shared Integration Verification

The integration branch contains all three isolated corrections from ISSUE-004, ISSUE-005, and ISSUE-006.
It contains no fork-only tracking or context files.

- [O] `tests/fuzzer -i1000 -s1` passed all basic cases and 1,000 random iterations.
- [O] `tests/zstreamtest -i100 -s1` passed its basic cases and 101 reported random iterations.
- [O] The four CLI literal/determinism cases passed with unchanged expectations.
- [O] `make -s -C tests/fuzz regressiontest FUZZ_TARGETS='simple_round_trip stream_round_trip block_round_trip sequence_compression_api' REGRESSION_TARGET='simple_round_trip stream_round_trip block_round_trip sequence_compression_api' PYTHON=python3` passed the selected official seed corpora.
- [O] Seed counts were simple roundtrip 25,942, streaming roundtrip 24,223, block roundtrip 11,712, and sequence API 1,630: 63,507 successful corpus cases.
- Existing fuzz-harness Python/C compatibility warnings remain; no suppression or unrelated cleanup was introduced.

## Performance-Evidence

Environment: AMD Ryzen 9 5950X, Ubuntu 24.04 x86-64, GCC 13.3.0, repository release `-O3`, CPU affinity 2.
Baseline source: the recorded upstream revision; baseline CLI SHA-256 `281ec2c6f736504fed29c32e70aeba712726a6703563ddc98bba96a97c0f0b1e`.
Candidate CLI SHA-256: `372ed7ce33c0e19b4f58e406c2d8bc7800358b5fc2ee5a76d96841588851efde`.
Build: `make -s -C programs -j8 zstd-release`.
Corpus timing: `taskset -c 2 VARIANT/programs/zstd -q -b3 -e3 -i1 -S source.tar docs.tar silesia.tar random256.bin`.
Three independent runs per variant used reversed variant order in round two; builds and tests were stopped during timing.
The numbers below are medians of three benchmark-reported speeds, not confidence-bounded latency estimates.

| Input | Input bytes | Output bytes, both | Encode MB/s, base/new | Decode MB/s, base/new |
| --- | ---: | ---: | ---: | ---: |
| Source archive | 4014080 | 738793 | 417.55 / 419.00 | 2176.5 / 2128.1 |
| Documentation archive | 942080 | 632922 | 293.38 / 297.47 | 2812.5 / 2849.1 |
| Silesia regression subset | 12021760 | 4829268 | 264.07 / 268.49 | 1626.8 / 1645.1 |
| Synthetic random-256 | 4194304 | 4194410 | 7333.02 / 7365.16 | 51511.2 / 52029.7 |

Source and documentation archives use `git archive upstream/dev lib programs` and `git archive upstream/dev doc`.
Their SHA-256 values are `62eadc1f97ac850ff4d1889a51e39fb2a98870dc5cef9ba508f9a876ff01a0bc` and `1de5a390b15411281913239e62e52bcb30b1a4344c9ad61730fbca8f08573c43`.
Silesia came from https://github.com/facebook/zstd/releases/download/regression-data/silesia.tar.zst and contains twelve 1,000,000-byte files, not the full Silesia corpus.
Its decoded tar SHA-256 is `b198f118bedc503ffc887436611ccc1b418afc756e80f468e4a23c153952e763`.
Random input uses `tests/datagen -g4MB -P0 -s1`; SHA-256 is `bca8ee2d00550c3d84637857b25ce392a33ed0fda686d872e7b6c577c62472cd`.

Noise-boundary timing adds `-B<size>` to the same level-three command, with three alternating baseline/candidate runs.
Every tested size retained identical output bytes.

| Chunk bytes | Baseline encode MB/s | Candidate encode MB/s |
| --- | ---: | ---: |
| 64 | 135.96 | 140.54 |
| 128 | 204.08 | 205.85 |
| 129 | 202.72 | 199.09 |
| 160 | 217.67 | 213.70 |
| 192 | 220.38 | 212.23 |
| 224 | 209.48 | 202.58 |
| 256 | 254.11 | 220.19 |
| 512 | 348.05 | 342.01 |
| 131072 | 6710.44 | 6624.45 |

The 256-byte cell was unstable: baseline runs were 254.11/260.59/208.16 MB/s and candidate runs were 220.19/243.25/198.21 MB/s.
These measurements do not establish CPU neutrality; additional cardinality work remains visible on small random inputs.
The rejected unrestricted version slowed 64-byte noise from approximately 135 to 21 MB/s without saving bytes.
The rejected 128-byte-cutoff-only version merely moved that cliff: 129-byte noise measured 204.7 versus 35.6 MB/s.
The final density rule removes those large observed cliffs, not all overhead or all possible heuristic misses.
Context-size snapshots matched baseline in the targeted frame probes; they are not peak-memory measurements.

Process memory was measured separately with `/usr/bin/time -f '%M' taskset -c 2 VARIANT/programs/zstd -q --single-thread -3 -c silesia.tar`, discarding output.
Across three runs, baseline, Huffman, FSE, and the integrated build each reported 5,632 KiB peak RSS in every run.
The short-RLE build reported 5,376/5,632/5,632 KiB.
This coarse process metric includes CLI overhead; it does not establish identical peak codec allocations.
The integrated CLI's source, documentation, Silesia-subset, and random outputs all roundtripped bytewise through the unchanged baseline decoder.

The reviewed extDict proposal https://github.com/facebook/zstd/pull/4764 at `7bf1756b9ac4635a6ec3a991dde5c605f2b293b2` was not adopted.
It already contains the improved candidate-length comparison, but bundles a broad pipeline rewrite and reported dictionary-size regressions.
Only CLA check evidence was visible; the maintainer's benchmark-reproduction concern remained unresolved.
This separate parser proposal is not part of the alphabet-aware correction or its measured results.
