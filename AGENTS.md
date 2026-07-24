# Repository Guidelines

## Project Overview

Zstandard (`zstd`) is the reference implementation of a fast, lossless compression format standardized by RFC 8878. The repository provides the portable C `libzstd` library, the `zstd` command-line tool, dictionary-training support, tests, benchmarks, fuzzers, and selected integrations. Preserve compression-format compatibility, portability, memory discipline, and performance when changing core code.

Development normally targets `dev`; release-ready work is merged into `release`. Keep changes focused and update user-facing documentation when a public API or CLI contract changes.

## Architecture & Data Flow

The code is deliberately layered:

- `lib/common/` owns shared primitives, error encoding, portability wrappers, threading, and the worker pool. It must not depend on other `lib/` modules.
- `lib/compress/` and `lib/decompress/` implement independent codec paths and must not depend on each other.
- `lib/dictBuilder/` depends on common and compression code to train dictionaries.
- `lib/legacy/` adds optional decoding for historical frame formats.
- `programs/` depends on `libzstd` through its public API. `programs/zstdcli.c::main()` parses modes and options, then dispatches file operations to `programs/fileio.c`, benchmarks to `programs/benchzstd.c`, or training to dictionary-builder APIs.

Library calls operate either one-shot or through explicit stateful contexts. Compression flows from input through a `ZSTD_CCtx` into a frame; decompression parses frame/block headers through a `ZSTD_DCtx` and emits decoded bytes. Streaming APIs exchange `ZSTD_inBuffer` and `ZSTD_outBuffer` values and require callers to preserve context lifecycle and progress semantics.

Multithreaded compression is isolated in `lib/compress/zstdmt_compress.c`. It schedules jobs through `POOL_ctx`; `lib/common/threading.h` abstracts POSIX, Windows, and no-thread builds. This is explicit worker-pool concurrency, not language-level async. Contexts and preference structs are passed explicitly; there is no dependency-injection container or global application state manager.

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

Useful focused builds include `make lib`, `make zstd`, `make examples`, and `make -C tests <target>`. Add `V=1` or `VERBOSE=1` to expose compiler commands.

Supported alternatives:

```bash
cmake -S . -B build-cmake
cmake --build build-cmake

meson setup build/meson mesonbuild
ninja -C mesonbuild
meson test -C mesonbuild --print-errorlogs
```

Use `make manual` to regenerate `doc/zstd_manual.html` from `lib/zstd.h`, and `make man` for CLI manpages. Do not hand-edit generated documentation without checking its owning generator.

## Code Conventions & Common Patterns

- Core code in `lib/` and `programs/` is strict C90 plus `long long` and variadic macros. Keep public declarations C++98-compatible with the existing `extern "C"` wrappers.
- Match adjacent style: 4-space indentation, lowercase `snake_case` filenames, and unique filenames across the repository.
- Public symbols use `ZSTD_`; functions and variables generally use `PREFIX_camelCase`, types use forms such as `ZSTD_CCtx`, and macros use `PREFIX_UPPER_CASE`.
- Prefer action-object names, positive predicates, and `const` for every value or pointee that is not modified.
- Preserve the dependency hierarchy. In particular, `programs/` must not consume private library internals, and compression/decompression must remain decoupled.
- Keep dependencies minimal. Core library code uses project wrappers instead of directly allocating; use `ZSTD_malloc()`-family mechanisms and respect small-stack requirements.
- Public APIs commonly return `size_t` with encoded errors. Check with `ZSTD_isError()`; internal code uses `ERROR`, `RETURN_ERROR_IF`, and `FORWARD_IF_ERROR` from `lib/common/error_private.h`.
- Use assertions for invariants and `DEBUGLOG(level, ...)` from `lib/common/debug.h` for debug tracing. Do not add comments that merely restate code.
- Context ownership is explicit: create, reset, reference/load dictionaries, and free using the matching API. Preserve cleanup on every error path.
- There is no repository-wide formatter target. Follow surrounding code rather than applying broad automated reformatting.

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

Use GNU Make (`gmake` on platforms where system `make` is not GNU-compatible) and a C compiler as the default toolchain. Python 3 and POSIX shell are required by several test harnesses; Ninja is used with Meson. CMake requires at least 3.10, while Meson declares at least 0.50.0.

The project is not a Node/Bun package and has no JavaScript package manager. Optional dependencies such as pthreads, zlib, liblzma, and liblz4 depend on selected CLI/library features. Keep cross-platform wrappers intact rather than introducing platform-specific calls into codec code.

## Testing & QA

Choose the narrowest test that covers the changed contract, then use the appropriate repository gate:

- `make check` for routine CLI behavior and smoke coverage.
- `make test` for broader, longer validation and build variants.
- `make -C tests test-cli-tests` for `programs/` behavior. Tests under `tests/cli-tests/` use executable scripts plus `.exit`, `.stdout.*`, and `.stderr.*` expectation files and run in isolated scratch directories.
- `make -C tests/fuzz regressiontest` after codec changes affecting fuzz targets or known corpora.
- `make regressiontest` or `make -C tests/regression test` for compression-ratio/performance regression workflows.
- `make -C tests test-valgrind` for memory checking where Valgrind is supported.

Sanitizer targets include ASan, UBSan, MSan, and TSan variants in the root and test Makefiles. Fuzzing supports libFuzzer and AFL through `tests/fuzz/fuzz.py`; corpora live under `tests/fuzz/corpora/`. Some suites are long-running, platform-specific, or require QEMU/external tools, so do not expand a focused change into the entire matrix without cause.

Performance is a maintained contract. For changes on hot codec paths, benchmark representative data and report compression ratio, compression speed, decompression speed, and memory trade-offs as applicable; avoid claiming improvement from unstable or single-run measurements.
