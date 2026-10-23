# project-zstd-fork

<essential-rule>
AGENTS.md is the sole authoritative project context file.
Read and edit AGENTS.md directly.
</essential-rule>

## Development Rules

Before launching agents, apply skill-xray, skill-expert, and skill-brutal to the task.
Surface expert-level issues, non-obvious issues, blindspots, stale assumptions, and hidden dependencies.
Also surface missed constraints, edge cases, false positives, verification gaps, overclaims, and weak assumptions.
Identify improvement potential, inefficiencies, and what is wrong without softening.
Use these findings to design safe slices, sequencing, checks, and boundaries for complete agent results.

Every agent prompt must require skill-xray, skill-expert, and skill-brutal for the assigned scope before acting.
It must surface non-obvious issues, blindspots, stale assumptions, hidden dependencies, and edge cases.
It must also surface verification gaps, overclaims, failure modes, weak assumptions, and what is wrong.
The agent must adjust its approach, challenge its assumptions, and flag misleading or incomplete output risks.

Implementation assignments must cover existing patterns, callers, exported-symbol consumers, and failure modes.
They must also cover concurrency safety and lifecycle cleanup.
Each assignment must state `Test decision: none` or `Test decision: update`.
`update` must name the exact existing test that follows an intentional contract change.
Never request new tests.
Prohibit broad edits, unrelated cleanup, and unassigned files.

No vague agents.
Each assignment needs exact targets, non-goals, evidence anchors, acceptance criteria, and an output contract.

Commit completed units continuously.
Before each commit, use skill-git-commit-format to determine whether staged effects are one coherent unit.
The skill owns commit-message format and evidence.
After the boundary is valid, run the repository-owned commit and push workflow.
Do not commit every trivial edit immediately or defer unrelated work into one end-of-session commit.

Every project-level quality command is quiet by default and verbose on demand.
This policy applies regardless of language or toolchain.
It covers Make targets, package scripts, Python CLIs, shell quality gates, and test runners.
Successful checks print only compact status such as `format: ok`, `lint: ok`, `test: ok`, or `check: ok`.
On failure, exit non-zero and print the failing step, exit code, and enough output to act without rerunning.
Full raw output must remain available through `--verbose`, `VERBOSE=1`, or the underlying tool's verbose mode.
New quality commands and future language setup must follow this policy instead of inventing another logging contract.

# Repository Guidelines

## Project Overview

Zstandard (`zstd`) is the reference implementation of the fast, lossless compression format standardized by RFC 8878.
The repository provides portable C `libzstd`, the `zstd` CLI, dictionary training, tests, benchmarks, fuzzers, and integrations.
Preserve compression-format compatibility, portability, memory discipline, and performance when changing core code.

Development normally targets `dev`; release-ready work is merged into `release`.
Keep changes focused and update user-facing documentation when a public API or CLI contract changes.

## Fork & Upstream Contribution Intent

