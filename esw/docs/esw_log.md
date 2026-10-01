# ESW Log

## 2026-09-27 — M0 Steps 1–2: Toolchain and first build
Installed the Ubuntu 24.04 C++ toolchain (GCC 13, CMake 3.28, Ninja, gdb, clang tools, perf), connected GitHub over SSH, cloned the repo to `~/projs/hft/tick_to_trade_hil`, and downloaded the ITCH 5.0 sample day and specification. Wrote a C++20 CMake project that configures with `cmake -S esw -B esw/build -G Ninja`, builds with `cmake --build esw/build`, and runs `itch_stats` with exit code 0 and no warnings.

| File | Requirement |
|---|---|
| `esw/CMakeLists.txt` | CMake ≥ 3.28; project `tick_to_trade_esw`; C++20, required, extensions off; `-Wall -Wextra -Wpedantic`; includes `apps/itch_stats` |
| `esw/apps/itch_stats/CMakeLists.txt` | Executable target `itch_stats` built from `main.cpp` |
| `esw/apps/itch_stats/main.cpp` | Prints `itch_stats: build OK`; returns 0 |
| `.editorconfig` | Final newline on every file for Notepad++ editing |
| `data/01302019.NASDAQ_ITCH50.gz` | Sample day; 4,764,426,091 bytes, matches server size; gitignored |
| `data/NQTVITCHspecification.pdf` | ITCH 5.0 specification; gitignored |
