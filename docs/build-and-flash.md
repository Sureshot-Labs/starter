<!-- SPDX-License-Identifier: MIT -->
# 2. Build & Flash

Firmware is built with the SDK's `Makefile`, which orchestrates `cargo build`,
LLVM binary tools, and two Python scripts that **sign** the binary and pack it
as a **UF2** image. The board runs signed UF2 blobs copied to its USB
mass-storage volume.

> These commands are run from a checkout of the SDK, **not** this
> getting-started repo:
>
> ```bash
> git clone https://github.com/samblenny/baochip-sdk.git
> cd baochip-sdk
> ```
>
> The `examples/*.rs` and `examples/hello_c.c` files in *this* repo are copies
> of the SDK's examples so you can read them first.

---

## The build pipeline

Every example target runs the same conceptual stages:

1. **Compile** — `cargo build` (Rust) and/or `riscv64-unknown-elf-gcc` (C) →
   an ELF.
2. **Extract** — `llvm-objcopy -O binary` pulls the loadable sections into a
   flat `.bin`.
3. **Sign** — `signer.py` wraps the `.bin` in a signed firmware blob (`.img`).
4. **Pack** — `uf2ify.py` converts the signed blob into a `.uf2`.
5. **Copy** — the resulting `.uf2` is placed in `examples/`.

You don't run these by hand — the Makefile targets below do it for you.

## Build a Rust example

```bash
make blinky      # examples/blinky.rs  -> examples/blinky.uf2
make uart        # examples/uart.rs    -> examples/uart.uf2
make timer0      # examples/timer0.rs  -> examples/timer0.uf2
```

## Build the C example (C linked to Rust drivers)

```bash
make hello_c     # examples/hello_c.c  -> examples/hello_c.uf2
```

`hello_c` compiles the C application, archives it, builds the Rust SDK library
(`libbaochip_sdk.a`), links them together with picolibc, then signs and packs
the result. It requires the C toolchain from
[setup.md](setup.md).

## Build just the SDK static library (for your own C project)

```bash
make             # builds target/.../release/libbaochip_sdk.a
```

Then in your own C code, `#include "baochip_sdk.h"` and add a
`-I<path-to-header>` to your compiler flags. The C FFI surface is documented in
[`../examples/baochip_sdk.h`](../examples/baochip_sdk.h).

## Clean

```bash
make clean
```

---

## Flash the UF2 to the board

When you plug the Dabao into USB (in bootloader mode) it presents a mass-storage
volume named **`BAOCHIP`**. Flashing is a file copy.

### macOS

```bash
cp examples/blinky.uf2 /Volumes/BAOCHIP && sync
diskutil unmountDisk /dev/disk4     # replace disk4 with your BAOCHIP disk
```

Find the right disk number with `diskutil list` (look for the `BAOCHIP`
volume). After the disk unmounts, **press the PROG button** to run the code
(assuming the bootloader has `bootwait` enabled).

### Linux

Mount the `BAOCHIP` volume (your desktop environment usually auto-mounts it),
copy the `.uf2`, `sync`, unmount, then press PROG:

```bash
cp examples/blinky.uf2 /media/$USER/BAOCHIP && sync
umount /media/$USER/BAOCHIP
```

---

## Watching serial output

The Rust `uart`/`timer0` examples and the C `hello_c` example print over the
**UART debug console** (not USB CDC). You need:

- A USB-serial adapter that can do **1 Mbaud** (FTDI, CP2102/CP2102N, or a
  Raspberry Pi Debug Probe all work).
- Wiring (adapter → Dabao): **TX → PB13 (RX)**, **RX → PB14 (TX)**, **GND → GND**.

Then open a 1 Mbaud monitor:

```bash
screen -fn /dev/ttyUSB0 1000000     # Linux; adjust the device path
```

> **macOS caveat:** `screen` cannot set the non-standard 1 Mbaud rate. Use a
> monitor that uses the `IOSSIOSPEED` ioctl — e.g. Homebrew
> [`picocom`](https://formulae.brew.sh/formula/picocom) or
> [`hbaud`](https://github.com/samblenny/hbaud).

You should see the bootloader print `boot0 console up` and then your app's
output. More detail — including bootloader updates for early boards — is in
[3. Resources & Cross-References](resources.md).
