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
