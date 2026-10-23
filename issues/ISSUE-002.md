# ISSUE-002 — CLI: avoid zero-distance seeks in automatic sparse decompression

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

Root-Cause [S]: `AIO_fwriteSparse()` seeks before writing nonzero data even when `storedSkips == 0` and the automatic sparse mode already restricts output to regular files.
Review mapping: P2, materially narrowed by the subsequent review.

## Reach-and-Impact

Reach [S]: CLI decompression through `AIO_fwriteSparse()`, including synchronous execution of its jobs with `--no-asyncio`.
Impact [O]: Decompressing 64 MiB random data produced 2,051 `lseek` calls and 2,052 `write` calls with sparse mode, versus zero and 1,024 with `--no-sparse`.
Impact [S]: Avoiding unnecessary stdio seeks can also avoid the buffer flushes those calls trigger.
The control disables all sparse handling, not just zero-distance seeks; it does not measure the proposed patch.

## Evidence

- [S] `programs/fileio_asyncio.c:76` and `:99` seek unconditionally before their nonzero writes.
- [S] `programs/fileio_types.h:37` distinguishes disabled=0, automatic=1, and forced=2 sparse modes.
- [S] `programs/fileio.c:856-863,917-925` disables automatic sparse handling for stdout and nonregular destinations.
- [S] `programs/zstdcli.c:1010` selects forced mode for `--sparse`.
- [S] `programs/zstdcli.c:1640` disables sparse mode before compression; the similar synchronous compression writer is not part of this reachable CLI optimization.
- [O] With `pipefail`, `programs/zstd -q -c lib/zstd.h | programs/zstd -q -d --sparse -c | wc -c` returned exit 92, `Sparse skip error`, and zero output bytes.
- [O] `programs/zstd -q -c lib/zstd.h | programs/zstd -q -d -c | cmp - lib/zstd.h` succeeded.

## Prior-Art

Coverage: GitHub issues `repo:facebook/zstd sparse seek`, PRs `repo:facebook/zstd sparse`, and the complete issue #615 discussion; checked=2026-10-23.
Gaps: PR search was limited to 20 results; discussions, releases, and every related patch were not exhaustively reviewed.

- Related: Historical sparse-output slowdown and an unsuccessful seek-reduction experiment; not proof of a current Linux gain: https://github.com/facebook/zstd/issues/615
- Related candidate: Regular-file detection after opening the destination; identified by title, not fully reviewed: https://github.com/facebook/zstd/pull/4353

Contribution fit: Unselected; prior discussion precludes presenting seek reduction as a novel or demonstrated large speedup.

## Proposed-Change

At the two `AIO_fwriteSparse()` seek sites, skip positioning only when `storedSkips == 0 && prefs->sparseFileSupport == 1`.
Retain existing positioning for forced sparse mode and every actual gap.
Clarify the compression-path exclusion in the AGENTS.md navigation map when that related documentation update is authorized.

## Scope-and-Constraints

- Preserve sparse holes, trailing-zero materialization, byte content, write errors, and actual-seek errors.
- Preserve the observed forced-sparse pipe failure rather than silently changing its behavior.
- Preserve read overlap, job ordering, buffer ownership, draining, and cleanup; do not redesign the pools.
- Exclude `fileio.c::FIO_sparseWrite()` from this CLI performance patch because compression disables its sparse branch.
- Cost: Two guarded seek sites; no new seekability abstraction or dependency.
- Test decision: none; use focused smoke comparisons and existing sparse scenarios, not new permanent tests.

## Performance-Evidence

Environment: Ubuntu 24.04 x86-64, Linux 6.8.0-107-generic, Ryzen 9 5950X, GCC 13.3.0, release `-O3`, strace 6.8.
Build: `make -C programs -j8 zstd-release`; local CLI reports v1.6.0.
Binary SHA-256: `8ce9b1f28c8e3b4703fa7a83f8a91d44c86b313e3af0e14f412535c5ee044066`.
Input: 64 MiB from `/dev/urandom`, compressed using `programs/zstd -q --single-thread`.
Trace filter: `strace -f -c -e trace=lseek,write` around decompression to a real file with `-q -d -f --no-asyncio`, compared with additional `--no-sparse`.
Untraced control: `hyperfine --warmup 2 --runs 9` measured 71.2 ± 1.2 ms with sparse mode and 69.5 ± 1.9 ms without it; values are mean ± standard deviation.
These small, overlapping timings do not establish a meaningful speedup, and the session did not guarantee an isolated host.
A detailed trace showed glibc translating zero-distance stdio seeks into absolute `lseek(..., SEEK_SET)` calls.
The original random input, output, and raw traces were removed; no input hash was retained, so the exact random corpus cannot be reconstructed.

## Verification

- After authorization, compare the actual patch against unchanged sparse mode with the same corpus, destination filesystem, and async setting.
- Count syscalls separately from repeated untraced timing runs; do not substitute `--no-sparse` for the baseline.
- Compare byte content and allocated blocks for dense, zero-rich, trailing-zero, and odd-sized outputs.
- Confirm the forced-sparse pipe reproduction still exits 92 and ordinary stdout decompression still succeeds.
- Existing sparse and odd-sized scenarios are in `tests/playTests.sh:990-1002`; no source patch or post-fix smoke has run.

## Publication-Blockers

Implementation authorization and publication target are unselected.
No patch comparison, full compatibility verification, completed prior-art review, or approved publication draft exists.
The superseded four-site proposal must not be reused.

## Next-Action

Summary: Select scoped implementation authorization
Action: Obtain the user's Authorized-Work selection for the two automatic sparse-decompression seek sites.
Done-When: The selection is recorded here and projected into ISSUES.md, preserving the forced-mode exclusion.
