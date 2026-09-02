<!-- SPDX-License-Identifier: MIT -->
# 1. Environment Setup

The toolchain builds `rv32imac` firmware for the HoloDi S1 SoC. You need Rust (for the
drivers), a RISC-V GCC + picolibc (for C code and linking), and a few LLVM
binary tools (for post-build packaging).

> The upstream instructions target **Debian 13 (Trixie)** and recent Ubuntu
> LTS. Notes for **macOS** are included below, but Linux is the best-supported
> path.

---

## 1. Install Rust

Use the official rustup installer: https://rust-lang.org/learn/get-started/

If you'd rather not pipe `curl` into a shell, download the script, read it, and
run it yourself — and consider the "Customize installation" option to skip the
automatic `PATH` edit and set it yourself.

## 2. Add the RISC-V target

```bash
rustup target add riscv32imac-unknown-none-elf
```

## 3. Install the RISC-V GCC toolchain + picolibc

**Debian / Ubuntu:**

```bash
sudo apt install binutils-riscv64-unknown-elf \
  gcc-riscv64-unknown-elf \
  picolibc-riscv64-unknown-elf
```

On Debian, picolibc lands at:

- Includes: `/usr/lib/picolibc/riscv64-unknown-elf/include`
- Release libs (rv32imac / ilp32):
  `/usr/lib/picolibc/riscv64-unknown-elf/lib/release/rv32imac/ilp32`

These paths are referenced by the SDK's `Makefile`. On Ubuntu (or other
distros) they may differ — adjust the `-I` / library paths in the Makefile to
match your system.

**macOS:** there is no first-class apt-style package for
`gcc-riscv64-unknown-elf` + picolibc. Options:

- Install a prebuilt RISC-V GCC toolchain (e.g. the xPack
  `riscv-none-elf-gcc`, or via `brew`), then point the SDK Makefile's compiler
  and picolibc include/lib paths at it.
- Or do Rust-only examples (like `blinky.rs`), which don't need the C compiler
  or picolibc for the application layer.

## 4. LLVM binary tools

The Makefile uses `llvm-objcopy` / `llvm-objdump` to extract and inspect the
loadable sections. These ship with the LLVM toolchain:

- Rust already bundles them via `rustup component add llvm-tools-preview`
  (exposed as `llvm-objcopy` etc. inside the toolchain), **or**
- install a system LLVM (`sudo apt install llvm` / `brew install llvm`).

## 5. Python 3

Firmware signing and UF2 packing are done by two Python scripts in the SDK
(`signer.py`, `uf2ify.py`), invoked automatically by the Makefile. Any recent
Python 3 works; no extra packages are required for the default flow.

---

## Verify your setup

```bash
rustc --version
rustup target list --installed | grep riscv32imac      # should list the target
riscv64-unknown-elf-gcc --version                        # Linux / configured macOS
llvm-objcopy --version
python3 --version
```

Once these all succeed, continue to **[2. Build & Flash](build-and-flash.md)**.
