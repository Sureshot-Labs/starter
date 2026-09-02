<!-- SPDX-License-Identifier: MIT -->
# Getting Started with the HoloDi S1 SoC

A curated starting point for bare-metal development on the **HoloDi S1 SoC**,
using the **Baochip Dabao** evaluation board. This repo gathers the key resources,
walks through a working toolchain setup, and ships a few runnable examples so
you can go from an unboxed board to blinking an LED and printing over serial.

## Naming: HoloDi S1 and Baochip-1x

The **HoloDi S1 SoC** is Sureshot Labs' productisation of the open-source
**Baochip-1x** design, published under the **CERN-OHL-W-2.0** licence.
"Baochip-1x" and "bao1x" are the upstream identifiers, and they appear
verbatim in repository paths, register names, tool paths and file names
throughout this guide — because you need the real identifiers to reproduce
anything stated here. They refer to the same silicon as "HoloDi S1 SoC".

Sureshot Labs' role is productisation, sustained support and certification of
that design. Because the design stays open, every page here can point straight
at the complete upstream source (see [Attribution](#attribution) and
[docs/resources.md](docs/resources.md)).

---

> **What this repo is:** a getting-started guide and example collection that
> cross-references the upstream drivers, board docs, and chip references.
> **What it is not:** the SDK itself. The drivers live in Sam Blenny's
> [baochip-sdk](https://github.com/samblenny/baochip-sdk) and the broader
> platform in [betrusted-io/xous-core](https://github.com/betrusted-io/xous-core).
> See [Attribution](#attribution).

---

## Contents

- [What is the HoloDi S1?](#what-is-the-holodi-s1)
- [Quick Start (TL;DR)](#quick-start-tldr)
- [Documentation](#documentation)
  - [1. Environment Setup](docs/setup.md)
  - [2. Build & Flash](docs/build-and-flash.md)
  - [3. Resources & Cross-References](docs/resources.md)
- [Examples](#examples)
- [Hardware Notes](#hardware-notes)
- [Attribution](#attribution)

---

## What is the HoloDi S1?

The **HoloDi S1** is a RISC-V SoC — upstream, the open hardware **Baochip-1x**
(`bao1x`) design. The **Dabao** is the evaluation board built around that chip.

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

This is a getting-started guide that packages and cross-references work by
others. Nothing here relicenses that work.

| Upstream | What this repo uses | Licence |
| --- | --- | --- |
| [baochip/dabao](https://github.com/baochip/dabao) — the Baochip-1x design and Dabao board | The hardware this guide targets: pinout, schematic, electrical limits | **CERN-OHL-W-2.0** |
| [samblenny/baochip-sdk](https://github.com/samblenny/baochip-sdk) — © 2026 Sam Blenny | The files in [`examples/`](examples/), copied verbatim, and the C FFI header | **MIT** |
| [betrusted-io/xous-core](https://github.com/betrusted-io/xous-core) — `bao1x-hal`, bare-metal platform, signing tooling | Referenced only; no code copied | **Apache-2.0** |

**Which licence covers what:**

- This repository's own material — the README, [`docs/`](docs/) — is MIT; see
  [LICENSE](LICENSE).
- The files in [`examples/`](examples/) are Sam Blenny's under MIT and keep his
  copyright. `blinky.rs`, `uart.rs`, `timer0.rs` and `baochip_sdk.h` carry his
  original SPDX headers verbatim; `hello_c.c` carries none because it has none
  upstream, so [LICENSE](LICENSE) names it explicitly instead.
- The **HoloDi S1 / Baochip-1x design itself is CERN-OHL-W-2.0** and is *not*
  covered by this repository's MIT licence. Sureshot Labs productises,
  supports and certifies that design; the complete source for it stays with
  the upstream projects above.
