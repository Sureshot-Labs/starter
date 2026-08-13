<!-- SPDX-License-Identifier: MIT -->
# Examples Walkthrough

These files are copies of the `baochip-sdk` examples, kept here so you can read
them before cloning. Build and flash them from a checkout of the SDK — see
[../docs/build-and-flash.md](../docs/build-and-flash.md).

Build order suggestion: **blinky → uart → timer0 → hello_c**. Blinky needs no
serial adapter, so it's the fastest way to confirm your toolchain and board
are healthy.

---

## `blinky.rs` — GPIO + heartbeat timer (Rust)

Blinks an LED on **PB12** once per second.

- Configures PB12 as a GPIO output (`gpio::set_alternate_function` → `AF0`,
  `enable_output`).
- Uses the **D11CTIME** heartbeat timer as a 1-second tick: sets the interval
  with `d11ctime::set_interval(millis_to_cycles(1000))` and toggles the LED
  each time `read_heartbeat()` changes.

**Wiring:** LED anode → PB12, through a **330–470Ω** resistor to GND.
(Remember: **3.3V only**, pull-down not supported.)

**Build:** `make blinky`

---

## `hello_c.c` — calling Rust drivers from C

Prints `Hello, world! (from C; i=N)` over UART every 5 seconds.

- Pure C application logic that calls into the Rust SDK via the C FFI
  (`dbs_uart_write`, `dbs_timer_sleep_ms`) — declared in
  [`baochip_sdk.h`](baochip_sdk.h).
- The Rust side handles hardware init, the UART driver, and the entry point; it
  calls the C `main()` after boot.
- **Key constraint:** `main()` must **never return** — the Rust `_start()`
  expects it to loop forever.

**Build:** `make hello_c` (requires the C toolchain from
[../docs/setup.md](../docs/setup.md)).
**Watch:** 1 Mbaud serial monitor.

---

## `uart.rs` — UART TX over uDMA + button input (Rust)

Prints `hello, world! [millis() = …]` each time you press and release the PROG
button.

- Reads the **PROG button (PC13)** as an input with pull-up.
- Uses the `log!` macro over UART2 (initialized at boot in `crate::init()`).
- Calls `uart::tick()` in the wait loops to service the DMA TX queue, and
  `sleep(10)` to debounce.
- Reads the millisecond clock via `ticktimer::millis()`.

**Build:** `make uart`
**Watch:** 1 Mbaud serial monitor.

---

## `timer0.rs` — interrupt-driven timer callback (Rust)

Prints `beep <t>...` then, ~2 s later, `boop <t>` from an **interrupt
callback**. Subsequent beep/boop pairs are triggered by the PROG button.

- Arms a 2-second alarm with `timer0::set_alarm_ms(2000, alarm_callback)`.
- `alarm_callback()` runs in **interrupt context** — it's kept tiny (just a
  `log!`). TIMER0 counts down at ACLK (350 MHz) and fires the IRQ at zero.
- Demonstrates the trap handler invoking a stored Rust callback.

**Build:** `make timer0`
**Watch:** 1 Mbaud serial monitor.

Example output:

```text
beep 5799...boop 7798
beep 12224...boop 14224
```

---

## `baochip_sdk.h` — the C FFI surface

The C-callable declarations exposed by the Rust SDK. Functions use the `dbs_`
(dabao-sdk) prefix:

- `dbs_uart_read_char()`, `dbs_uart_write(data, len)`, `dbs_uart_tick()`
- `dbs_timer_sleep_ms(ms)`, `dbs_timer_millis()`

GPIO FFI functions are noted as a future addition in the header. Include this
header (and add `-I<path>`) when linking your own C code against
`libbaochip_sdk.a`.
