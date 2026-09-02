<!-- SPDX-License-Identifier: MIT -->
# 3. Resources & Cross-References

A curated, annotated index of everything worth bookmarking for HoloDi S1 / Dabao
development. Grouped by what you're trying to do.

---

## Start here

| Resource | What it is |
| --- | --- |
| [baochip.com](https://baochip.com) | Project site; jumping-off point for docs. |
| [samblenny/baochip-sdk](https://github.com/samblenny/baochip-sdk) | The `no_std` Rust SDK these examples come from: drivers, linker script, signing scripts, examples. |
| [armstrongsubero/dabao-sdk](https://github.com/armstrongsubero/dabao-sdk) | A more complete **C SDK** for Dabao. Consider this if your project is primarily C. |
| [betrusted-io/xous-core](https://github.com/betrusted-io/xous-core) | Upstream platform: HAL (`bao1x-hal`), bare-metal setup, and signing tools. |
| [xous-book: Hello World](https://betrusted.io/xous-book/ch01-02-hello-world.html) | Background on the Xous build/sign/flash model. |
| [xous-core/README-baochip.md](https://github.com/betrusted-io/xous-core/blob/main/README-baochip.md) | Baochip-specific build, signing, flashing, and bootloader notes. |

---

## Hardware & board

| Resource | What it is |
| --- | --- |
| [github.com/baochip/dabao](https://github.com/baochip/dabao) | Dabao board design files. |
| [dabao_v3c.pdf](https://github.com/baochip/dabao/blob/main/dabao_v3c.pdf) | Dabao v3 schematic. |

**Electrical rules (repeat of the README warnings — worth internalizing):**

- **3.3V IO only. Not 5V tolerant.** Applying 5V can destroy the chip.
- 12 mA drive strength, 2 kV HBM ESD protection.
- Pull-up supported; **pull-down is not**.
- Bootloader serial console: **TX = PB14, RX = PB13, 1 Mbaud 8N1**.

---

## Bootloader update (do this first on early boards)

⚠️ The first batch of boards (e.g. from 39C3) shipped with **alpha firmware
that must be updated** before the flashing flow above behaves as documented.

- **Instructions:**
  [xous-core/README-baochip.md](https://github.com/betrusted-io/xous-core/blob/main/README-baochip.md)
- **Prebuilt bootloader UF2s** (if you don't want to build `bao1x-alt-boot1.uf2`
  / `bao1x-boot1.uf2` from source) from bunnie's CI:
  [ci.betrusted.io/latest-ci/baochip/bootloader/](https://ci.betrusted.io/latest-ci/baochip/bootloader/)

---

## Chip / SoC internals (HoloDi S1 / `bao1x`)

| Topic | Reference |
| --- | --- |
| CPU cluster registers (IRQArray, timers, suspend/resume) | [ci.betrusted.io/bao1x-cpu](https://ci.betrusted.io/bao1x-cpu/) |
| Peripheral cluster registers (crypto, TRNG, UDMA, BIO, …) | [ci.betrusted.io/bao1x](https://ci.betrusted.io/bao1x/) |
| uDMA subsystem (UART/I2C/SPI/camera/SDIO/ADC) | [openhwgroup docs](https://docs.openhwgroup.org/projects/core-v-mcu/doc-src/udma_subsystem.html), [pulp-rt sample drivers](https://github.com/pulp-platform/pulp-rt/tree/master/drivers) |
| PLL / clock programming | [bao1x-hal/src/clocks.rs](https://github.com/betrusted-io/xous-core/blob/main/libs/bao1x-hal/src/clocks.rs) |
| BIO (timing-sensitive IO: I2S, 1-wire, LED strings) | [bao1x-hal/src/bio_hw.rs](https://github.com/betrusted-io/xous-core/blob/main/libs/bao1x-hal/src/bio_hw.rs) |

> **IRQArray** replaces a traditional NVIC: IRQs are split into banks of 16,
> each bank on its own memory page, so drivers can hold their own memory spaces
> (useful under virtual memory).

---

## Bare-metal boot, linking, and signing

| Topic | Reference |
| --- | --- |
| Linker script | [baremetal/.../bao1x/link.x](https://github.com/betrusted-io/xous-core/blob/main/baremetal/src/platform/bao1x/link.x) |
| Stack pointer / trap / entry setup | [baremetal/src/asm.rs](https://github.com/betrusted-io/xous-core/blob/944a8082ec235339e5e73165da48fd209f4a0724/baremetal/src/asm.rs#L35-L56) |
| Image signing tool (Rust) | [tools/src/bin/sign_image.rs](https://github.com/betrusted-io/xous-core/blob/main/tools/src/bin/sign_image.rs) |
| Signing script (PowerShell) | [baosign.ps1](https://github.com/betrusted-io/xous-core/blob/main/baosign.ps1) |
| Dev signing key / cert | [devkey/dev.key](https://github.com/betrusted-io/xous-core/blob/main/devkey/dev.key), [devkey/dev-x509.crt](https://github.com/betrusted-io/xous-core/blob/main/devkey/dev-x509.crt) |
| Public keys | [bao1x-api/src/pubkeys](https://github.com/betrusted-io/xous-core/tree/main/libs/bao1x-api/src/pubkeys) |

The `baochip-sdk` bundles its own `signer.py` and `uf2ify.py` so you don't need
the Rust `sign_image.rs` path for the default flow.

---

## Serial console tooling

You need a **1 Mbaud**-capable monitor for the UART debug console.

| Platform | Options |
| --- | --- |
| Linux | `screen -fn /dev/ttyUSB0 1000000` (or your device path). |
| macOS | `screen` **won't** do 1 Mbaud. Use [picocom](https://formulae.brew.sh/formula/picocom) or [hbaud](https://github.com/samblenny/hbaud) (both use the `IOSSIOSPEED` ioctl). |

Adapters known to work at 1 Mbaud: FTDI, CP2102 / CP2102N, Raspberry Pi Debug
Probe.

---

## Where to look in the HAL for a peripheral

When the SDK doesn't yet expose something, the upstream HAL is the reference
implementation:

- UART: [bao1x-hal/src/udma/uart.rs](https://github.com/betrusted-io/xous-core/blob/main/libs/bao1x-hal/src/udma/uart.rs)
- uDMA core: [bao1x-hal/src/udma/mod.rs](https://github.com/betrusted-io/xous-core/blob/main/libs/bao1x-hal/src/udma/mod.rs)
- Debug/logging: [bao1x-hal/src/debug.rs](https://github.com/betrusted-io/xous-core/blob/5eec0702f9a989144739ff08d419ed7445c2ecc9/libs/bao1x-hal/src/debug.rs)
- Clocks/PLL: [bao1x-hal/src/clocks.rs](https://github.com/betrusted-io/xous-core/blob/main/libs/bao1x-hal/src/clocks.rs)
- BIO: [bao1x-hal/src/bio_hw.rs](https://github.com/betrusted-io/xous-core/blob/main/libs/bao1x-hal/src/bio_hw.rs)
