### Optimizing Zynq-7000 PS-GEM Ethernet Performance for High-Bandwidth SDR Applications

The Zynq-7000 Processing System (PS) Gigabit Ethernet Controller (GEM) features a hard-coded **4KB Receive FIFO**.

Instead of standard 4K/9K/16K Jumbo Frames, we must use an intermediate MTU.
* **Value:** **2048 bytes**.
* **Reasoning:** A 2KB packet fits comfortably (2x) within the Zynq's 4KB FIFO, preventing hardware overruns, while reducing the interrupt rate by **~27%** compared to MTU 1500.
* **Requirement:** The host PC must use a network adapter that supports granular MTU setting (e.g., **ASIX AX88179** supports 2048, whereas Realtek 5GbE/2.5GbE drivers often force 4K jumps).

For board designers planning the next iteration of LibreSDR/Zynq SDRs, the current connection of the Ethernet PHY to **MIO Pins** is the limiting factor (see xapp1082-zynq-eth). 

The default Linux network buffers are too small for bursty SDR traffic. The following parameters should be applied at boot (via `init.d` or `sysctl.conf`):

```bash
# Increase backlog to handle CPU scheduling latency (Default is often 1000)
sysctl -w net.core.netdev_max_backlog=30000
# Increase RX/TX Windows to ~50MB
sysctl -w net.core.rmem_max=52428800
sysctl -w net.core.rmem_default=31457280
sysctl -w net.core.wmem_max=52428800
sysctl -w net.core.wmem_default=31457280
```