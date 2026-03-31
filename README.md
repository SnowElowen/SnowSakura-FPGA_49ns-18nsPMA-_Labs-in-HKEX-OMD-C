# SnowSakura: Physical Layer Implementation Specs for 15EG  VU9P
## Target: 31.5ns Deterministic Latency for HKEX-OMD-C

### Physical Layer Design Philosophy

* **Determinism over Abstraction**: In the realm of 31.5ns latency, standard software stacks are nothing but propagation noise.
* **Hardware Sovereignty**: We bypass OS kernels, PCIe overhead, and standard IP blocks. Logic is mapped directly to the **GTH Transceiver** and dedicated **LUT** resources via manual routing.
* **Timing is Law**: Every clock cycle at 322.26MHz counts. If your logic takes more than 10 cycles, you've already lost the trade.

---
*(Detailed XDC constraints are kept in internal private labs due to proprietary physical optimization logic.)*
# HKEX-OMD-C 31.5ns Parser: Physical Layer Implementation Log
**Target Device**: Xilinx Zynq UltraScale+ XCZU15EG (FFVB1156)
**Operating Frequency**: 322.265625 MHz (GTH Raw Mode, Bypassing PCS)

---

## The Physical Truth of Zero-Jitter Trading

In HFT, software architecture is an illusion; only **Physical Layer Logic** dictates the outcome. The following logs document the three-stage manual routing and timing closure process for a 31.5ns OMDC parser. 

Relying on Vivado's Auto-Router for OMD-C parsing is a death sentence. To squeeze latency down to 31.5ns, every **LUT**, every **Register**, and every **Routing path** must be manually constrained via precise XDC definitions.

### Stage 1: Datapath Routing & Net Delay Suppression

*![Weixin Image_20260318204056_4_26](https://github.com/user-attachments/assets/d9c97f71-b2dd-4a0b-ab1d-b9e36a9b5e23)*


The raw battle between **Logic Delay** and **Net Delay**. When operating in the `gt_txusrclk2` domain at 322MHz, the propagation delay across the silicon is your biggest enemy.
* **Observation**: We manually forced the **Net Delay** to converge around 1.5ns - 1.6ns, keeping the **Logic Delay** strictly under 1ns (e.g., 0.973ns). 
* **Logic**: If you let the GUI decide your Placement, your Net Delay will spike, and your 31.5ns target will be shattered by routing congestion.

### Stage 2: Floorplanning & Initial Timing Closure

* ![Weixin Image_20260318204058_5_26](https://github.com/user-attachments/assets/914102f8-78b1-4161-87de-6fe76e48c760)*

Initial logic mapping and physical isolation. 
* **Timing Met**: **WNS (Worst Negative Slack)** secured at **0.708 ns**. **WHS (Worst Hold Slack)** tightly locked at **0.024 ns**.
* **Logic**: The logic cells (CLEM) are tightly packed to minimize interconnect latency. This is not arbitrary; this is the result of strict **Pblock** constraints. Zero failing endpoints mean the Triple-FF synchronization logic is physically solid.

### Stage 3: Full Pipeline Squeeze @ 322MHz

*![Weixin Image_20260318204100_6_26](https://github.com/user-attachments/assets/22b17bb2-21c9-4b91-9fdb-dabb24aca638)*

*![Weixin Image_20260318204101_7_26](https://github.com/user-attachments/assets/52906542-3e55-47e9-aceb-8c766849af79)*

As the parsing logic scales, the timing window shrinks to its absolute physical limit.
* **Timing Met**: **WNS** squeezed to **0.472 ns**, **WHS** at **0.030 ns**. 
* **Logic**: 0 Failing Endpoints across 542 endpoints. This proves the deterministic stability of the manual routing pipeline. We are pushing the Ultrascale+ architecture to its extreme edge without violating setup/hold times. 

---

### Proprietary Disclaimer
**Do not ask for the XDC scripts.** The exact coordinates, `set_property LOC/BEL` mappings, and Phase Interpolator calibration values are proprietary and isolated in private labs. What you see here is the physical result; the manual routing logic behind it remains classified.
### Phase 3 - Extreme RX-Parser-TX (Single Channel) Summary

*Oops, sorry guys, I simply forgot to include these waveforms and schematics in yesterday's push. Here's the final validation of the deterministic single-channel pipeline before we scale up to the dual-path arbiter architecture.*

#### I. Latency Validation: Waveform Snapshot
We are running `GTH Raw Mode` on the Ultrascale+ architecture, stripping away all non-essential protocol overhead (e.g., standard 802.3 buffers, PCS alignment primitives) for direct hardware parallel data access. 
<img width="1105" height="754" alt="Snipaste_2026-03-28_01-31-59" src="https://github.com/user-attachments/assets/bec0526e-c7d4-4c81-9426-657ddc7cd315" />



* **Highlight:** The cursor measurements demonstrate deterministic, extreme low-cycle latency from Start-of-Packet (SoP) detection directly to the Parser Output pulse. 

#### II. Implementation Details: Synthesis Schematic
This isn't generic RTL synthesis; this is **direct physical mapping**. We are manually configuring registers (`mock_gth_data_reg`) and logic gates to absolutely minimize interconnect routing delay at the silicon level.
<img width="1039" height="694" alt="Snipaste_2026-03-28_01-32-47" src="https://github.com/user-attachments/assets/93f19018-b87c-4863-b8d9-126920ec3e24" />

<img width="947" height="733" alt="Snipaste_2026-03-28_01-32-55" src="https://github.com/user-attachments/assets/d99bd1ff-3d88-4709-976e-d0e2044cfc39" />


* **Clock Tree:** `IBUFDS_GTE4` -> `BUFG_GT_SYNC`. Direct-driven reference clock path ensuring zero-latency clock enables across the 16nm die matrix.
* **Matrix Mapping:** Direct-mapped parallel registers to output pins with aggressive LUT-1 combinational bypass elements. We do not waste clock cycles waiting to propagate simple data mappings.

#### III. Static Timing Report Summary
As the parsing logic scales, the timing window shrinks to its absolute physical limit. The final synthesis proves deterministic stability under extreme constraint conditions.

* **Timing Constraints**: **Met**
* **Failing Endpoints**: **0** (Across all 542 endpoints)
* **Worst Negative Slack (WNS)**: **0.472 ns** (Setup)
* **Worst Hold Slack (WHS)**: **0.030 ns** (Hold)

> **Proprietary Disclaimer:** > **Do not ask for the XDC constraint scripts.** The exact `set_property LOC/BEL` coordinate mappings and Phase Interpolator calibration values are proprietary and isolated. What you see here is the physical result; the manual routing logic behind it remains classified.
