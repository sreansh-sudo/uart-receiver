# UART Receiver — Verilog

A synthesizable UART receiver implemented in Verilog with a self-checking testbench. Verified using Verilator simulation.

---

## Files

| File | Description |
|------|-------------|
| `uart_receiver.v` | RTL implementation of the UART receiver |
| `uart_receiver_tb.v` | Self-checking testbench with VCD dump |

---

## Design Overview

The receiver is built as a 4-state FSM:

```
IDLE → START → RECV → STOP → IDLE
```

| State | Behaviour |
|-------|-----------|
| **IDLE** | Waits for start bit (line goes low) |
| **START** | Samples at mid-bit to reject glitches |
| **RECV** | Shifts in 8 data bits, LSB first |
| **STOP** | Validates stop bit; asserts `data_ready` on framing success |

**Key features:**
- Two-flop synchronizer on `serial_in` to prevent metastability
- Parameterizable baud rate via `CLKS_PER_BIT`
- Framing check — `data_ready` is only asserted when a valid stop bit is detected
- Counter width auto-sized using `$clog2` to prevent silent overflow

---

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `CLKS_PER_BIT` | `217` | System clock cycles per UART bit period |

For a 25 MHz clock and 115200 baud: `CLKS_PER_BIT = 25_000_000 / 115200 ≈ 217`

---

## Ports

| Port | Direction | Width | Description |
|------|-----------|-------|-------------|
| `clk` | input | 1 | System clock |
| `rst_n` | input | 1 | Active-low synchronous reset |
| `serial_in` | input | 1 | UART RX line |
| `data_out` | output | 8 | Received byte |
| `data_ready` | output | 1 | Pulses high for one cycle when a byte is ready |

---

## Simulation

### Requirements
- [Verilator](https://verilator.org/) v5.x
- GTKWave (for waveform viewing)
- make

### Build and Run

```bash
verilator --binary -j 0 -Wall uart_receiver.v uart_receiver_tb.v \
  --top uart_receiver_tb --timing --CFLAGS "-std=c++20" --trace

cd obj_dir
make -f Vuart_receiver_tb.mk Vuart_receiver_tb
./Vuart_receiver_tb
```

### View Waveforms

```bash
gtkwave uart_receiver_tb.vcd
```

### Expected Output

```
RX[1] @ ... ns = 0x3c
RX[2] @ ... ns = 0x2f
Final data_out = 0x2f
PASS: Two bytes received successfully
```

---

## Testbench Details

- Sends two UART frames: `0x3C` and `0x2F`
- Monitors `data_ready` and logs each received byte with a timestamp
- Checks that exactly 2 bytes were received and prints PASS/FAIL
- Dumps full waveform to `uart_receiver_tb.vcd`
