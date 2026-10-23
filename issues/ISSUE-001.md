# ISSUE-001 — CLI: skip the unused batch size prepass outside patch mode

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

Root-Cause [S]: `FIO_compressMultipleFilenames()` computes the largest input size even when patch mode is disabled and its consumer ignores the result.
Review mapping: P1, retained by the subsequent review.

## Reach-and-Impact

Reach [S]: CLI compression dispatched to `FIO_compressMultipleFilenames()`, including ordinary multi-file compression.
Impact [S]: One avoidable filesystem size query per input precedes compression outside patch mode.
Impact [O]: A run over 104 archived library files performed 104 queries from this prepass.
No patched timing comparison establishes an end-to-end speedup; cached metadata may make the latency benefit small.

## Evidence

- [S] `programs/zstdcli.c:1661-1664` dispatches compression to single-file or batch entry points.
- [S] `programs/fileio.c:2457-2465` iterates all inputs and calls `UTIL_getFileSize()`.
- [S] `programs/fileio.c:2485-2487` invokes that prepass unconditionally.
- [S] `programs/fileio.c:1406-1410` consumes `maxSrcFileSize` only under `prefs->patchFromMode`.
- [S] `programs/util.c:545-557` implements the filesystem query; failures return `UTIL_FILESIZE_UNKNOWN`.
- [S] `programs/fileio.c:2291-2322` and `:2004` retain later source validation and sizing independently of the prepass.
- [O] `git ls-remote upstream refs/heads/dev` matched the recorded revision on 2026-10-23.

## Prior-Art

Coverage: GitHub PR search `repo:facebook/zstd FIO_getLargestFileSize` and issue search `repo:facebook/zstd "stat" "performance"`; checked=2026-10-23.
Gaps: No exhaustive discussion, release, or alternate-symbol search; no publication-ready novelty claim.
The PR query returned no matches; the issue query returned seven results without a directly identified owner for this root cause.
Contribution fit: Unselected; a small performance patch is plausible only after authorization, focused verification, and prior-art closure.

## Proposed-Change

Evaluate `FIO_getLargestFileSize()` only when `prefs->patchFromMode` is enabled.
Pass an explicitly unused value otherwise, without changing later per-file size queries or patch parameter selection.

## Scope-and-Constraints

- Preserve ordinary output, source validation, missing-file handling, stdin handling, and patch-mode size requirements.
- Preserve context and dictionary reuse, file ordering, and resource cleanup.
- Exclude the separate single-file entry point, metadata caching, codec changes, and unrelated stat calls.
- Cost: One conditional at the batch resource-creation boundary; performance value remains unmeasured.
- Test decision: none; use existing cases and focused smoke comparisons, not new permanent tests.

## Performance-Evidence

Environment: Ubuntu 24.04 x86-64, Linux 6.8.0-107-generic, Ryzen 9 5950X, GCC 13.3.0, release `-O3`, strace 6.8.
Build: `make -C programs -j8 zstd-release`; local CLI reports v1.6.0.
Binary SHA-256: `8ce9b1f28c8e3b4703fa7a83f8a91d44c86b313e3af0e14f412535c5ee044066`.
Input: `lib/` extracted from `git archive --format=tar --output=/tmp/zstd-perf-QzgStH/source.tar upstream/dev lib programs`.
Trace: `strace -qq -k --stack-trace-frame-limit=6 -e trace=newfstatat -o /tmp/zstd-perf-QzgStH/stat.trace programs/zstd -q -f --single-thread -r /tmp/zstd-perf-QzgStH/lib -o /dev/null`.
Observed: 104 stacks reached `UTIL_getFileSize()` directly from the inlined batch prepass at `FIO_compressMultipleFilenames+0x79`.
The first attempt omitted `-f` and aborted at the concatenation prompt; only the successful retry supplies the reported evidence.
The temporary archive, extraction, and raw trace were removed after analysis; observations remain in the session transcript.

## Verification

- After an authorized patch, compare syscall counts on the same batch: one fewer prepass query per input outside patch mode.
- Compare baseline and patched compressed bytes and roundtrips with identical parameters.
- Exercise missing files, stdin, and patch mode, including its unknown-size error contract.
- Use existing `tests/cli-tests/compression/multiple-files.sh` where applicable; measure untraced runs separately from syscall tracing.

## Publication-Blockers

Implementation authorization and publication target are unselected.
No source patch, matched before/after timing, completed prior-art search, or approved publication draft exists.

## Next-Action

Summary: Select scoped implementation authorization
Action: Obtain the user's Authorized-Work selection for the batch-prepass scope without inferring publication approval.
Done-When: The selected authorization is recorded here and projected into ISSUES.md.
