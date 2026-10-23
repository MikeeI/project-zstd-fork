# ISSUE-003 — Compression level comparison (not a repository issue)

State: Archived
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Other
Created: 2026-10-23
Updated: 2026-10-23
Source: `upstream/dev@49cf51799ea051457f57243527aa2058f999f005`

## Root-Cause

Root-Cause [O]: The initial investigation treated low implementation effort as requiring a CPU-free compression gain and omitted a comparison of existing settings.
Review mapping: A1, an added reporting and tuning decision, not a zstd source defect.
This entry was misclassified as an open finding; it is archived to preserve its ID and measured evidence.

## Reach-and-Impact

Reach [O]: The user's request explicitly included easily obtainable compression improvements.
Impact [O]: On the measured C source file, level 6 produced 72,252 bytes versus 80,686 bytes at level 3, approximately 10.5% smaller.
Impact [O]: Level 6 required approximately three times the compression time in the measured runs.
No evidence establishes the same trade-off for the user's production workload or its memory limits.

## Evidence

- [O] Three executions of `programs/zstd -b3 -e6 -i1 lib/compress/zstd_compress.c` measured the existing levels without source changes.
- [O] Input size was 368,588 bytes; output sizes were identical across all three runs for each level.
- [O] `programs/zstd -q -6 -c lib/compress/zstd_compress.c | programs/zstd -q -d -c | cmp - lib/compress/zstd_compress.c` succeeded.
- [S] `lib/compress/clevels.h:25-50` already defines the relevant compression presets.
- [S] `lib/compress/zstd_compress.c::ZSTD_selectBlockCompressor()` dispatches existing strategies; no new codec implementation is needed to compare levels.

## Prior-Art

Coverage: Existing level presets and direct local benchmark output; checked=2026-10-23.
Gaps: No external novelty search or representative application-corpus evaluation was performed for this reporting item.
Contribution fit: Internal decision support only; existing compression levels and one corpus comparison do not justify an upstream bug report.

## Proposed-Change

No repository correction is proposed.
The original proposal was to report the measured level-3/6 size-versus-CPU trade-off without changing defaults.
That is usage guidance, not an issue; reusable investigation rules belong in `AGENTS.md`.
Archiving does not invalidate the recorded measurements or authorize adopting a setting.

## Scope-and-Constraints

- Preserve lossless decoding, existing presets, source code, and current runtime defaults.
- Exclude claims of free compression gain, universal speedup, upstream novelty, or measured memory improvement.
- Cost: Existing CLI options only; the measured compression-time penalty remains explicit.
- Test decision: none; an existing benchmark and byte-identical roundtrip supplied the evidence.

## Performance-Evidence

Environment: Ubuntu 24.04 x86-64, Ryzen 9 5950X, GCC 13.3.0, release `-O3`, local zstd v1.6.0.
Build: `make -C programs -j8 zstd-release`.
Binary SHA-256: `8ce9b1f28c8e3b4703fa7a83f8a91d44c86b313e3af0e14f412535c5ee044066`.
Corpus: `lib/compress/zstd_compress.c` from the recorded upstream revision, 368,588 bytes.
Command: `programs/zstd -b3 -e6 -i1 lib/compress/zstd_compress.c`, repeated three times.

| Level | Output bytes | Compression MB/s, runs 1/2/3 | Decompression MB/s, runs 1/2/3 |
| --- | ---: | --- | --- |
| 3 | 80686 | 385.0 / 381.2 / 393.7 | 1997.1 / 1912.5 / 2012.8 |
| 4 | 80580 | 372.0 / 367.8 / 383.2 | 1995.1 / 1961.3 / 1989.6 |
| 5 | 74839 | 175.6 / 173.7 / 175.4 | 2015.0 / 1986.2 / 1998.5 |
| 6 | 72252 | 130.3 / 128.0 / 129.2 | 2141.8 / 2112.4 / 2102.8 |

These are the benchmark's reported per-run speeds, not an end-to-end filesystem benchmark or latency distribution.
Short runs on one source file support this comparison only; memory usage and other architectures were not measured.

## Verification

- Completed: All three runs agree on output sizes; the level-6 streaming roundtrip is byte-identical.
- Before adopting a setting, repeat the existing comparison on representative user data and check the workload's throughput and memory requirements.
- Keep benchmark inputs, revision, compiler flags, output sizes, and repeated-run speeds with any resulting recommendation.

## Publication-Blockers

No zstd source defect or externally useful new contribution has been established.
Authorized-Work and Publication-Target remain unselected; this request authorizes documentation only.
A production recommendation lacks representative user data and a size/speed/memory objective.

## Next-Action

Summary: —
Action: None.
Done-When: None.

## Archive

Archive-Reason: Finding-Invalidated
Detail: None.
Evidence: The Root-Cause and Prior-Art sections identify a reporting omission and existing parameter trade-off, not a repository defect.
The user explicitly requested genuine issues rather than tips; no bounded repository correction was established.
Checked: 2026-10-23
