# Deadlocked — Review & Modernization Plan

A review of the current state of the project and a phased plan to bring it up to
modern engineering standards: portable reproducible builds, no-root execution,
CI, tests, and a browser-based demo.

## Current state

Deadlocked is a complete, working 32-bit x86 kernel — it boots into protected
mode, sets up paging and interrupts, runs system calls, schedules across multiple
terminals, and executes a Unix-like userspace from a read-only filesystem image.
The code works; the *project* is frozen in an early form:

- History is two squashed "Initial commit" dumps — no meaningful git history.
- The build only runs on Linux, as root.
- Build artifacts and large binaries are committed into the tree.

## Problems holding it back

| # | Problem | Evidence | Impact |
|---|---------|----------|--------|
| 1 | Build only works on Linux, as root | `debug.sh` uses `sudo mount -o loop` to splice the kernel into an 8 MB disk image | Cannot build on Windows/macOS at all |
| 2 | 8+ MB of binaries committed | `mp3.img` (8 MB), `test.txt` (2 MB), `createfs`/`elfconvert`, `bootimg`, `*.o`, `fish.exe`, a `.swp` vim swapfile | Repo bloat; unclear what's canonical; non-reproducible |
| 3 | Build cruft tracked as source | `warnings.txt`, `err.txt`, `error.txt`, `DEBUG` are captured compiler output | Noise in the tree |
| 4 | No toolchain pinning | Bare `gcc`, no `-m32`/`-march`, assumes a 32-bit host | Breaks on any modern 64-bit-default distro |
| 5 | No CI, tests, `.gitignore`, license, or real README | — | No regression safety net; not portfolio-ready |

## Phased plan

Each phase ships independently, lowest-risk first.

### Phase 1 — Repo hygiene + boot without root  *(the big unlock)*
- Remove all build artifacts, binaries, and cruft from the tree; add a real `.gitignore`.
- Optionally purge large binaries from git *history* with `git filter-repo` to shrink `.git`.
- **Replace the `sudo mount` image hack with direct QEMU multiboot boot:**
  `qemu-system-i386 -kernel bootimg -initrd filesys_img` — no root, no loopback, no disk image.
  This single change makes the kernel build and run identically on Windows, macOS, and Linux.
- Polish `README.md`.

### Phase 2 — Reproducible cross-compiler toolchain
- Pin an `i686-elf-gcc` cross-toolchain with explicit
  `-m32 -march=i386 -ffreestanding -Wall -Wextra` and a real **linker script**
  in place of `-Ttext=0x400000`.
- Modernize the Makefile: auto-dependencies via `-MMD`, and `make run` / `make debug` / `make clean` targets.
- Ship a **Dockerfile + `.devcontainer`** with the toolchain, QEMU, and gdb pinned,
  for one-command builds on any OS.

### Phase 3 — CI + tests + quality gates
- **GitHub Actions:** build in the Docker image on every push, then boot headless
  in QEMU with a serial-console smoke test (assert the kernel reaches a known
  state / runs `ls`), guarded by a timeout.
- Fix the compiler warnings recorded in `warnings.txt`.
- Add `clang-format` (preserving the existing `ts=4 noexpandtab` style) and `cppcheck`.

### Phase 4 — Modern showcase
- **Boot Deadlocked in the browser** via [v86](https://github.com/copy/v86)
  (an x86 emulator compiled to WebAssembly), hosted on **GitHub Pages** — anyone
  can run the OS, fish animation and all, from a link.
- Rewrite docs: an architecture diagram (memory map, PCB stacks, paging), a
  screen-capture GIF of the shell and `fish`, and a polished retrospective built
  from `buglog.txt`.
- Add a `docs/STATUS.md` so fresh sessions can resume work.

### Out of scope (intentionally)
- A 64-bit / long-mode port, or an APIC/HPET rewrite — large effort that defeats
  the purpose of a focused i386 teaching kernel.

## Recommended starting point

Start with **Phase 1**. It is roughly an afternoon of work, it is the
highest-value change (the kernel becomes buildable and runnable on any machine,
including Windows), and it de-risks every phase that follows.
