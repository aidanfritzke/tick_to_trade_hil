# Plan: Tick-to-Trade HIL

**BLUF:** Build a hardware-in-the-loop tick-to-trade system in which a Basys 3 FPGA receives replayed ITCH market data over Ethernet, maintains an order book, and sends OUCH orders to a PC exchange simulator. End-to-end latency is measured and compared against software (PC) and microcontroller (ESP32-C5) baselines. The C++ implementation from Phase 1 is the reference model that every hardware stage is verified against.

## System Roles

| Node | Role |
|---|---|
| Linux PC | Exchange simulator (ITCH replay as MoldUDP64, matching engine); software trading host (C++ reference model, strategy); measurement station (NIC hardware timestamps); development tools (Vivado, Verilator, cocotb) |
| Basys 3 + LAN8720 | Hardware trading engine: Ethernet MAC and UDP, ITCH parser, order book, risk checks, strategy, latency counters |
| ESP32-C5 | Control plane: Wi-Fi dashboard, remote kill switch, live parameter changes; CPU data point for latency comparison |

## Phases

| # | Phase | Scope | Exit Criterion |
|---|---|---|---|
| 1 | Software baseline | C++ ITCH parser and order book | Processes a full-day ITCH file; adopted as reference model |
| 2 | Ethernet bring-up | PHY ID over MDIO, 100 Mb/s link, raw frame RX/TX with CRC check | FPGA echoes frames back to the PC |
| 3 | Feed handler (RTL) | UDP, MoldUDP64, ITCH decode | GHDL/cocotb output matches reference model on replayed data |
| 4 | Order book (RTL) | Book in block RAM | Best bid/offer updates sent to PC match reference model |
| 5 | Close the loop | PC matching engine, OUCH order entry | Orders flow FPGA to simulator; fills return to FPGA |
| 6 | Latency measurement | FPGA cycle counters combined with PC hardware timestamps | Latency histograms including tail percentiles |
| 7 | Risk and control | Pre-trade risk checks; ESP32-C5 dashboard and kill switch | Out-of-limit orders rejected; kill switch halts order flow; parameters update live |
| 8 | Advanced | FPGA strategy; PC strategy preloading orders for FPGA to fire; AF_XDP vs. standard sockets; PTP time sync | Each item benchmarked against the Phase 6 baseline |

## Verification Approach

- **Reference model:** RTL blocks are checked against the Phase 1 C++ model, first in simulation (Verilator/cocotb), then on hardware with PC replay.
- **Latency:** The same data set and timestamp method are used for FPGA, PC, and ESP32-C5 comparisons.
