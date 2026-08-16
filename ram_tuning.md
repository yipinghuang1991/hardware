# RAM Tuning Manual

## References

- [[DDR4 OC Guide|DDR4 OC Guide]] — integralfx/MemTestHelper — [source](https://github.com/integralfx/MemTestHelper/blob/oc-guide/DDR4%20OC%20Guide.md)
- [[Infinity Fabric Overclocking on Zen2_3|Infinity Fabric Overclocking on Zen2/3]] — /u/RonLazer — [source](https://docs.google.com/document/d/1FsUuYtjztbqgOiR3uUCtzlTyzB2WRFUm-kXbboECj2s/edit?tab=t.0)

## Table of Contents

- [[#System Architecture & Environment]]
- [[#Voltages]]
    - [[#DRAM Voltage]]
    - [[#SoC Voltage (VDDCR_SOC)]]
    - [[#CLDO VDDP / VDDG CCD / VDDG IOD (Infinity Fabric Sub-Voltages)]]
- [[#Core Signal Control]]
    - [[#Infinity Fabric & Memory Controller (Pre-Foundation)]]
        - [[#FCLK (Infinity Fabric Frequency)]]
        - [[#UCLK (Memory Controller Frequency)]]
        - [[#ProcODT (Processor On-Die Termination)]]
    - [[#Structural & Non-Timing Prerequisites]]
        - [[#Gear Down Mode (GDM)]]
        - [[#Power Down Enable (PDE)]]
        - [[#Command Rate (Cmd2T)]]
- [[#Primary Timings]]
    - [[#tCL (CAS Latency)]]
    - [[#tRCDRd (RAS to CAS Delay - Read)]]
    - [[#tRCDWr (RAS to CAS Delay - Write)]]
    - [[#tRP (Row Precharge)]]
    - [[#tRAS (Active to Precharge Delay)]]
- [[#Secondary Timings]]
    - [[#Write Latency]]
        - [[#tCWL (CAS Write Latency)]]
    - [[#Holy Trinity]]
        - [[#tRRDS (Row Active to Row Active Delay, Short)]]
        - [[#tRRDL (Row Active to Row Active Delay, Long)]]
        - [[#tFAW (Four Activate Window)]]
    - [[#Bank Cycle Times]]
        - [[#tRC (Row Cycle Time)]]
    - [[#Recovery Delay]]
        - [[#tWR (Write Recovery Time)]]
        - [[#tRTP (Read to Precharge)]]
    - [[#Wr-to-Rd Turnaround]]
        - [[#tWTRS (Write to Read Delay, Short)]]
        - [[#tWTRL (Write to Read Delay, Long)]]
    - [[#Refresh Cycle Time]]
        - [[#tRFC (Refresh Cycle Time)]]
        - [[#tRFC2 (Refresh Cycle Time 2)]]
        - [[#tRFC4 (Refresh Cycle Time 4)]]
- [[#Tertiary Timings]]
    - [[#Same Bank Group Delay (SCLs)]]
        - [[#tRDRDSCL (Read to Read, Same Bank Group, Long)]]
        - [[#tWRWRSCL (Write to Write, Same Bank Group, Long)]]
    - [[#Raw Turnaround]]
        - [[#tRdWr (Read to Write Delay)]]
        - [[#tWrRd (Write to Read Delay)]]
    - [[#Same Chip Delay (SC)]]
        - [[#tRdRdSC (Read to Read, Same Chip)]]
        - [[#tWrWrSC (Write to Write, Same Chip)]]
    - [[#Same DIMM Delay (SD)]]
        - [[#tRdRdSD (Read to Read, Same DIMM, Different Rank)]]
        - [[#tWrWrSD (Write to Write, Same DIMM, Different Rank)]]
    - [[#Diff DIMM Delay (DD)]]
        - [[#tRdRdDD (Read to Read, Different DIMM)]]
        - [[#tWrWrDD (Write to Write, Different DIMM)]]
- [[#Signal Integrity & Drive Strength]]
    - [[#Misc Signal]]
        - [[#tCKE (Clock Enable)]]
        - [[#tTRCPAGE (Target Row Cycle Page)]]
    - [[#Termination Impedance]]
    - [[#Setup Times]]
    - [[#Drive Strength]]
    - [[#Misc]]
- [[#Validation Protocol]]

---

## System Architecture & Environment

*   **Operating System:** Arch Linux 
*   **Validation Toolchain:** `stressapptest` (memory architecture), `mprime` (Large FFTs for IMC/Fabric), `dmesg` (hardware error monitoring)
*   **Processor:** AMD Ryzen 7 5800X3D (Infinity Fabric Target: 1866 MHz)
*   **Memory Platform:** 32GB (2x16GB) Crucial Ballistix (Part: BL16G32C16U4B)
*   **Memory Silicon:** Dual-Rank Micron Rev. E-die (Factory Bin: 3200 MT/s)
*   **Target Overclock:** 3733 MT/s 
*   **Motherboard Firmware:** ASUS UEFI BIOS Utility (Advanced Mode)

---

## Voltages

### DRAM Voltage
*   **Recommended Setting:** **1.35V to 1.38V.**
*   **Reasoning:** Micron E-die actually tolerates high voltage well (extreme overclockers routinely push 1.45V+ to tighten tCL). The real constraint is thermal: excess voltage increases DRAM temperature, which directly destabilizes temperature-sensitive timings like tRFC. On dual-rank modules, heat accumulates faster, so 1.35–1.38V is a practical ceiling for sustained daily use — not because E-die is voltage-fragile, but because the resulting heat threatens tRFC stability. See [[DDR4 OC Guide#Voltage Scaling|DDR4 OC Guide: Voltage Scaling]] and [[DDR4 OC Guide#Maximum Recommended Daily Voltage|Maximum Recommended Daily Voltage]].

### SoC Voltage (VDDCR_SOC)
*   **Recommended Setting:** **1.10V**
*   **Reasoning:** The SoC voltage powers the memory controller and Infinity Fabric. 1.10V is sufficient for 3733 MT/s on most 5800X3D samples. Increase to 1.15V only if the IMC fails training at tight timings — exceeding 1.20V yields no benefit and risks degradation. See [[DDR4 OC Guide#AMD IMC|DDR4 OC Guide: AMD IMC]] and [[Infinity Fabric Overclocking on Zen2_3#Relevant CPU Voltages|Infinity Fabric: Relevant CPU Voltages]].

### CLDO VDDP / VDDG CCD / VDDG IOD (Infinity Fabric Sub-Voltages)
*   **Recommended Settings:** **Auto** (reference values below for manual tuning)
    *   CLDO VDDP: ~0.900V (stabilizes memory module signal strength)
    *   VDDG CCD: ~0.950V (core-to-I/O die data transfer)
    *   VDDG IOD: ~0.950V–1.050V (memory controller-to-I/O die transfer)
*   **Reasoning:** These sub-voltages drive the Infinity Fabric and memory controller signaling. On most ASUS B550 boards, Auto derives them from SoC voltage (VDDG ≈ SoC/2 + offset) and works fine for 1866 MHz FCLK. Manual tuning is only needed if pushing FCLK beyond 1900 MHz or diagnosing elusive WHEA errors. Note: VDDG must not exceed SoC voltage minus ~50mV, or the IMC becomes unstable. See [[DDR4 OC Guide#AMD IMC|DDR4 OC Guide: AMD IMC]] (CLDO_VDDG / VDDP) and [[Infinity Fabric Overclocking on Zen2_3#Relevant CPU Voltages|Infinity Fabric: Relevant CPU Voltages]].

---

## Core Signal Control

### Infinity Fabric & Memory Controller (Pre-Foundation)

#### FCLK (Infinity Fabric Frequency)
*   **Recommended Setting:** **1866 MHz**
*   **Reasoning:** At 3733 MT/s, MEMCLK = 1866 MHz. The 1:1 mode requires FCLK = MEMCLK = 1866 MHz. This is the sweet spot for Verdeille (5800X3D) — pushing beyond 3800 MT/s forces a 2:1 FCLK divider that increases latency by ~10ns, negating all timing gains. See [[Infinity Fabric Overclocking on Zen2_3#Setting Realistic Expectations|Infinity Fabric: Setting Realistic Expectations]] and [[Infinity Fabric Overclocking on Zen2_3#Step-by-Step FCLK Overclocking Guide|Step-by-Step FCLK Overclocking Guide]].

#### UCLK (Memory Controller Frequency)
*   **Recommended Setting:** **UCLK = MEMCLK (1:1)**
*   **Reasoning:** The UCLKDIVEN setting must be set so UCLK runs synchronous to MEMCLK. Any async mode introduces a latency penalty on every memory transaction.

#### ProcODT (Processor On-Die Termination)
*   **Recommended Setting:** **48Ω** (Range: 43.6–60Ω)
*   **Reasoning:** ProcODT controls signal reflection at the CPU socket. Dual-rank Micron E-die at 3733 MT/s typically requires 48Ω for stable training. Too low (40Ω) causes POST failures; too high (60Ω+) can cause signal ringing and subtle instability. This is often the single most impactful non-timing parameter for 5800X3D dual-rank stability. See [[DDR4 OC Guide#AMD IMC|DDR4 OC Guide: AMD IMC]] (/r/overclocking ProcODT notes) and [[Infinity Fabric Overclocking on Zen2_3#Relevant BIOS settings|Infinity Fabric: Relevant BIOS settings]].

### Structural & Non-Timing Prerequisites

#### Gear Down Mode (GDM)
*   **Recommended Setting:** **Enabled.**
*   **Reasoning:** Acts as a hybrid "1.5T" command rate. The structural keystone that prevents dual-rank memory from instantly crashing at 3733 MT/s. See [[DDR4 OC Guide#Finding a Baseline|DDR4 OC Guide: Finding a Baseline]] (Gear Down Mode / Command Rate).

#### Power Down Enable (PDE)
*   **Recommended Setting:** **Disabled.**
*   **Reasoning:** Prevents memory from entering micro-sleep states. Instantly shaves 2-3ns off system latency by eliminating the electrical "wake-up" penalty. See [[DDR4 OC Guide#Finding a Baseline|DDR4 OC Guide: Finding a Baseline]] (DRAM PowerDown Mode).

#### Command Rate (Cmd2T)
*   **Recommended Setting:** **1T.**
*   **Reasoning:** Managed securely by GDM. Keeping it at 1T ensures the base signal structure is as fast as possible.

---

## Primary Timings

The core pillars of memory operation. They establish the baseline latency floor.

### tCL (CAS Latency)
*   **Explanation:** The raw delay between the processor requesting data and the RAM initiating the read.
*   **Values:** Loose: 18 | Stable: 16 | Tight: 16
*   **Latency/Throughput Value:** Foundational latency. Dictates the absolute floor for system responsiveness.
*   **Interaction:** Sets the mathematical baseline for `tRAS` and `tCWL`. When Gear Down Mode is enabled, all primary timings must be integers (not half-cycle values), but they do not need to be exclusively even.
*   **Voltage/Temp Interaction:** Negligible thermal impact. Scales poorly with voltage on Micron E-die. See [[DDR4 OC Guide#Voltage Scaling|DDR4 OC Guide: Voltage Scaling]] (tCL voltage scaling) and [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 5 (tCL).

### tRCDRd (RAS to CAS Delay - Read)
*   **Explanation:** Time required to activate a row and locate the specific data column for a read.
*   **Values:** Loose: 20 | Stable: 19 | Tight: 19
*   **Latency/Throughput Value:** The single most critical bottleneck for raw read latency.
*   **Interaction:** Mathematically drives the minimum `tRAS` value.
*   **Voltage/Temp Interaction:** Highly resistant to voltage scaling. E-die physically cannot execute reads at lower delays regardless of voltage, but tightening generates no extra heat. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 6 (tRCD).

### tRCDWr (RAS to CAS Delay - Write)
*   **Explanation:** Time required to activate a row and locate the specific data column for a write.
*   **Values:** Loose: 16 | Stable: 8 | Tight: 8
*   **Latency/Throughput Value:** Secondary latency bottleneck. E-die can execute this almost instantly compared to reads.
*   **Interaction:** Minimal interaction with other timings. On Micron E-die, tRCDWr=8 at 3733 MT/s is a known sweet spot, but dropping below 8 causes immediate instability due to the internal write recovery path requiring additional cycles that E-die physically cannot short-circuit.
*   **Voltage/Temp Interaction:** Highly resistant to voltage scaling. Generates no extra heat when tightened.

### tRP (Row Precharge)
*   **Explanation:** Delay mandated to close an active memory row and chemically recharge the bank.
*   **Values:** Loose: 16 | Stable: 14 | Tight: 12
*   **Latency/Throughput Value:** Massive latency reduction in highly randomized workloads where rows are constantly opened and closed.
*   **Interaction:** Directly dictates the cycle time floor via the formula $tRC = tRP + tRAS$.
*   **Voltage/Temp Interaction:** Minor thermal impact. A value of 12 relies entirely on the underlying silicon quality.

### tRAS (Active to Precharge Delay)
*   **Explanation:** Minimum duration a row must stay open to fully complete its operation before closing.
*   **Values:** Loose: 38 | Stable: 36 | Tight: 36
*   **Latency/Throughput Value:** Bandwidth consistency. Squeezing too tight terminates operations prematurely, destroying throughput.
*   **Interaction:** JEDEC defines the architectural minimum as $tRAS_{min} \approx tRCDRd + tRP$. For 3733 MT/s with tRCDRd=19 and tRP=14, this yields $tRAS_{min} \approx 33$, so 36 provides a safe margin above the floor. Note: the [[DDR4 OC Guide#Tightening Timings|DDR4 OC Guide: Tightening Timings]] step 7 uses an alternative extreme-tuning heuristic ($tRAS = tRCD_{RD} + tRTP$) for aggressive overwrite paths — this is a tightening guideline, not the JEDEC architectural floor.
*   **Voltage/Temp Interaction:** Negligible thermal generation. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 8 (tRC = tRP + tRAS).

---

## Secondary Timings

### Write Latency

#### tCWL (CAS Write Latency)
*   **Explanation:** The exact equivalent of `tCL`, but for writing data instead of reading.
*   **Values:** Loose: Auto | Stable: 16 | Tight: 14
*   **Latency/Throughput Value:** Dictates the baseline responsiveness for write operations.
*   **Interaction:** For optimal stability on Ryzen, `tCWL` should be ≤ `tCL`. The guideline `tCWL = tCL − 2` is achievable on Micron E-die at 3733 MT/s with sufficient DRAM voltage (1.38V).
*   **Voltage/Temp Interaction:** Negligible thermal impact. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 3 (tCWL = tCL - 2 on AMD).

### Holy Trinity

Governs raw data volume and is the primary source of memory heat.

#### tRRDS (Row Active to Row Active Delay, Short)
*   **Explanation:** Mandatory speed limit for activating rows in *different* bank groups.
*   **Values:** Loose: Auto | Stable: 4 | Tight: 4
*   **Latency/Throughput Value:** Foundational throughput multiplier.
*   **Interaction:** The formula $tFAW = 4 \times tRRDS$ strictly dictates the relationship.
*   **Voltage/Temp Interaction:** High thermal generator when squeezed.

#### tRRDL (Row Active to Row Active Delay, Long)
*   **Explanation:** Mandatory speed limit for activating rows in the *same* bank group.
*   **Values:** Loose: Auto | Stable: 6 | Tight: 4
*   **Latency/Throughput Value:** Exponential throughput multiplier. Dictates internal bandwidth bottlenecks.
*   **Interaction:** Works in tandem with `tRRDS` and `tFAW` to control total activation volume.
*   **Voltage/Temp Interaction:** Extreme thermal generator. Squeezing this to 4 is the primary cause of heat-soaked bit-flips on dual-rank modules. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 1 (tRRDS tRRDL tFAW).

#### tFAW (Four Activate Window)
*   **Explanation:** The total rolling time window allowed for exactly four row activations.
*   **Values:** Loose: Auto | Stable: 16 | Tight: 16
*   **Latency/Throughput Value:** Maximizes CPU efficiency by flooding the Infinity Fabric with continuous data.
*   **Interaction:** JEDEC mandates $tFAW_{min} = 4 \times tRRDS$. Setting tFAW higher is allowed but yields no performance gain; setting it lower causes immediate instability.
*   **Voltage/Temp Interaction:** Maximum thermal generator. Forces the controller to relentlessly hammer the RAM.

### Bank Cycle Times

#### tRC (Row Cycle Time)
*   **Explanation:** Total time required for a memory bank to complete a full open-read-close sequence.
*   **Values:** Loose: Auto | Stable: 60 | Tight: 56 (58 stable on this IMC; 56 unstable)
*   **Latency/Throughput Value:** Dictates the absolute maximum sustained bandwidth limit of the module.
*   **Interaction:** Base mathematics dictate $tRC = tRP + tRAS$. Dual-rank configurations strictly require extra clock cycles of padding above this sum: typically $tRC \geq tRP + tRAS + 1\text{-}2$ cycles to account for rank-to-rank switching overhead.
*   **Voltage/Temp Interaction:** High thermal generator. Increasing this value acts as an immediate thermal relief valve if the Holy Trinity generates too much heat.

### Recovery Delay

#### tWR (Write Recovery Time)
*   **Explanation:** Cooldown period after finishing a write operation before precharging the bank.
*   **Values:** Loose: Auto | Stable: 16 | Tight: 12
*   **Latency/Throughput Value:** Secondary latency reduction during mixed workloads.
*   **Interaction:** JEDEC does not hard-link `tWR` and `tRTP`. However, for Ryzen IMC stability at 3733 MT/s dual-rank, maintaining $tWR \geq 2 \times tRTP$ is a recommended guideline — not an architectural constraint.
*   **Voltage/Temp Interaction:** Minor thermal impact. A value of 16 is highly recommended for dual-rank stability at 3733 MT/s. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 1 (tWR tRTP; `tWR = 2 × tRTP` per Micron/JEDEC datasheet).

#### tRTP (Read to Precharge)
*   **Explanation:** Cooldown period after finishing a read operation before precharging the bank.
*   **Values:** Loose: Auto | Stable: 8 | Tight: 6
*   **Latency/Throughput Value:** Modest reduction in read/precharge cycling delays.
*   **Interaction:** Closely related to `tWR`. The guideline $tRTP \approx tWR / 2$ is a useful starting point, but it is not an architectural hard-link — the IMC can accept other ratios if signal integrity allows.
*   **Voltage/Temp Interaction:** Minimal thermal generation.

### Wr-to-Rd Turnaround

#### tWTRS (Write to Read Delay, Short)
*   **Explanation:** Enforced pause when the pipeline violently shifts from writing to reading across different bank groups.
*   **Values:** Loose: Auto | Stable: 4 | Tight: 4
*   **Latency/Throughput Value:** Smoothes out heavy, unpredictable I/O workloads.
*   **Interaction:** Overrides and functionally replaces standard turnaround timings (`tWrRd`).
*   **Voltage/Temp Interaction:** Minimal thermal generation. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 3 (tWTRS tWTRL, tCWL).

#### tWTRL (Write to Read Delay, Long)
*   **Explanation:** Enforced pause when shifting from writing to reading within the same bank group.
*   **Values:** Loose: Auto | Stable: 12 | Tight: 8
*   **Latency/Throughput Value:** Crucial for preventing data collisions during same-bank I/O shifts.
*   **Interaction:** Functions alongside `tWTRS` to override `tWrRd`.
*   **Voltage/Temp Interaction:** Minor impact.

### Refresh Cycle Time

Tuned independently after cycle times are verified thermally stable. **Tune last — temp sensitive.**

#### tRFC (Refresh Cycle Time)
*   **Explanation:** The primary system blackout duration where all operations halt to electrically recharge storage cells.
*   **Values:** Loose: 653 (~350ns) | Stable: 630 (~337ns) | Tight: 560 (~300ns)
*   **Latency/Throughput Value:** Substantial reduction in overall system latency and inter-process stuttering.
*   **Interaction:** Formula: $tRFC = \frac{\text{Target Time (ns)} \times \text{Memory Speed (MT/s)}}{2000}$. For 300ns at 3733 MT/s: $300 \times 3733 / 2000 \approx 560$.
*   **Voltage/Temp Interaction:** **Hyper-sensitive to temperature.** Because heat accelerates electrical leakage, hot silicon requires longer refreshes. A tight value of 560 will immediately trigger a bit-flip if temperatures rise. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 2 (tRFC, ns conversion formula) and [[DDR4 OC Guide#Temperatures and Its Effect on Stability|Temperatures and Its Effect on Stability]].

#### tRFC2 (Refresh Cycle Time 2)
*   **Explanation:** Secondary refresh interval profile, originally designed for high-temperature states.
*   **Values:** **Auto** (BIOS trained; guideline: ≤ 0.7 × tRFC)
*   **Latency/Throughput Value:** None.
*   **Interaction:** BIOS trained reference value. On AMD Zen 3 platforms, the memory controller effectively ignores user-set tRFC2/tRFC4 values — these are auto-trained by BIOS during POST and do not impact core stability or performance tuning. Focus debugging effort on tRFC instead.
*   **Voltage/Temp Interaction:** N/A.

#### tRFC4 (Refresh Cycle Time 4)
*   **Explanation:** Tertiary refresh interval profile.
*   **Values:** **Auto** (BIOS trained; guideline: ≤ 0.5 × tRFC2)
*   **Latency/Throughput Value:** None.
*   **Interaction:** BIOS trained reference value.
*   **Voltage/Temp Interaction:** N/A.

---

## Tertiary Timings

Tertiary timings that optimize data pipeline efficiency. Tune order: SCLs → Raw Turnarounds → SC → SD → DD. This order follows dependency layers from inner (bank group) to outer (cross-DIMM).

### Same Bank Group Delay (SCLs)

Optimizes data pipeline efficiency within the same bank groups. **Tune first** — these are granular and depend on IC quality.

#### tRDRDSCL (Read to Read, Same Bank Group, Long)
*   **Explanation:** Delay enforced between consecutive read commands executing within the same bank group.
*   **Values:** Loose: Auto | Stable: 5 | Tight: 4
*   **Latency/Throughput Value:** Measurable increase in raw read MB/s in synthetic benchmarks.
*   **Interaction:** Best practice dictates keeping this perfectly synchronized with `tWRWRSCL`.
*   **Voltage/Temp Interaction:** Minimal heat generation; stability relies almost entirely on the CPU's IMC quality. See [[DDR4 OC Guide#Tightening Timings|Tightening Timings]] step 4 (tRDRDSCL tWRWRSCL).

#### tWRWRSCL (Write to Write, Same Bank Group, Long)
*   **Explanation:** Delay enforced between consecutive write commands executing within the same bank group.
*   **Values:** Loose: Auto | Stable: 5 | Tight: 4
*   **Latency/Throughput Value:** Measurable increase in raw write MB/s.
*   **Interaction:** Keep synchronized with `tRDRDSCL`.
*   **Voltage/Temp Interaction:** Minimal heat generation.

### Raw Turnaround

Controls how rapidly the memory bus can swap between read and write states. Related to `tCL` and `tCWL`.

#### tRdWr (Read to Write Delay)
*   **Explanation:** Raw turnaround timing required to swap the memory bus from a read state to a write state.
*   **Values:** Loose: Auto (usually 18) | Stable: 18 | Tight: 16
*   **Latency/Throughput Value:** Reduces latency penalty when swapping operation directions.
*   **Interaction:** Operates semi-independently of the WTR timings.
*   **Voltage/Temp Interaction:** Negligible.

#### tWrRd (Write to Read Delay)
*   **Explanation:** Raw turnaround timing to swap the memory bus from a write state to a read state.
*   **Values:** Loose: Auto | Stable: Auto (often trains to 7) | Tight: Auto
*   **Latency/Throughput Value:** Highly superseded by the tighter WTR constraints.
*   **Interaction:** Heavily overridden by `tWTRS` and `tWTRL` on the Ryzen architecture.
*   **Voltage/Temp Interaction:** Negligible.

### Same Chip Delay (SC)

Delay when operations stay on the exact same physical chip. **Tune after SCLs and Raw Turnarounds.**

#### tRdRdSC (Read to Read, Same Chip)
*   **Explanation:** Delay when read operations stay on the exact same physical memory module chip.
*   **Values:** **1**
*   **Latency/Throughput Value:** Practically instant throughput execution.
*   **Interaction:** Baseline internal chip timing.
*   **Voltage/Temp Interaction:** Zero impact.

#### tWrWrSC (Write to Write, Same Chip)
*   **Explanation:** Delay when write operations stay on the exact same physical chip.
*   **Values:** **1**
*   **Latency/Throughput Value:** Practically instant throughput execution.
*   **Interaction:** Matches the read variant.
*   **Voltage/Temp Interaction:** Zero impact.

### Same DIMM Delay (SD)

Delay when switching between ranks on the same DIMM. **Tune after SC.** Sensitive to voltage and signal integrity.

#### tRdRdSD (Read to Read, Same DIMM, Different Rank)
*   **Explanation:** Delay when the controller stops reading from the chips on the front of the stick and switches to the chips on the back of the exact same stick.
*   **Values:** Loose: Auto (6-7) | Stable: 5 | Tight: 4
*   **Latency/Throughput Value:** Major impact on dual-rank interleaving bandwidth.
*   **Interaction:** If set too tight (e.g., 1), signals physically crash into each other on the PCB causing instant lockups.
*   **Voltage/Temp Interaction:** Requires strong signal integrity.

#### tWrWrSD (Write to Write, Same DIMM, Different Rank)
*   **Explanation:** Delay when switching write operations from the front to the back of the exact same stick.
*   **Values:** Loose: Auto | Stable: 5 | Tight: 4 (5 unstable on this IMC at 1.35V; stable at 1.38V)
*   **Latency/Throughput Value:** Optimizes write interleaving across dual-rank memory.
*   **Interaction:** Matches the read variant closely.
*   **Voltage/Temp Interaction:** Requires strong signal integrity.

### Diff DIMM Delay (DD)

Delay when switching between DIMMs. **Tune last** — outermost layer, most sensitive to PCB trace quality and IMC pressure.

#### tRdRdDD (Read to Read, Different DIMM)
*   **Explanation:** Delay when switching read operations from the stick in RAM Slot A2 to the stick in RAM Slot B2.
*   **Values:** Loose: Auto | Stable: 5 | Tight: 4
*   **Latency/Throughput Value:** Optimizes cross-channel interleaving.
*   **Interaction:** In a 2-stick setup, this timing generally just needs to parallel your "SD" timing.
*   **Voltage/Temp Interaction:** Negligible thermal impact.

#### tWrWrDD (Write to Write, Different DIMM)
*   **Explanation:** Delay when switching write operations from Slot A2 to Slot B2.
*   **Values:** Loose: Auto | Stable: 5 | Tight: 4
*   **Latency/Throughput Value:** Optimizes cross-channel write throughput.
*   **Interaction:** Parallels the read variant.
*   **Voltage/Temp Interaction:** Negligible.

---

## Signal Integrity & Drive Strength

These timings manage signal termination, drive strength, and miscellaneous signal control. Best left to Auto unless chasing 4000+ MT/s.

### Misc Signal

#### tCKE (Clock Enable)
*   **Explanation:** Minimum time required between power-down and power-up commands.
*   **Values:** Loose: Auto | Stable: 1
*   **Latency/Throughput Value:** None.
*   **Interaction:** Because Power Down Enable is strictly Disabled, this timing is functionally bypassed entirely by the controller.
*   **Voltage/Temp Interaction:** Zero impact.

#### tTRCPAGE (Target Row Cycle Page)
*   **Explanation:** An obscure timing related to macOS-specific memory page management limits.
*   **Values:** **0** or **Auto**
*   **Latency/Throughput Value:** None.
*   **Interaction:** Rarely utilized by the Ryzen IMC and has absolutely zero impact on Arch Linux or Windows environments.
*   **Voltage/Temp Interaction:** Zero impact.

### Termination Impedance

#### RttNom / RttWr / RttPark
*   **Recommended Setting:** **Auto.** (Or specifically `Off / Off / RZQ/5` if Auto fails).
*   **Reasoning:** Controls signal reflection on the motherboard traces. Auto is highly accurate on ASUS B550/X570 boards.

### Setup Times

#### MemAddrCmdSetup / MemCsOdtSetup / MemCkeSetup
*   **Recommended Setting:** **Auto.**
*   **Reasoning:** Memory address command, chip select ODT, and clock enable setup times. Best left to the motherboard's training algorithm.

### Drive Strength

#### MemCadBus (Clk, AddrCmd, CsOdt, Cke)
*   **Recommended Setting:** **Auto.** (Usually trains to `24-24-24-24` Ohms).
*   **Reasoning:** Drive strengths. Crucial for E-die stability but best left to the motherboard's training algorithm unless chasing 4000+ MT/s.

### Misc

#### Mem Over Clock Fail Count
*   **Recommended Setting:** **Auto.**
*   **Reasoning:** Memory overclock fail counter. Leave at Auto.

---

## Validation Protocol

**Staged Tuning Methodology:** Do not apply all 30+ timings at once. If the system fails, you cannot isolate which parameter caused it. Follow this staged approach:

1.  **Stage 1 — FCLK & Primary:** Set frequency (3733 MT/s), FCLK (1866), UCLK (1:1), SoC voltage, ProcODT, and the primary 5-tuple only. Boot into OS and check `dmesg`/`journalctl` for WHEA errors. Run mprime Small FFTs for 30 minutes to confirm FCLK stability.
2.  **Stage 2 — Secondaries & Tertiaries:** Once Stage 1 passes, apply secondary timings (tCWL, tRRDS/L, tFAW, tRC, tWR, tRTP, tWTRS/L, tRFC/tRFC2/tRFC4) and tertiary timings (SCLs, Raw Turnarounds, SC/SD/DD). Run mprime Large FFTs for 1 hour.
3.  **Stage 3 — Aggressive Tightening:** Apply tight tRFC, tRRDL=4, and tight SCLs. Run the full validation protocol below.

After applying timings at each stage, stability must be verified in a specific order. Each test catches different failure modes — skipping any step risks intermittent crashes that are difficult to diagnose later.

**Step 1: Boot & Initial Training**
*   **Action:** Boot into BIOS, apply all settings, cold boot.
*   **Pass Criteria:** Successful POST and OS boot without memory training fallback (check RAM speed in `dmidecode -t memory`).
*   **Note:** If the system fails to POST, clear CMOS and loosen primary timings by one step (e.g., tCL 16→18, tRCDRd 19→20).

**Step 2: mprime (Large FFTs) — 4 Hours**
*   **Action:** `mprime -t`, select option 2 (Small FFTs, CPU L1/L2/L3) then option 3 (Large FFTs, memory controller & RAM).
*   **Pass Criteria:** Zero rounding errors, zero FATAL ERROR lines in `results.txt`.
*   **Catches:** IMC instability, voltage starvation, tRFC too aggressive, ProcODT mismatch.

**Step 3: stressapptest — 2 Hours**
*   **Action:** `stressapptest -s 7200 -M 30000 -W`
*   **Pass Criteria:** No miscompare errors, exit code 0.
*   **Catches:** Memory cell-level corruption, data line signal integrity, rank-to-rank switching failures (SD/DD timings).

**Step 4: TestMem5 (Anta777 Extreme1 or Absolut config) — 3 Cycles**
*   **Action:** Run TM5 with the **Anta777 Extreme1** or **Anta777 Absolut** preset for 3 full cycles.
*   **Environment Note:** TM5 is a Windows-only application. On an Arch Linux system, run it from a Windows PE USB drive or a dual-boot Windows partition. Running TM5 via WINE is not recommended — the translation layer prevents accurate physical memory mapping, defeating TM5's low-level diagnostic purpose. If Windows is unavailable, substitute with `y-cruncher` (stresses IMC and FCLK heavily) or `memtester` (native Linux, though less aggressive on timing interactions).
*   **Pass Criteria:** Zero errors across all 3 cycles.
*   **Catches:** Subtle timing interaction bugs that mprime and stressapptest miss. Anta777 configs are specifically tuned for heat-soaking dual-rank memory under sustained load, and are more effective at catching high-temperature bit-flips than the older 1usmus_v3 preset.

**Step 5: dmesg Monitoring (Continuous)**
*   **Action:** After each test, check `journalctl -k --grep=mce` for machine check exceptions.
*   **Pass Criteria:** No MCE events, no corrected/uncorrected error entries.
*   **Catches:** Silent hardware errors that don't cause immediate crashes but indicate marginal stability. See [[Infinity Fabric Overclocking on Zen2_3#Some thoughts on the notorious "WHEA-19s"|Infinity Fabric: WHEA-19s]].

**Failure Recovery:**
*   If Step 2 fails → loosen tRFC (+30), tRRDL (6→8), or increase DRAM V to 1.38V.
*   If Step 3 fails → loosen SD/DD timings by 1 (5→6), or increase ProcODT to 53.3Ω.
*   If Step 4 fails → loosen tRTP (6→8), tWTRL (8→12), or tWR (12→16).
*   If random WHEA/crash under mixed load → increase SoC V to 1.12-1.15V.
