# Phase 1 Plan: Software Baseline (Guided)

## Context
Phase 1 of `docs/plan.md`: a C++ ITCH 5.0 parser and order book that processes a full-day NASDAQ file and becomes the reference model for the RTL phases. The user writes all code, specs, tests, and build files; Claude teaches the concepts and reviews the work.. The user has VHDL experience and no C++ experience.

**BLUF:** Set up the Ubuntu toolchain and a Git-based review loop, then complete seven milestones (M0–M6). Each milestone pairs one set of C++ concepts with one project deliverable, verified by tests. Phase 1 is done when the book builder processes a full-day file with no invariant violations and emits a deterministic best bid/offer (BBO) output file for later RTL comparison.

## Workflow
- **Sync through GitHub** (`origin` = `github.com/aidanfritzke/tick_to_trade_hil`), not file copying.
- Each milestone: short concept brief (with VHDL analogies) → spec in `esw/docs/` → GoogleTest tests in `esw/tests/` → user implementation → review → merge.

## Environment Setup (Ubuntu 24.04)
- `sudo apt install build-essential cmake ninja-build gdb git clangd clang-format clang-tidy valgrind linux-tools-common linux-tools-$(uname -r)`
- GCC 13, C++20. GoogleTest pulled by CMake `FetchContent`.
- CMake presets: `debug`, `release`, `asan` (AddressSanitizer + UndefinedBehaviorSanitizer). `CMAKE_EXPORT_COMPILE_COMMANDS=ON` for clangd.
- Optional vim support: a minimal `.vimrc` plus clangd through a vim language-server plugin, for errors and go-to-definition.
- Data: download one NASDAQ TotalView-ITCH 5.0 sample day (e.g., `01302019.NASDAQ_ITCH50.gz` from `emi.nasdaq.com`) into `data/` (gitignored), plus the ITCH 5.0 specification PDF. Allow about 20 GB free disk space. Use `gunzip -k` to decompress.

## Code Layout
```
esw/
  CMakeLists.txt, CMakePresets.json
  libs/itch/   include/itch/*.hpp, src/*.cpp   byte readers, framing, message decode
  libs/book/   include/book/*.hpp, src/*.cpp   order store, price levels, BBO
  apps/itch_stats/     message counts per type
  apps/book_builder/   full-day run, BBO output
  tests/               test suites, regressions, etc
  docs/                milestone specs and concept briefs
util/test_vectors/     small hand-built and sliced ITCH inputs
```
Libraries are separate from applications so later phases (MoldUDP64 replay, cocotb comparison) reuse the same code.

## Milestones
| # | Deliverable | C++ concepts (VHDL analogy) | Done when |
|---|---|---|---|
| M0 | Toolchain, CMake project, one passing test, Git loop | compile/link (analyze/elaborate), headers vs. sources (package vs. package body), `main` | `ctest` passes on Linux; first push reviewed |
| M1 | Big-endian readers: u16, u32, u48, u64 | fixed-width integers, shifts, `std::span`, `constexpr`, why not to cast raw bytes to structs | Unit tests pass under the `asan` preset |
| M2 | Length-prefixed framing and `itch_stats` app | file I/O, buffers, `std::array`, error handling | Full-day file parsed; per-type message counts reported with throughput |
| M3 | Message structs and decode for all ITCH 5.0 types; unknown types skipped | `struct` (record), `enum class` (enumerated type), `switch` (case statement) | Decode tests pass against hand-built byte vectors |
| M4 | Order book: add, execute, cancel, delete, replace (A, F, E, C, X, D, U) and BBO tracking | `std::unordered_map`, `std::map`, classes, references, RAII | Per-event unit tests pass; book invariants hold |
| M5 | `book_builder` full-day run | `std::chrono` timing, `perf`, basic profiling | Full day completes with no invariant violations; runtime recorded |
| M6 | Reference-model outputs: deterministic BBO update file and sliced test vectors | binary/CSV output, command-line arguments | Output format documented for cocotb comparison in Phases 3–4 |

Book design starts simple and correct (hash map of orders keyed by order reference number; ordered map of price levels per side per stock locate). Array-based structures that mirror block RAM come later as an optimization, checked against the simple version.

## Verification
- `ctest --preset debug` and `ctest --preset asan` pass on the Linux PC at every milestone.
- `itch_stats` framing consumes exactly the file size in bytes; no truncated or leftover bytes at end of file.
- `book_builder` invariants: no negative quantities, no references to unknown orders, and every order removed from the store once fully executed or deleted.
- BBO output is identical across runs (determinism check by file hash).

