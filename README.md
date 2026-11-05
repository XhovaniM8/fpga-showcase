# FPGA Showcase

Index of my FPGA / RTL / hardware-adjacent projects. Each links out to its own repo.

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

## Roadmap

Ordered so each project builds on the ones before it. Every project ships with RTL, a testbench, constraints, a hardware demo video, and a write-up.

### Phase 1: Core primitives and interfaces (Arty Z7-10)

| # | Project | Covers | Extra hardware |
|---|---|---|---|
| 03 | cdc-primitives | 2-FF synchronizer, pulse synchronizer, req/ack handshake, debouncer, gray-code async FIFO. Each has a cocotb testbench and XDC constraints | None |
| 04 | uart | UART TX/RX with an oversampled receiver, looped back against the Pi 5's GPIO UART. Becomes the PL debug channel | Pi 5 + jumpers |
| 05 | spi-regbank | SPI slave + register bank (status counters, config registers), with the Pi 5 as master | Pi 5 |
| 06 | hdmi-pong | TMDS encoder + OSERDESE2 serializers; port Pong from Pmod VGA to the onboard HDMI out | None |

### Phase 2: Market data pipeline (Arty Z7-10 + PC)

| # | Project | Covers | Extra hardware |
|---|---|---|---|
| 07 | itch-parser | cocotb verification of `itch_parser.sv` against a Python ITCH 5.0 decoder, replaying the public NASDAQ ITCH sample file | None |
| 08 | order-book | SystemVerilog order book (add / execute / cancel / delete → BBO), scoreboarded against [eXSimulator](https://github.com/XhovaniM8/eXSimulator) via an ITCH replay adapter. Sized for the XC7Z010 (60 BRAM36): 1–2 symbols | None |
| 09 | zynq-feed-handler | PC replays ITCH over UDP → PS lwIP → AXI DMA → PL parser + order book → BBO back over UDP. T1–T4 cycle-counter timestamps | Switch |
| 10 | risk-gate | Hardware pre-trade risk gate: position and notional limits plus a kill switch enforced in the order path | None |

### Phase 3: KV260 (needs Zynq UltraScale+ device support in Vivado)

| # | Project | Covers | Extra hardware |
|---|---|---|---|
| 11 | kv260-remote-lab | KV260 hosts the Arty Z7: `openFPGALoader`, `ser2net`, reachable over Tailscale | None |
| 12 | kv260-feed-handler | Port 09 to Linux: UIO/udmabuf userspace driver, PS Ethernet IEEE 1588 hardware timestamps for wire-arrival latency | Switch |

### Optional

| Project | Covers | Extra hardware |
|---|---|---|
| rmii-mac | Pure-RTL Ethernet MAC on a Pmod-attached RMII PHY; latency compared against the PS path from 09 | LAN8720 module |
| quadrature-decoder | 4× decoder with synchronized inputs; rotary-encoder paddle for Pong | Digilent PmodENC |
| pwm-audio-dds | DDS tone generator + PWM / one-bit DAC on the onboard audio jack | None |

---
*Note: some links above use the repo-name convention from my project notes — if a name has drifted since, update the link accordingly.*