- Official upstream: [facebook/zstd](https://github.com/facebook/zstd).
- This checkout is the [MikeeI/project-zstd-fork](https://github.com/MikeeI/project-zstd-fork) fork, not an independent product.
- The goal is to support upstream with evidence-backed issues, comments, and pull requests.
- `origin` is `git@github.com:MikeeI/project-zstd-fork.git`; `upstream` is `git@github.com:facebook/zstd.git`.
- `personal` tracks `origin/personal` and owns fork-only context and tracking.
- The fork's `dev` branch is the clean upstream mirror; the GitHub default remains `dev`.
- Base each contribution branch or separate worktree on current `upstream/dev`, never on `personal`.
- Keep fork-only context, ledgers, configuration, and personal commits out of upstream contribution diffs.
- `ISSUES.md` provides the compact finding overview and global ID allocator.
- `issues/ISSUE-NNN.md` owns the complete durable record for one root cause.
- `FORMAT.md` owns research, drafting, implementation authorization, approval, and publication rules.
- Prefer small, well-scoped corrections with outsized maintainer or user value.
- Recommend a pull request when a bounded verified fix is ready and no active implementation owns it.
- Otherwise recommend a comment, a new issue, or continued investigation according to the evidence.
- Apply `skill-fork-contribution-tracking` for ledger, lifecycle, personal-branch, and upstream handoff work.
- Apply `skill-maintainer-communication` before external issues, pull requests, reviews, comments, or discussions.
- Search existing work first and follow current upstream templates and disclosure rules.
- Apply `skill-semantic-compression-3` when authoring or restructuring tracking content.
- Apply `skill-git-commit-format` while respecting explicit upstream contribution conventions.
- Never choose `Authorized-Work` or `Publication-Target` on the user's behalf.
- `Research-and-Reporting` permits research, issues, and comments but no source implementation.
- `Pull-Request-Implementation` authorizes only the scoped implementation recorded for that finding.
- Reproduce claimed bugs against current upstream and run the narrowest conclusive verification.
- Publish one coherent root cause per issue, comment, or pull request.
- Follow `CONTRIBUTING.md`, including the CLA requirement and fork CI before upstream submission.
- Use `.github/PULL_REQUEST_TEMPLATE.md` and disclose tools used for discovery, code, and tests.
- Bug fixes need a reproducer or a test that fails without the fix.
- `lib/legacy/` is frozen except for memory-safety and regression fixes, each accompanied by a test.
- Security findings follow `SECURITY.md`; do not submit a public security issue or pull request.
- Performance evidence must identify CPU, compiler, flags, data, and repeated runs without generalizing one configuration.
- Non-x86-64/AArch64 performance changes must be small, architecture-guarded, benchmarked, and pass existing CI.

## Finding and Contribution Ledger

- At the start of every agent session, agents MUST read root `ISSUES.md` before repository work.
- `ISSUES.md` owns the global `Next finding ID` allocator and compact cross-finding overview.
- Each `issues/ISSUE-NNN.md` owns one finding's state, evidence, drafts, and next action.
- `FORMAT.md` is authoritative for research, drafting, implementation boundaries, and publication format.
- Before adding a finding, search the index and every relevant issue record for the same symptom or root cause.
- New findings MUST use `Next finding ID`.
- Create the issue file, add its index row, and increment the allocator together.
- Finding IDs use `ISSUE-NNN`, start at `ISSUE-001`, and remain permanent.
- Never reuse, renumber, or scope IDs by subsystem, status, session, or contribution type.
- Update the issue file and `ISSUES.md` together after state, authorization, target, priority, next action, or reference changes.
- Every issue record MUST use the field and section contract in `FORMAT.md`.
- Use the current `skill-fork-contribution-tracking/templates/ISSUE.md` template for each new finding.
- New findings start with `State: Investigating`, `Authorized-Work: Not-Selected`, and `Publication-Target: Not-Selected`.
- New findings use `External-Reference: Not published.` until an external reference exists.
- Keep findings Investigating until currentness, prior art, impact, and correction value are evidence-backed.
- Open findings require a concrete repository-owned problem, evidence of its reachable impact, and a bounded correction to investigate.
- Usage tips, level comparisons, benchmark baselines, and mistakes in an agent's report are not repository issues.
- Keep reusable investigation guidance here; archive misclassified findings without deleting their evidence or reusing their IDs.
- Clone detectors, AST matches, text similarity, shared names, and TODOs produce candidates only.
- A duplication finding requires shared change pressure, realistic drift, and simpler consolidation.
- The user selects `Authorized-Work` for each finding.
- `Research-and-Reporting` MUST NOT implement the finding.
- `Pull-Request-Implementation` MAY implement only the recorded scope after research resolves callers and failure modes.
- Pull-request work MUST verify behavior, commit, push, and reach `PR-Ready` before publication.
- Show the exact draft and target before publishing an issue, comment, or pull request to official upstream.
- Publish to official upstream only after the user approves the exact current draft and target.
- Any draft or target change requires showing the complete current draft and target again before publication.
- Run the read-only validator bundled with `skill-fork-contribution-tracking` after every ledger mutation.
- Record the final external URL in `External-Reference` immediately after publication.
- Store archived records under `issues/archive/` and update index links in the same change.
- Keep `FORMAT.md`, `ISSUES.md`, `issues/`, and fork-only `AGENTS.md` changes out of upstream contribution diffs.

### External publication approval

Only an external issue, comment, review, discussion, or pull request write is approval-gated.
Before publication, read current contribution guidance and explain applicable project policy.
The human must be able to review and own every submission statement.
Fork commits, pushes, tracking updates, and source implementation follow the active repository contract.

## Architecture & Data Flow

The code is deliberately layered:

- `lib/common/` owns shared primitives, error encoding, portability wrappers, threading, and the worker pool; it must not depend on other `lib/` modules.
- `lib/compress/` and `lib/decompress/` implement independent codec paths and must not depend on each other.
- `lib/dictBuilder/` depends on common and compression code to train dictionaries.
- `lib/legacy/` adds optional decoding for historical frame formats.
- `programs/` depends on `libzstd` through its public API.
- `programs/zstdcli.c::main()` dispatches parsed options to `fileio.c`, `benchzstd.c`, or dictionary-builder APIs.

Library calls operate either one-shot or through explicit stateful contexts.
Compression flows from input through a `ZSTD_CCtx` into a frame.
Decompression parses frame/block headers through a `ZSTD_DCtx` and emits decoded bytes.
Streaming APIs exchange `ZSTD_inBuffer` and `ZSTD_outBuffer` values.
Callers must preserve context lifecycle and progress semantics.

`lib/compress/zstdmt_compress.c` isolates multithreaded compression and schedules jobs through `POOL_ctx`.
`lib/common/threading.h` abstracts POSIX, Windows, and no-thread builds.
This is explicit worker-pool concurrency, not language-level async.
Contexts and preference structs are passed explicitly.
There is no dependency-injection container or global application state manager.

## Key Directories

- `lib/` — public headers and codec implementation.
- `programs/` — CLI parsing, file I/O, benchmarking, training, and helper commands.
- `tests/` — CLI integration tests, native test tools, regression checks, fuzzing, sanitizers, and compatibility suites.
- `examples/` — small public-API examples, including streaming and dictionary use.
- `contrib/` — separately scoped integrations such as `pzstd` and seekable format support; conventions may differ from core code.
- `build/cmake/`, `build/meson/`, `build/VS_scripts/` — secondary build-system and platform integrations.
- `doc/` — compression-format documentation, generated API manual, errata, and educational decoder.
- `zlibWrapper/` — zlib-compatible wrapper and its dedicated tests.

## Development Commands

GNU Make is the reference build system; prefer it when possible.

```bash
make                         # build release libzstd and zstd CLI
make check                   # basic CLI smoke/integration tests
make test                    # longer test suite with variants
make -C tests test-cli-tests # CLI test harness only
make staticAnalyze           # clang scan-build analysis
make clean                   # remove generated build outputs
```

Useful focused builds include `make lib`, `make zstd`, `make examples`, and `make -C tests <target>`.
Add `V=1` or `VERBOSE=1` to expose compiler commands.

Supported alternatives:

```bash
cmake -S . -B build-cmake
cmake --build build-cmake

meson setup build/meson mesonbuild
ninja -C mesonbuild
meson test -C mesonbuild --print-errorlogs
```

Use `make manual` to regenerate `doc/zstd_manual.html` from `lib/zstd.h`, and `make man` for CLI manpages.
Do not hand-edit generated documentation without checking its owning generator.

## Code Conventions & Common Patterns

- Core code in `lib/` and `programs/` is strict C90 plus `long long` and variadic macros.
- Keep public declarations C++98-compatible with the existing `extern "C"` wrappers.
- Match adjacent style: 4-space indentation, lowercase `snake_case` filenames, and unique filenames across the repository.
- Public symbols use `ZSTD_`; functions and variables generally use `PREFIX_camelCase`, types use forms such as `ZSTD_CCtx`, and macros use `PREFIX_UPPER_CASE`.
- Prefer action-object names, positive predicates, and `const` for every value or pointee that is not modified.
- Preserve the dependency hierarchy: `programs/` uses public library APIs, and compression/decompression stay decoupled.
- Keep dependencies minimal and respect small-stack requirements.
- Core library code allocates through `ZSTD_malloc()`-family wrappers rather than directly.
- Public APIs commonly return `size_t` with encoded errors checked through `ZSTD_isError()`.
- Internal code uses `ERROR`, `RETURN_ERROR_IF`, and `FORWARD_IF_ERROR` from `lib/common/error_private.h`.
- Use assertions for invariants and `DEBUGLOG(level, ...)` from `lib/common/debug.h` for debug tracing.
- Do not add comments that merely restate code.
- Context ownership is explicit: create, reset, reference/load dictionaries, and free using the matching API.
- Preserve cleanup on every error path.
- No repository-wide formatter target exists; follow surrounding code rather than broadly reformatting.

## Important Files

- `lib/zstd.h` — stable public compression, decompression, streaming, and context API; also contains static-linking-only experimental APIs.
- `lib/zdict.h`, `lib/zstd_errors.h` — dictionary training and stable error API.
- `lib/common/zstd_internal.h` — shared internal codec definitions.
- `lib/compress/zstd_compress.c`, `lib/decompress/zstd_decompress.c` — primary codec implementations.
- `programs/zstdcli.c` — CLI entry point and option dispatcher.
- `programs/fileio.c`, `programs/fileio.h` — filesystem-oriented compression/decompression orchestration.
- `Makefile` — authoritative build and top-level QA command surface.
- `tests/Makefile`, `tests/playTests.sh` — main native and CLI test orchestration.
- `tests/cli-tests/run.py` — isolated CLI test runner and output-expectation contract.
- `tests/fuzz/fuzz.py`, `tests/fuzz/Makefile` — fuzz target build, corpus, and regression workflows.
- `CMakeLists.txt`, `build/cmake/CMakeLists.txt`, `build/meson/meson.build`, `Package.swift` — supported non-Make integrations.
- `CONTRIBUTING.md`, `TESTING.md` — contributor workflow, coding rules, CI tiers, and performance expectations.

## Performance Investigation Map

This map was checked against `upstream/dev@49cf51799ea051457f57243527aa2058f999f005` on 2026-10-23.
Use symbols as navigation anchors; line numbers and defaults can move with upstream.
Start with the affected flow below instead of repeating a repository-wide scan.
This is navigation and investigation context, not implementation authorization or a substitute for finding records.

### CLI I/O and resource ownership

- `programs/zstdcli.c::main()` owns option parsing and dispatch into single-file or multiple-file operations.
- `programs/fileio_types.h` owns `FIO_prefs_t`; `fileio.c::FIO_createPreferences()` initializes its runtime defaults.
- Compression follows `FIO_compressMultipleFilenames()` → `FIO_compressFilename_srcFile()` → `FIO_compressFilename_dstFile()` → `FIO_compressFilename_internal()` → `FIO_compressZstdFrame()`.
- `fileio.c::FIO_createCResources()` owns the reusable compression context, dictionary, and synchronous I/O buffers.
- `fileio.c::FIO_SyncCompressIO_*()` implements compression I/O; do not assume compression uses the asynchronous pools.
- `fileio.c::FIO_getLargestFileSize()` performs the batch size prepass; `FIO_createCResources()` consumes its result only for patch-from parameter selection.
- `programs/util.c::UTIL_getFileSize()` performs a filesystem stat; `UTIL_getFileSizeStat()` consumes existing metadata.
- `fileio.c::FIO_compressFilename_srcFile()` and `FIO_compressFilename_internal()` own later source validation and sizing.
- Decompression resources originate in `fileio.c::FIO_createDResources()` and use read/write pools from `fileio_asyncio.c`.
- `fileio_asyncio.c::AIO_IOPool_*()` owns jobs, buffers, mutexes, worker activation, draining, and destruction.
- `fileio.c::FIO_decompressSrcFile()` disables async dispatch for known inputs smaller than three maximum blocks.
- Sparse output has two owners: `fileio.c::FIO_sparseWrite()` and `fileio_asyncio.c::AIO_fwriteSparse()`.
- Both sparse writers scan 32 KiB segments; their matching end functions materialize the final zero byte.
- `fileio.c::FIO_openDstFile()` owns regular-file detection, sparse-mode eligibility, and stdio buffering.
- `programs/platform.h::ZSTD_SPARSE_DEFAULT` disables sparse output by default on macOS, unlike Linux.
- `fileio.c::FIO_compressZstdFrame()` collects adaptive statistics; `zstdmt_compress.c::ZSTDMT_toFlushNow()` locks the oldest active job.

### Compression ratio and CPU ownership

- `lib/compress/clevels.h::ZSTD_defaultCParameters` owns level presets, including separate source-size classes.
- `zstd_compress.c::ZSTD_getCParams_internal()` selects size classes at 16, 128, and 256 KiB; unknown sizes select differently.
- `lib/compress/zstd_compress.c::ZSTD_selectBlockCompressor()` dispatches by strategy, dictionary mode, and row-matchfinder mode.
- `zstd_fast.c` and `zstd_double_fast.c` own fast parsers; `zstd_lazy.c` owns greedy/lazy and row-based matching.
- `zstd_opt.c` owns optimal parsing; `zstd_ldm.c` owns long-distance matching.
- `zstd_opt.c::ZSTD_getMatchPrice()` deliberately favors decoder speed through offset and sequence-price penalties.
- `lib/common/zstd_internal.h::ZSTD_OPT_NUM` bounds optimal parsing; larger `targetLength` values do not bypass it.
- `zstdmt_compress.c::ZSTDMT_computeTargetJobLog()` and `ZSTDMT_computeOverlapSize()` own MT job sizing and history overlap.
- `zstd_compress.c::ZSTD_optimalBlockSize()` gates pre-splitting; `zstd_preSplit.c::ZSTD_splitBlock()` implements fingerprint-based choices.
- Pre-splitting requires full 128 KiB blocks and prior savings; bypassing that gate risks expansion guarantees.
- `zstd_compress.c::ZSTD_resolveBlockSplitterMode()` selects post-sequence splitting for stronger strategies.
- `zstd_compress_superblock.c` owns the separate `targetCBlockSize` path; do not conflate it with ordinary pre-splitting.
- `zstd_compress.c::ZSTD_compress_frameChunk()` selects the target-size path instead of ordinary post-sequence splitting when enabled.
- `zstd_compress_literals.c::ZSTD_compressLiterals()` owns literal thresholds, raw/RLE fallback, and Huffman-table state.
- `huf_compress.c::HUF_compress_internal()` owns incompressibility sampling, histograms, table validation, and table reuse.
- `huf_compress.c::HUF_optimalTableLog()` probes tree depths; removing rebuilds requires preserving the selected table, not merely the last candidate.
- `zstd_compress_sequences.c::ZSTD_selectEncodingType()` chooses basic, repeated, RLE, or compressed FSE tables.
- `zstd_compress.c::ZSTD_entropyCompressSeqStore()` integrates literals and sequences and enforces minimum compression gain.
- `zstd_compress.c::ZSTD_initLocalDict()` already caches the prepared local dictionary across session-only resets.
- `zstd_cwksp.h` owns reusable workspace allocation; inspect its phase and initialization invariants before proposing allocation reuse.
- `lib/zstd.h` owns public and experimental parameter contracts, including stable buffers, size hints, dictionaries, and splitters.
- `lib/zdict.h` documents dictionary suitability and training; `lib/dictBuilder/fastcover.c` and `programs/dibio.c` implement it.
- `lib/zstd.h::ZSTD_CCtx_refCDict()` documents which context parameters a prepared dictionary supersedes.
- `lib/zstd.h::ZSTD_CCtx_refPrefix()` and `programs/zstd.1.md` document prefix lifetime and `--patch-from` requirements.
- `lib/zstd.h::ZSTD_EndDirective` distinguishes frame-ending history loss from flushing within an existing frame.
- `lib/decompress/zstd_decompress.c` owns frame/stream dispatch; `zstd_decompress_block.c` and `huf_decompress.c` own block and Huffman decoding.

### Measurement and verification entry points

- Build the local release CLI with `make -C programs -j8 zstd-release`; invoke `programs/zstd`, not a global installation.
- `programs/Makefile` owns CLI feature detection and hashed object directories; `lib/libzstd.mk` owns compiler defaults.
- `programs/benchzstd.c` and `benchfn.c` own `-b` timing; use repeated `programs/zstd -b3 -e3 -i1 CORPUS` runs for an initial signal.
- Record the binary hash, source revision, compiler flags, and build provenance separately; a version string does not link a binary to a checkout.
- Enable `pipefail` for measurement pipelines so a failed compressor cannot appear as a successful zero-byte result.
- Inspect benchmark diagnostics: the final hash-mismatch warning in `BMK_benchMemAdvancedNoAlloc()` does not return failure.
- Verify selected outputs separately with bytewise roundtrips through the actual CLI or API path.
- `benchzstd.c::BMK_initCCtx()` maps one benchmark worker to zero; `-b -T1` does not reproduce ordinary CLI `-T1`.
- `tests/paramgrill.c` searches eight parameters; `tests/README.md` documents speed, memory, and runtime constraints.
- Paramgrill does not search row-matchfinder, LDM, or splitter switches; compare relevant omitted controls separately.
- Bound Paramgrill with `-t#`; its table-generation mode writes `grillResults.txt`.
- A reproducible source corpus is `git archive --format=tar --output=CORPUS REVISION lib programs`.
- Separate codec timing from filesystem timing; `-b` does not measure ordinary CLI file I/O.
- For sparse output, compare `strace -f -c -e trace=lseek,write` counts against `--no-sparse` on real destination files.
- Do not use `/dev/null` as sparse-output evidence: destination detection disables sparse mode there.
- glibc may translate `fseek(..., 0, SEEK_CUR)` into absolute `lseek(..., SEEK_SET)` calls.
- For metadata prepasses, `strace -k -e trace=newfstatat` identifies call stacks; optimized helper functions may be inlined.
- `tests/cli-tests/run.py` accepts test paths relative to `tests/cli-tests/` and explicit `--zstd`/`--datagen` paths.
- Existing focused cases include `compression/multiple-files.sh`, `compression/stream-size.sh`, and `compression/adapt.sh`.
- Literal and dictionary contracts have `compression/compress-literals.sh` and `dictionaries/golden.sh`.
- `compression/row-match-finder.sh` and `compression/long-distance-matcher.sh` cover their respective CLI switches.
- `tests/fuzzer.c` contains incompressible-input splitter checks; `tests/fuzz/zstd_helpers.c` varies splitter parameters.
- `tests/cli-tests/determinism/` owns exact bitstream expectations that encoder changes can intentionally affect.
- `tests/playTests.sh` owns sparse, odd-sized sparse, async-I/O, and patch-from scenarios; inspect those sections before choosing a broader run.
- `tests/regression/README.md` documents corpus-based ratio comparisons; do not overwrite `results.csv` merely to run an investigation.
- `tests/regression/data.c` owns corpus URLs and checksums; `config.c` and `method.c` own profiles and API paths.
- Treat `tests/regression/results.csv` as a historical size snapshot, not a current speed measurement or dictionary-inclusive total.

### Avoid rediscovering false positives

- Batch compression already reuses its context, buffers, and prepared dictionary; generic pooling advice adds no new optimization.
- Huffman sampling, table reuse, literal thresholds, and block-splitting heuristics already exist.
- Stronger levels, larger windows, forced literal compression, and extra splitting are trade-offs, not free ratio improvements.
- Smaller independent chunks can change both history and size-dependent presets; hold parameters fixed to isolate either cause.
- Neither exact source sizes nor single-thread mode guarantee smaller output; compare the actual workload and output path.
- Dictionary and prefix gains must include any additionally required side data and preserve decoder availability.
- Keep tuning and validation inputs separate; one successful sample does not justify a new global preset.
- Related size-tier discussion: https://github.com/facebook/zstd/issues/3729
- Existing splitter design and expansion constraints: https://github.com/facebook/zstd/pull/4136
- Optimal-parser parameter limits: https://github.com/facebook/zstd/issues/2730
- The 2026-10-23 source-corpus baseline was 4,014,080 bytes → 738,793 bytes at level 3; it proves no improvement.
- That baseline used GCC 13.3.0, release `-O3`, Ryzen 9 5950X, and three one-second benchmark runs.
- Sparse syscall reduction is not a demonstrated speedup: nine local runs measured 71.2 ± 1.2 ms versus 69.5 ± 1.9 ms with `--no-sparse` on 64 MiB random data.
- Those sparse timings used `--no-asyncio`, warm files, and the same release build; neither run implemented a source fix.
- Historical sparse discussion includes an unsuccessful seek-reduction experiment, so avoid generic speed claims: https://github.com/facebook/zstd/issues/615
- The 64 MiB integrity-check comparison favored enabled async I/O; it did not isolate read and write workers.
- An unused adaptive-statistics lock is a candidate only until profiling establishes material cost.
- Test-mode write jobs return without writing, but eliminating their worker requires preserving read overlap and buffer ownership.

## Runtime/Tooling Preferences

Use GNU Make (`gmake` where system `make` is incompatible) and a C compiler as the default toolchain.
Python 3 and POSIX shell are required by several test harnesses; Ninja is used with Meson.
CMake requires at least 3.10, while Meson declares at least 0.50.0.

The project is not a Node/Bun package and has no JavaScript package manager.
Optional dependencies such as pthreads, zlib, liblzma, and liblz4 depend on selected CLI/library features.
Keep cross-platform wrappers intact rather than introducing platform-specific calls into codec code.
Preserve upstream build scripts and their output conventions; do not retrofit personal scaffolding into upstream tooling.

## Testing & QA

Choose the narrowest test that covers the changed contract, then use the appropriate repository gate:

- `make check` for routine CLI behavior and smoke coverage.
- `make test` for broader, longer validation and build variants.
- `make -C tests test-cli-tests` covers `programs/` behavior in isolated scratch directories.
- `tests/cli-tests/` uses executable scripts plus `.exit`, `.stdout.*`, and `.stderr.*` expectation files.
- `make -C tests/fuzz regressiontest` after codec changes affecting fuzz targets or known corpora.
- `make regressiontest` or `make -C tests/regression test` for compression-ratio/performance regression workflows.
- `make -C tests test-valgrind` for memory checking where Valgrind is supported.

Sanitizer targets include ASan, UBSan, MSan, and TSan variants in the root and test Makefiles.
Fuzzing supports libFuzzer and AFL through `tests/fuzz/fuzz.py`; corpora live under `tests/fuzz/corpora/`.
Some suites are long-running, platform-specific, or require QEMU/external tools.
Do not expand a focused change into the entire matrix without cause.

Performance is a maintained contract.
For hot codec paths, benchmark representative data and report ratio, compression/decompression speed, and memory trade-offs.
Do not claim improvement from unstable or single-run measurements.
