# FPGA

Index of my FPGA / RTL / hardware-adjacent projects. Each links out to its own repo.

## Projects

| Project | Description | Status |
|---|---|---|
| [01-binary-counter-sync-reset](https://github.com/XhovaniM8/01-binary-counter-sync-reset) | Binary counter with synchronous reset via button, built on the Arty Z7-10 | Done |
| [02-arty-pong](https://github.com/XhovaniM8/02-arty-pong) | VGA Pong on the Arty Z7-10 + Digilent Pmod VGA (RGB444, 1024x768@60Hz) | In progress |
| [fpga-orderbook-analysis](https://github.com/XhovaniM8/fpga-orderbook-analysis) | NYU FRE capstone — LSTM trained on order-book data, quantized (PyTorch + Brevitas), compiled via Xilinx FINN, targeting a Zynq-7020 to detect market behaviors (spoofing, liquidity sweeps, momentum ignition) in real time | Blocked on an ONNX shape-compatibility issue in FINN |
| [riscv16](https://github.com/XhovaniM8/riscv16) | 16-bit RiSC-16 processor — ALU + Verilator/CMake testbench verified via GTKWave | Top-level integration not yet complete |
| [sram-bist](https://github.com/XhovaniM8/sram-bist) | SRAM Built-In Self-Test (256x4-bit) with March C-/March A/Checkerboard patterns — NYU ECE 6443 VLSI Systems & Architecture project, synthesized on ASAP7 7nm via Genus | Done |
| [a-very-simple-switch-fabric](https://github.com/XhovaniM8/a-very-simple-switch-fabric) | Parameterizable SystemVerilog network switch fabric — per-input FIFOs, priority arbitration, Icarus Verilog + Verilator testbenches | Done |
| [boerboel](https://github.com/XhovaniM8/boerboel) | VHDL coding-test-style project — AXI-stream module prepending an Ethernet header to a streaming payload at 250MHz | RTL + testbench present |

## Related

- [eXSimulator](https://github.com/XhovaniM8/eXSimulator) — C++20 limit order book matching engine (not FPGA, but feeds the order-book analysis project above)

---
*Note: some links above use the repo-name convention from my project notes — if a name has drifted since, update the link accordingly.*
