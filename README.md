# Deadlocked

A from-scratch **32-bit x86 operating system kernel**, written in C and assembly. Deadlocked boots on bare metal (or QEMU) via a Multiboot-compliant bootloader, brings the CPU up into protected mode, and runs a small Unix-like userspace — multiple terminals, preemptive scheduling, a read-only filesystem, and a handful of shell programs including an animated `fish`.

> ⚠️ This is an educational kernel. It targets the legacy i386 architecture and is meant for learning OS internals, not production use.

## Features

- **Protected-mode boot** — Multiboot header, GDT/IDT setup, segment configuration, and jump to the C kernel.
- **Interrupt handling** — a full IDT with the 8259 PIC, the Programmable Interval Timer (PIT), the Real-Time Clock (RTC), and a PS/2 keyboard driver.
- **Virtual memory** — paging with a 4 MB kernel page and per-process address spaces.
- **Processes & system calls** — `execute`, `halt`, `read`, `write`, `open`, `close`, `getargs`, and `vidmap`, backed by per-process control blocks and full context switching.
- **Read-only filesystem** — an indexed file format (boot block + inodes + data blocks) loaded from a filesystem image at boot.
- **Multiple terminals** — three independent terminals with their own video memory and keyboard buffers, switchable on the fly.
- **Preemptive scheduling** — round-robin scheduling across active terminals driven by the PIT.

## Userspace programs

The bundled filesystem image ships several programs that exercise the kernel's system-call interface:

| Program    | Description                                  |
|------------|----------------------------------------------|
| `shell`    | Interactive command shell                    |
| `ls`       | List files in the filesystem                 |
| `cat`      | Print a file's contents                      |
| `grep`     | Search files for a pattern                   |
| `fish`     | ASCII fish animation (uses the RTC)          |
| `hello`    | Prompt-and-greet demo                        |
| `counter`  | Counting demo                                |
| `pingpong` | Timer-driven animation demo                  |

## Repository layout

```
student-distrib/   The kernel source (boot, paging, IDT, drivers, syscalls, scheduler, filesystem)
syscalls/          The userspace system-call library and the bundled programs
fish/              Source for the fish animation program
fsdir/             The source directory the filesystem image is built from
createfs           Tool: build a filesystem image from a flat directory
elfconvert         Tool: convert a 32-bit ELF executable to the kernel's executable format
```

## Building and running

> The current build process is Linux-only and root-dependent; making it portable
> and reproducible is the first item on the modernization roadmap — see
> [PLAN.md](PLAN.md).

```sh
cd student-distrib
make dep          # generate dependency info
sudo make         # build the kernel (bootimg) and assemble the boot image
```

Then boot `bootimg` under QEMU (see `student-distrib/INSTALL`).

## Roadmap

A detailed review and modernization plan — portable builds, a pinned cross-compiler
toolchain, CI, and an in-browser demo — lives in [PLAN.md](PLAN.md).

## Credits

Built as a team project by Navid, Benson, Saleh, Bala, and collaborators. The
`buglog.txt` is a candid record of the bugs fought (and won) along the way.
