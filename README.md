# C++ and RTL for HFT

Learning C++ and RTL for HFT on real hardware

## Hardware

Linux PC: Exchange simulator (replays ITCH data as MoldUDP64 packets and runs a matching engine), software trading host (the C++ reference model and strategy code), measurement station (network-card hardware timestamps), and development tools (Vivado, Verilator, cocotb).
Digilent Basys3 with Waveshare LAN8720 ETH Board: Hardware trading engine: Ethernet MAC and UDP handling, ITCH parser, order book, risk checks, strategy, and latency counters.
ESP32-C5: Control plane: a Wi-Fi dashboard, a remote kill switch, and live parameter changes. It also serves as the CPU data point in latency comparison.

## Build phases

1. Software baseline. Write a C++ ITCH parser and order book on the PC that processes full-day files. This becomes reference model.
2. Ethernet bring-up. Read the PHY's ID over MDIO, confirm a 100 Mb/s link, then receive and send raw frames with CRC checking. Finish with the FPGA echoing frames back to the PC.
3. Feed handler in RTL. Add UDP, MoldUDP64, and ITCH decoding, and verify them in Verilator or cocotb against the reference model while the PC replays data.
4. Order book. Keep the book in block RAM and send best bid/offer updates back to the PC.
5. Close the loop. Add the PC matching engine and OUCH order entry, so orders flow from the FPGA to the exchange simulator and fills come back.
6. Latency measurement. Combine FPGA cycle counters with the PC's hardware timestamps, and build latency histograms that include tail percentiles.
7. Risk and control. Add the pre-trade risk checks, then connect the ESP32-C5 dashboard and kill switch.
8. Advanced. Move to strategy logic in the FPGA, strategy on the PC that preloads orders for the FPGA to fire, an AF_XDP versus standard-socket comparison, and PTP time sync.