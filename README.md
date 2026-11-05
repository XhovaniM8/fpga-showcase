# FPGA Showcase

Index of my FPGA / RTL / hardware-adjacent projects. Each links out to its own repo.

The roadmap is built to master FPGA fundamentals, not to fake a trading stack. Each project is a small module, built properly, verified properly, and measured on hardware. The trading pipeline (feed handler → order book → risk gate) is the context that ties them together.

## Hardware

| Board | Notes |
|---|---|
| Digilent Arty Z7-10 (XC7Z010) | Main dev board. Ethernet and USB-UART are on the PS side; PL has HDMI in/out, PWM audio, 2× Pmod |
| AMD Kria KV260 (K26, Zynq UltraScale+) | Ubuntu on the PS; 1 GbE (PS only) |
| Raspberry Pi 5 | SPI/UART host for PL-side debug interfaces |
| Netgear unmanaged GbE switch | PC ↔ boards network |

## Projects

| Project | Description | Status |
|---|---|---|
| [vending-machine-controller-vhdl](https://github.com/XhovaniM8/vending-machine-controller-vhdl) | Finite state machine vending machine controller on Nexys 3 | Done |
| [01-binary-counter-sync-reset](https://github.com/XhovaniM8/01-binary-counter-sync-reset) | Binary counter with synchronous reset via button, built on the Arty Z7-10 | Done |
| [02-arty-pong](https://github.com/XhovaniM8/02-arty-pong) | VGA Pong on the Arty Z7-10 + Digilent Pmod VGA (RGB444, 1024x768@60Hz) | In progress |

## Fundamentals

What each fundamental means in practice, and where it gets proven.

| Fundamental | Mastery looks like | Proven in |
|---|---|---|
| **CDC** | Picks the right crossing for level, pulse, bus, or stream; can explain why the wrong one fails | 03, 04, 09 |
| **Reset strategy** | Async assert / sync deassert per domain; knows when not to reset a flop | 03 |
| **FSM design** | Clean, glitch-free FSMs for serial protocols and handshakes | 04, 05 |
| **Timing closure** | Hits a stated Fmax; reads the critical path; fixes it by pipelining or restructuring, with before/after WNS | 06, 08, 10 |
| **Constraint writing** | Every clock defined; every CDC exception chosen deliberately (`set_false_path` vs `set_max_delay -datapath_only`); IO timing | 03, 04, 06 |
| **Streaming datapaths** | AXI-Stream valid/ready; processes data as it arrives (no whole-packet buffering); correct under backpressure | 07, 09 |
| **Memory architecture** | BRAM inference, dual-port use, read-modify-write hazards handled with forwarding | 08 |
| **Protocol parsing** | Big-endian field extraction, framing, sequence-gap detection, malformed-input handling | 07, 09, rmii-mac |
| **Ethernet without vendor IP** | MAC framing, preamble/SFD, CRC32, interframe gap written in RTL | rmii-mac |
| **HW/SW interface** | Register maps, AXI-Lite, DMA, a userspace driver, host-side tooling in C++/Python | 05, 09, 12 |
| **Verification** | Self-checking tests against a reference model, constrained-random stimulus, coverage, fault injection | All; centerpieces 03, 07, 08 |
| **Deterministic latency** | Fixed, documented cycle counts; no data-dependent stalls on the critical path | 08, 10 |
| **Latency measurement** | Cycle-counter timestamps at each stage; hardware timestamps at the wire; honest end-to-end numbers | 09, 12 |
| **Hardware debug** | ILA captures, on-chip error counters, long soak tests | 03, 09 |

## Definition of done

Every project ships with:

- SystemVerilog RTL. No vendor IP unless integrating it is the point (PS7, AXI DMA)
- Self-checking cocotb testbench against a reference model, with constrained-random stimulus and a Verilator coverage report
- XDC with every clock and every timing exception explained
- Timing report: target Fmax, WNS/WHS, critical path described
- Utilization: LUT / FF / BRAM / DSP
- Latency in cycles, where it applies
- Hardware demo video
- Write-up: design decisions, what broke and why, what I'd change

## Roadmap

Ordered so each project builds on the ones before it.

### Phase 1: Core primitives and interfaces (Arty Z7-10)

| # | Project | Builds | Fundamentals | Extra hardware |
|---|---|---|---|---|
| 03 | cdc-primitives | 2-FF synchronizer, reset synchronizer, pulse synchronizer, req/ack handshake, debouncer, gray-code async FIFO. Metastability injection in sim; broken variants shown failing; multi-hour soak across PS and PL clocks | CDC, Reset, Constraints, Verification, Debug | None |
| 04 | uart | UART TX/RX with an oversampled receiver, looped back against the Pi 5's GPIO UART. Becomes the PL debug channel | FSM, CDC, Constraints | Pi 5 + jumpers |
| 05 | spi-regbank | SPI slave + register bank (status counters, config registers), with the Pi 5 as master | FSM, HW/SW interface | Pi 5 |
| 06 | hdmi-pong | TMDS encoder + OSERDESE2 serializers; port Pong from Pmod VGA to the onboard HDMI out | Timing closure, Constraints (IO timing) | None |

### Phase 2: Market data pipeline (Arty Z7-10 + PC)

| # | Project | Builds | Fundamentals | Extra hardware |
|---|---|---|---|---|
| 07 | itch-parser | `itch_parser.sv` reworked as an AXI-Stream parser that extracts big-endian fields as bytes stream in. Verified against a Python ITCH 5.0 decoder on the public NASDAQ sample file, with random backpressure and coverage on every message type | Streaming, Parsing, Verification | None |
| 08 | order-book | BRAM-based order book (add / execute / cancel / delete → BBO) with read-modify-write hazard forwarding and a pipelined BBO tracker at a fixed cycle count. Target 200 MHz. Scoreboarded against [eXSimulator](https://github.com/XhovaniM8/eXSimulator) via an ITCH replay adapter. Sized for the XC7Z010 (60 BRAM36): 1–2 symbols | Memory, Timing closure, Deterministic latency, Verification | None |
| 09 | zynq-feed-handler | PC replays MoldUDP64-framed ITCH over UDP → PS lwIP → AXI DMA → PL sequence-gap detection, parser, and order book → BBO back over UDP. AXI-Lite control/status registers. C++ replay tool; Python latency analysis of T1–T4 cycle-counter timestamps | Streaming, Parsing, HW/SW interface, Latency measurement, Debug | Switch |
| 10 | risk-gate | Hardware pre-trade risk gate: position and notional limits plus a latched kill switch, with pipelined comparators at a fixed cycle count. Fault-injection tests (limit breach, overflow, kill mid-stream) | Deterministic latency, Timing closure, Verification | None |

### Phase 3: KV260 (needs Zynq UltraScale+ device support in Vivado)

| # | Project | Builds | Fundamentals | Extra hardware |
|---|---|---|---|---|
| 11 | kv260-remote-lab | KV260 hosts the Arty Z7: `openFPGALoader`, `ser2net`, reachable over Tailscale | Infrastructure | None |
| 12 | kv260-feed-handler | Port 09 to Linux: UIO/udmabuf userspace driver, PS Ethernet IEEE 1588 hardware timestamps for wire-arrival latency | HW/SW interface, Latency measurement | Switch |

### Optional

| Project | Builds | Fundamentals | Extra hardware |
|---|---|---|---|
| rmii-mac | Pure-RTL Ethernet MAC on a Pmod-attached RMII PHY; latency compared against the PS path from 09. **The only project covering Ethernet without vendor IP; worth promoting** | Ethernet without vendor IP, Parsing, CDC | LAN8720 module |
| quadrature-decoder | 4× decoder with synchronized inputs; rotary-encoder paddle for Pong | CDC, FSM | Digilent PmodENC |
| pwm-audio-dds | DDS tone generator + PWM / one-bit DAC on the onboard audio jack | DSP basics | None |

---
*Note: some links above use the repo-name convention from my project notes — if a name has drifted since, update the link accordingly.*
