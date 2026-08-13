<!-- SPDX-License-Identifier: MIT -->
# Getting Started with Baochip (Dabao / Bao1x)

A curated starting point for bare-metal development on the **Baochip Dabao**
evaluation board (Bao1x RISC-V SoC). This repo gathers the key resources,
walks through a working toolchain setup, and ships a few runnable examples so
you can go from an unboxed board to blinking an LED and printing over serial.

> **What this repo is:** a getting-started guide and example collection that
> cross-references the upstream drivers, board docs, and chip references.
> **What it is not:** the SDK itself. The drivers live in Sam Blenny's
> [baochip-sdk](https://github.com/samblenny/baochip-sdk) and the broader
> platform in [betrusted-io/xous-core](https://github.com/betrusted-io/xous-core).
> See [Attribution](#attribution).

---

## Contents

- [What is Baochip?](#what-is-baochip)
- [Quick Start (TL;DR)](#quick-start-tldr)
- [Documentation](#documentation)
  - [1. Environment Setup](docs/setup.md)
  - [2. Build & Flash](docs/build-and-flash.md)
  - [3. Resources & Cross-References](docs/resources.md)
- [Examples](#examples)
- [Hardware Notes](#hardware-notes)
- [Attribution](#attribution)

---

## What is Baochip?

**Baochip** is an open hardware project centered on the **Bao1x** RISC-V SoC.
The **Dabao** is the evaluation board built around that chip.

- **CPU:** RISC-V `rv32imac` core (target triple `riscv32imac-unknown-none-elf`)
- **Firmware model:** signed firmware blobs packed as **UF2**, copied to the
  board's USB mass-storage volume (`BAOCHIP`) and launched via the PROG button
- **Peripherals:** GPIO, UART, timers, and a Pulp-platform **uDMA** subsystem
  (UART / I2C / SPI / camera / SDIO / ADC), plus math/crypto accelerators and a TRNG
- **Programming languages:** drivers are `no_std` Rust with a C FFI layer, so
  you can write applications in **either Rust or C**

| Resource | Link |
| --- | --- |
| Baochip project site | https://baochip.com |
| Dabao board design files | https://github.com/baochip/dabao |
| Dabao v3 schematic (PDF) | https://github.com/baochip/dabao/blob/main/dabao_v3c.pdf |
| Bao1x SDK (drivers + examples) | https://github.com/samblenny/baochip-sdk |
| More complete C SDK (Dabao) | https://github.com/armstrongsubero/dabao-sdk |
| Xous platform (upstream) | https://github.com/betrusted-io/xous-core |

A fuller, annotated list is in [docs/resources.md](docs/resources.md).

---

## Quick Start (TL;DR)

Full detail lives in the docs pages linked below — this is the 60-second version.

```bash
# 1. Toolchain (Rust + RISC-V target + GCC/picolibc). See docs/setup.md.
rustup target add riscv32imac-unknown-none-elf

# 2. Clone the SDK (this repo's examples are copies of the SDK's examples).
git clone https://github.com/samblenny/baochip-sdk.git
cd baochip-sdk

# 3. Build an example → produces a signed .uf2
make blinky

# 4. Flash: copy the UF2 to the board's USB volume, then press PROG.
cp examples/blinky.uf2 /Volumes/BAOCHIP && sync   # macOS
```

Then jump to:

1. **[docs/setup.md](docs/setup.md)** — install Rust, the RISC-V target, GCC, and picolibc.
2. **[docs/build-and-flash.md](docs/build-and-flash.md)** — build a UF2 and flash it to the board.
3. **[docs/resources.md](docs/resources.md)** — datasheets, register docs, bootloader updates, serial console.

---

## Examples

Each example is a copy of an upstream `baochip-sdk` example, kept here so you
can read the code without cloning first. Build them from a checkout of the SDK
(`make <name>`) as described in [docs/build-and-flash.md](docs/build-and-flash.md).

| Example | Language | Demonstrates | Hardware needed |
| --- | --- | --- | --- |
| [`blinky.rs`](examples/blinky.rs) | Rust | GPIO output + heartbeat timer (`d11ctime`) | LED + 330–470Ω resistor on PB12 |
| [`hello_c.c`](examples/hello_c.c) | C | Calling Rust drivers from C over the FFI (`dbs_*`) | Serial adapter (1 Mbaud) |
| [`uart.rs`](examples/uart.rs) | Rust | UART TX via uDMA, `ticktimer`, PROG-button input | Serial adapter (1 Mbaud) |
| [`timer0.rs`](examples/timer0.rs) | Rust | Interrupt-driven timer callbacks (`timer0::set_alarm_ms`) | Serial adapter (1 Mbaud) |

The C FFI surface used by `hello_c.c` is declared in
[`examples/baochip_sdk.h`](examples/baochip_sdk.h).

See [examples/README.md](examples/README.md) for a walkthrough of each one.

---

## Hardware Notes

Read these **before** wiring anything to the board:

- ⚡🔥☠️ **DO NOT apply 5V.** The IO is **not** 5V tolerant. Use **3.3V**.
- GPIO: 3.3V IO, 12 mA drive, 2 kV HBM ESD protection.
- Pull-**up** is supported; pull-**down** is **not**.
- Debug serial console (bootloader): **TX = PB14**, **RX = PB13**, **1 Mbaud, 8N1**.
- ⚠️ Early boards (e.g. from 39C3) shipped with alpha firmware that **must be
  updated** — see the bootloader update instructions in
  [docs/resources.md](docs/resources.md).

---

## Attribution

This is a getting-started guide that packages and cross-references work by others:

- **baochip-sdk** — drivers, examples, and the C FFI header — © 2026 Sam Blenny,
  MIT licensed: https://github.com/samblenny/baochip-sdk
- **xous-core / bao1x-hal** — upstream platform, HAL, and signing tooling —
  betrusted-io: https://github.com/betrusted-io/xous-core
- **Dabao hardware** — https://github.com/baochip/dabao

The example source files in [`examples/`](examples/) retain their original
SPDX headers and copyright. This repository is MIT licensed (see
[LICENSE](LICENSE)).
