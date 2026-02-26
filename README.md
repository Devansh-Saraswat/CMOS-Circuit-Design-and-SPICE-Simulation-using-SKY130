# CMOS Circuit Design & SPICE Simulation — SKY130 Technology

> A complete 5-day course evaluation covering MOSFET fundamentals, SPICE simulation, and CMOS inverter robustness analysis using the open-source SKY130 PDK.

---

## 📋 Course Overview

This workshop dives into CMOS circuit design and SPICE simulation built on the SKY130 130nm process node, hosted by [VLSI System Design](https://www.vlsisystemdesign.com/). Over five days, the curriculum builds progressively from transistor physics to full inverter robustness analysis.

| Day | Topic |
|-----|-------|
| Day 1 | NMOS fundamentals, drain current equations, intro to SPICE |
| Day 2 | Velocity saturation in short-channel devices, CMOS VTC basics |
| Day 3 | Switching threshold (Vm), dynamic transient simulation |
| Day 4 | Noise margin analysis and robustness |
| Day 5 | Power supply scaling and process variation effects |

---

## ⚙️ Environment Setup — VirtualBox

### Installing VirtualBox
Head to https://www.virtualbox.org/wiki/Downloads and install the appropriate version for your OS.

### Creating the Virtual Machine
Launch VirtualBox and click **New**. Configure as follows:

| Parameter | Value |
|-----------|-------|
| OS Type | Linux |
| Version | Ubuntu 18.04 Bionic Beaver (64-bit) |
| RAM | 4096 MB (recommended) |

### Attaching the VDI Disk Image
When prompted for a hard disk, choose **"Use an existing virtual hard disk file"**, browse to the unzipped CMOS VDI file, and confirm.

<img width="1102" height="927" alt="image" src="https://github.com/user-attachments/assets/a183867c-dc8e-4c8b-8f1d-ccb65391a0af" />


Once the VM is created, select it and hit **Start** to boot into Ubuntu.

---

## 📑 Index

1. [Day 1 — NMOS Fundamentals & Drain Current](#day-1--nmos-fundamentals--drain-current)
   - [Why SPICE?](#why-spice)
   - [NMOS Device Structure](#nmos-device-structure)
   - [Strong Inversion & Threshold Voltage](#strong-inversion--threshold-voltage)
   - [Body Effect](#body-effect)
   - [Linear Region of Operation](#linear-region-of-operation)
   - [Drift Current & the Id Equation](#drift-current--the-id-equation)
   - [Saturation & Pinch-Off](#saturation--pinch-off)
   - [SPICE Primer](#spice-primer)
   - [Lab — Day 1](#lab--day-1)
2. [Day 2 — Velocity Saturation & CMOS VTC](#day-2--velocity-saturation--cmos-vtc)
   - [Short-Channel Effects](#short-channel-effects)
   - [Lab — Day 2](#lab--day-2)
   - [CMOS Inverter & VTC](#cmos-inverter--vtc)
3. [Day 3 — Switching Threshold & Transient Analysis](#day-3--switching-threshold--transient-analysis)
   - [SPICE Deck for the Inverter](#spice-deck-for-the-inverter)
   - [Lab — Day 3](#lab--day-3)
   - [Switching Threshold (Vm)](#switching-threshold-vm)
4. [Day 4 — Noise Margin Robustness](#day-4--noise-margin-robustness)
   - [Noise Margin Theory](#noise-margin-theory)
   - [Lab — Day 4](#lab--day-4)
5. [Day 5 — Supply & Device Variation](#day-5--supply--device-variation)
   - [Power Supply Scaling](#power-supply-scaling)
   - [Lab — Day 5 (Supply)](#lab--day-5-supply)
   - [Manufacturing Variation](#manufacturing-variation)
   - [Lab — Day 5 (Device)](#lab--day-5-device)
6. [Conclusion](#conclusion)
7. [References](#references)

---

## Day 1 — NMOS Fundamentals & Drain Current

### Why SPICE?

Modern digital chips are built from logic gates — AND, OR, NAND, NOR, XOR — which are themselves assembled from CMOS transistor pairs. The simplest example is the **CMOS inverter**: a PMOS transistor in the Pull-Up Network (PUN) stacked with an NMOS in the Pull-Down Network (PDN).

<img width="469" height="508" alt="image" src="https://github.com/user-attachments/assets/1b2c5722-1a9d-4b7d-b24d-47282e2cf256" />


Its logic behavior is straightforward:
- **Input LOW (0 V)** → PMOS conducts, NMOS off → output pulls to VDD
- **Input HIGH (VDD)** → NMOS conducts, PMOS off → output pulls to GND

While the logic is simple, **timing is not**. When inverters drive capacitive loads, the charge and discharge time determines propagation delay — and that delay is non-trivial to calculate by hand. This is exactly where SPICE comes in.

SPICE solves the circuit's differential equations numerically across a sweep of input values. By scanning both PMOS and NMOS I-V responses, it reconstructs the full inverter waveform and accurately captures delay:

<img width="846" height="367" alt="image" src="https://github.com/user-attachments/assets/2ea247d5-3cfc-43c8-8172-c44342b2694d" />


SPICE also generates **delay vs. W/L curves**, which show how adjusting transistor sizing trades off speed and power:

<img width="764" height="345" alt="image" src="https://github.com/user-attachments/assets/30a5406f-ff60-49c6-89ac-3c35c9f7b362" />


**Connecting SPICE to the physical design flow:** When a standard cell (say, a buffer) drives different load capacitances at various input transition rates, SPICE generates a table of delays. These tables become the Liberty (.lib) timing models consumed by STA tools like OpenSTA.

<img width="1217" height="708" alt="image" src="https://github.com/user-attachments/assets/28cd2ee7-4b55-4fc4-82ea-a4ce6a261fd3" />


Three practical questions SPICE answers that hand calculations cannot:
- What is the **true source** of a cell's delay number in a timing library? → SPICE transistor-level simulation.
- Are simplified delay models **accurate enough**? → Models approximate; SPICE remains the ground truth.
- How do we **validate** STA results on a chip? → Cross-check critical paths against SPICE simulation.

---

### NMOS Device Structure

The NMOS is a **four-terminal** device. Its cross-sectional structure (from bottom to top) is:

- P-type silicon substrate (body, terminal **B**)
- n+ doped source and drain diffusion regions (terminals **S** and **D**)
- SiO₂ isolation separating active regions
- Thin gate oxide grown on the substrate between source and drain
- Polysilicon (or metal) gate electrode on top of the oxide (terminal **G**)

<img width="509" height="464" alt="image" src="https://github.com/user-attachments/assets/bfb6bfa6-5f7d-44f6-ae5b-b44b0eb53a7c" />


The ratio of channel **W**idth to **L**ength (W/L) is the single most important sizing parameter — it sets drive current, switching speed, and input capacitance.

---

### Strong Inversion & Threshold Voltage

**At rest (Vgs = 0 V):** With drain, source, and body all at ground, the B-S and B-D junctions are unbiased reverse-junction diodes. No channel forms and S-D resistance is extremely high.

**Ramping up Vgs:** The positive gate voltage attracts electrons from the n+ regions toward the surface under the gate oxide, gradually creating a thin electron-rich layer.

<img width="815" height="403" alt="image" src="https://github.com/user-attachments/assets/51063569-b24e-4826-b684-751dff081e31" />

**Strong Inversion:** Once Vgs crosses a critical level, the surface beneath the gate flips from p-type to n-type — forming a complete conducting channel between source and drain. This transition is called **strong inversion**, and the gate voltage at which it occurs is the **Threshold Voltage (Vt)**.

<img width="788" height="431" alt="image" src="https://github.com/user-attachments/assets/c467f1e4-a1d8-4eac-9a7e-0b86b1714e57" />


Beyond Vt, the depletion layer stops growing. Further increases in Vgs only pull more electrons into the channel, increasing its conductivity proportionally. The Vt equation is:

```
Vt = Vt0 + γ · (√|2φf + VSB| − √|2φf|)
```

**where:**
- `Vt0` — threshold at zero body bias; set by the manufacturing process
- `γ` — body-effect coefficient (V^0.5), relates depletion charge to oxide capacitance
- `φf` — Fermi potential of the substrate

The supporting expressions:
```
γ = √(2·εsi·q·NA) / Cox        [body-effect coefficient]
φf = VT · ln(NA / ni)          [Fermi potential]
Cox = εox / tox                 [gate oxide capacitance per unit area]
```

---

### Body Effect

The body terminal is not always tied to source. Applying a **source-to-body voltage (Vsb > 0)** widens the depletion region beneath the channel, as shown below:

<img width="1207" height="605" alt="image" src="https://github.com/user-attachments/assets/21fb4e1e-401b-4d1c-a70f-ffc489491499" />


The wider depletion region pulls electrons away from the channel-forming surface, so **more gate voltage is needed** to achieve inversion. In other words, Vt increases with Vsb — this is the **body effect** (also called the back-gate effect).

<img width="1300" height="593" alt="image" src="https://github.com/user-attachments/assets/13734e79-d8cd-4127-8584-d56ab43ef500" />


<img width="1340" height="652" alt="image" src="https://github.com/user-attachments/assets/53bd1039-e243-4070-a53b-e0544494ae93" />


The body-effect coefficient γ and Fermi potential φf are not design choices — they come from foundry characterization. SKY130's SPICE model files encode these values precisely.


---

### Linear Region of Operation

The MOSFET has three primary regions: **cutoff** (OFF), **linear/resistive**, and **saturation**.

In the **linear region** (Vgs > Vt, small Vds), a conducting channel spans the full length from source to drain. As Vgs increases, the channel thickens and its conductance rises.

<img width="676" height="436" alt="image" src="https://github.com/user-attachments/assets/88a494d7-1528-4b2b-9168-e2f569a633ae" />


With Vds applied, the channel is no longer uniform — a voltage gradient V(x) builds along its length (x-axis). Using Vgs = 1 V, Vt = 0.45 V, Vds = 0.05 V as an example:
- At the source end (x=0): effective gate overdrive = Vgs − V(0) = 1.0 V
- At the drain end (x=L): effective gate overdrive = Vgs − V(L) = 0.95 V

<img width="790" height="573" alt="image" src="https://github.com/user-attachments/assets/7f9a9c92-7ec3-4a26-8c25-abbf3385d0cb" />


The local induced charge at any point x is therefore: `Qi(x) = −Cox · (Vgs − V(x) − Vt)`

<img width="432" height="547" alt="image" src="https://github.com/user-attachments/assets/b46abda5-757a-425c-bb76-961115b310a6" />


<img width="449" height="292" alt="image" src="https://github.com/user-attachments/assets/16e84919-7b3b-4dda-a931-4cec25b95e78" />


**Operating region boundary:** Vds ≤ (Vgs − Vt) → linear; once Vds crosses this threshold, the device enters saturation.

<img width="525" height="145" alt="image" src="https://github.com/user-attachments/assets/3de351c0-1dc4-48fd-b775-9d41c50c93c7" />


---

### Drift Current & the Id Equation

Two mechanisms carry current in a MOSFET channel:
- **Drift current** — carriers accelerated by the lateral electric field (the dominant mechanism here)
- **Diffusion current** — carriers moving down a concentration gradient (important sub-threshold)

Integrating the drift current over the full channel length yields the **complete drain current equation** for the linear region:

```
Id = µn·Cox·(W/L) · [(Vgs − Vt)·Vds − Vds²/2]
```

Key groupings: `kn' = µn·Cox` (process transconductance), `kn = kn'·(W/L)` (gain factor).

<img width="423" height="153" alt="image" src="https://github.com/user-attachments/assets/52519c11-7aef-4a00-a07c-5b631831b0da" />


---

### Saturation & Pinch-Off

As Vds increases toward (Vgs − Vt), the overdrive at the drain end approaches zero. At exactly Vgs − Vds = Vt, the channel "pinches off" at the drain. Beyond this point, the channel adopts a triangular shape — still connected at the source, absent at the drain.

<img width="1260" height="633" alt="image" src="https://github.com/user-attachments/assets/fb5242cb-3849-4524-a02b-8493c36ad1b7" />


The current no longer grows with Vds; it saturates at:

<img width="387" height="104" alt="image" src="https://github.com/user-attachments/assets/a53ebf83-1097-4e59-8595-dcf2b1cdc34a" />

<img width="1285" height="638" alt="image" src="https://github.com/user-attachments/assets/ceb9f02c-c101-4fe4-817a-413252c84733" />



```
<img width="346" height="70" alt="image" src="https://github.com/user-attachments/assets/e0e72f69-ea1f-472d-8a4a-2e45c73f33b5" />

```

<img width="1195" height="649" alt="image" src="https://github.com/user-attachments/assets/ba06ab6d-a44a-4ba6-9cff-c3640e8125e9" />

The `(1 + λ·Vds)` term captures **channel-length modulation** — as Vds pushes beyond pinch-off, the depletion region at the drain expands, effectively shortening the channel and allowing Id to increase very slowly. λ is a process-dependent parameter provided in SPICE model files.

---

### SPICE Primer

SPICE reads a **netlist** describing the circuit topology, device models, and simulation commands. Key workflow elements:

> **SPICE Simulation Flow**

<img width="1313" height="683" alt="image" src="https://github.com/user-attachments/assets/8d5c7f6d-8782-4a27-a224-a53f3c917a87" />

SPICE Netlist

<img width="1216" height="612" alt="image" src="https://github.com/user-attachments/assets/3b8b1c09-3300-4527-92ec-df2351a7fe08" />


SKY130 provides models for five statistical **process corners**:

| Corner Code | Meaning |
|-------------|---------|
| `tt` | Typical NMOS / Typical PMOS |
| `ff` | Fast NMOS / Fast PMOS |
| `ss` | Slow NMOS / Slow PMOS |
| `sf` | Slow NMOS / Fast PMOS |
| `fs` | Fast NMOS / Slow PMOS |

A **node** is any wire junction connecting two or more device terminals. The SPICE netlist assigns a name to every node; all unlabelled nodes default to 0 (GND).

<img width="683" height="518" alt="image" src="https://github.com/user-attachments/assets/66a2095d-e0d4-46ba-9381-3391f6994d80" />

---

### Lab — Day 1

**Objective:** Simulate Id vs. Vds for a long-channel NMOS in SKY130 and observe the linear and saturation regions.

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1  n1 in 55
Vdd vdd 0 1.8V
Vin in  0 1.8V

* ─── Simulation ──────────────────────────────────────────
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
  run
  display
  setplot dc1
.endc
.end
```

<img width="791" height="491" alt="image" src="https://github.com/user-attachments/assets/c928beed-7938-496b-97cc-3e3b15a52444" />


<img width="811" height="514" alt="image" src="https://github.com/user-attachments/assets/62889e83-ebdc-4bdf-8ef9-1af3ef014190" />


> 💡 **Reading a value:** Left-click any point on the curve. The terminal prints `x0` (Vds) and `y0` (Id in amps).

<img width="939" height="609" alt="image" src="https://github.com/user-attachments/assets/4b25c418-e33c-4af3-b728-746271d044ce" />


---

## Day 2 — Velocity Saturation & CMOS VTC

### Short-Channel Effects

**The complete Id–Vds landscape:**

Sweeping Vgs across multiple values produces a family of Id–Vds curves that reveals all operating regions simultaneously. The zero-overdrive trace (Vgs = Vt) sits flat along the x-axis — the device is fully OFF. Higher Vgs traces climb steeply in the linear region, then level off once pinch-off occurs at the saturation boundary.

![Day 2 — Id vs Vds Family of Curves](YOUR_IMAGE_URL — friend's image: Id-vs-Vds all regions/curves)

**Velocity saturation in short-channel devices:**

Long-channel MOSFET theory assumes carrier velocity rises proportionally with the lateral electric field. This relationship breaks down once the channel shortens to the 250 nm regime or below: the field intensity becomes large enough to push carriers into **velocity saturation**, where drift velocity plateaus at a material-specific maximum (vsat) regardless of further field increase.

<img width="623" height="289" alt="image" src="https://github.com/user-attachments/assets/c6c1a257-6b01-4f37-81ac-6a12e4f53990" />


<img width="681" height="284" alt="image" src="https://github.com/user-attachments/assets/d8ec917b-2d41-4eb1-be73-655d81ff954d" />


The practical consequence is that short-channel devices (L < 250 nm) exhibit **four** distinct operating regions rather than three:

| Channel Length | Operating Regions |
|----------------|-------------------|
| Long (> 250 nm) | Cutoff → Linear → Saturation |
| Short (< 250 nm) | Cutoff → Linear → **Velocity Saturation** → Saturation |

To handle both long- and short-channel behavior within a unified framework, SPICE introduces `Vmin = min(Vgt, Vds, Vdsat)`, yielding the generalized drain current expression:

```
Id = µn·Cox·(W/L) · (Vmin²/2) · (1 + λ·Vds)
```

Here `Vdsat` is a **process-defined** constant — the drain-source voltage at which velocity saturates — extracted from foundry measurements and encoded in the SKY130 model files. Shorter devices reach Vdsat at lower Vds values, making velocity saturation increasingly dominant as technology scales.

---

### Lab — Day 2

**Part A — Short-channel Id vs Vds (w=0.39 µm, l=0.15 µm):**

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1  n1 in 55
Vdd vdd 0 1.8V
Vin in  0 1.8V

* ─── Simulation ──────────────────────────────────────────
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
  run
  display
  setplot dc1
.endc
.end
```

<img width="848" height="591" alt="image" src="https://github.com/user-attachments/assets/812b2fa3-f379-405b-b9cb-4ceaafa8acea" />


<img width="793" height="552" alt="image" src="https://github.com/user-attachments/assets/ddd0ddb2-6aad-4651-8952-198d655c3a46" />


**Part B — Id vs Vgs (Threshold Voltage extraction):**

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1  n1 in 55
Vdd vdd 0 1.8V
Vin in  0 1.8V

* ─── Simulation ──────────────────────────────────────────
.op
.dc Vin 0 1.8 0.1

.control
  run
  display
  setplot dc1
.endc
.end
```

<img width="837" height="525" alt="image" src="https://github.com/user-attachments/assets/a5bc63cf-98dd-443f-9bcc-6741260e1c32" />


<img width="937" height="468" alt="image" src="https://github.com/user-attachments/assets/4236e3f7-599f-4e70-9860-faa13745a952" />



> 💡 **Extracting Vt graphically:** On the Id–Vgs plot, locate the segment where current begins to climb linearly. Extend this slope back to the x-axis — the intercept is the threshold voltage Vt for this short-channel device.

<img width="650" height="414" alt="image" src="https://github.com/user-attachments/assets/0a8c327d-1f2d-44e9-a17d-a8f5093dc94f" />


---

### CMOS Inverter & VTC

**Switch-level model of the inverter:**

Each transistor acts as a gate-controlled switch: below threshold it appears as near-infinite resistance; above threshold it presents a finite resistance whose value is set by W/L. This simplification captures inverter logic but not propagation delay — that requires full I–V analysis.

<img width="893" height="504" alt="image" src="https://github.com/user-attachments/assets/130e5c73-f2b1-46b0-8976-370d627eaa7d" />

<img width="895" height="504" alt="image" src="https://github.com/user-attachments/assets/325f226b-44ed-4fa0-8e09-6e1391a44d38" />



The terminal voltages governing each device:

<img width="348" height="501" alt="image" src="https://github.com/user-attachments/assets/4d8c33cf-1507-462e-bef4-8bfa1bb571b5" />


Plotting the PMOS and NMOS load lines individually first clarifies where each device transitions between regions. Superimposing both and tracing the intersection at every Vin value sweeps out the complete **Voltage Transfer Characteristic (VTC)**:

PMOS I-V characteristic or load curve:
<img width="831" height="246" alt="image" src="https://github.com/user-attachments/assets/2c2c6957-c239-40e2-b76f-6e762b565fab" />


NMOS I-V characteristic or load curve:
<img width="809" height="351" alt="image" src="https://github.com/user-attachments/assets/09ddf45f-8a94-4c1e-b575-1e1a80d75994" />

PMOS + NMOS load lines superimposed on same axes:
<img width="450" height="286" alt="image" src="https://github.com/user-attachments/assets/8060648d-d147-4f7f-892e-2b2213a6b44b" />


Final Vout vs Vin voltage transfer characteristic:
<img width="902" height="508" alt="image" src="https://github.com/user-attachments/assets/e3aa991d-af08-480b-89f2-9c0afbf4be22" />

---

## Day 3 — Switching Threshold & Transient Analysis

### SPICE Deck for the Inverter

A well-structured SPICE netlist for the inverter requires four ingredients:
1. **Component connectivity** — which terminal connects to which node
2. **Device parameters** — W/L sizing and model name for each transistor
3. **Source and load definitions** — supply voltages, input stimulus, output capacitance
4. **Simulation commands** — analysis type, sweep range, output variables

The standard inverter netlist template used throughout Day 3:

![Inverter SPICE Netlist Reference](YOUR_IMAGE_URL — friend's image: inverter SPICE deck/netlist screenshot)

---

### Lab — Day 3

**VTC Simulation (DC sweep):**

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0   0   sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in  0 1.8V

* ─── Simulation ──────────────────────────────────────────
.op
.dc Vin 0 1.8 0.01

.control
  run
  setplot dc1
  display
.endc
.end
```

![Day 3 VTC — NGSpice Terminal](YOUR_IMAGE_URL — friend's image: Day3 VTC DC sweep terminal output)

![Day 3 VTC — Output Waveform](YOUR_IMAGE_URL — friend's image: Day3 VTC Vout vs Vin plot)

> 💡 **Locating Vm:** Zoom into the transition region where Vout ≈ Vin. Click at the Vout = Vin crossing — because Vm lies precisely on the 45° diagonal, the readout satisfies x0 ≈ y0 ≈ Vm.

**Transient Analysis — Propagation Delay:**

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0   0   sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in  0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

* ─── Simulation ──────────────────────────────────────────
.tran 1n 10n

.control
  run
.endc
.end
```

![Day 3 Transient — NGSpice Terminal](YOUR_IMAGE_URL — friend's image: Day3 transient simulation terminal)

![Day 3 Transient — Input/Output Waveform](YOUR_IMAGE_URL — friend's image: Day3 transient waveform plot)

**Measuring rise and fall delay:**
- **Rise delay** — Zoom to the output rising edge near Vdd/2. Record the time difference between the output crossing Vdd/2 upward and the input falling through Vdd/2 downward.
- **Fall delay** — Mirror the same process on the output's falling edge against the input's rising edge.

---

### Switching Threshold (Vm)

Vm is the gate input voltage at which output equals input — the midpoint of the VTC. On the transfer curve it is the point where the characteristic crosses the unity-gain diagonal (slope = −1).

At Vm, both PMOS and NMOS are simultaneously in saturation with `Vgs = Vds` for each. KCL at the output node mandates `|IdP| = |IdN|`:

![IdN and IdP Current Balance at Vm](YOUR_IMAGE_URL — friend's image: IdN + IdP = 0 at Vm)

![Vm Derivation Equation](YOUR_IMAGE_URL — friend's image: Vm closed-form derivation)

Solving yields the closed-form expression:

![Vm Closed-Form Formula](YOUR_IMAGE_URL — friend's image: Vm formula result)

Working in reverse — if a target Vm is specified, the required PMOS-to-NMOS width ratio is:

![W/L Ratio for a Target Vm](YOUR_IMAGE_URL — friend's image: W/L ratio equation for target Vm)

**Measured results across PMOS widths:**

| PMOS sizing | Rise delay | Fall delay | Vm |
|-------------|-----------|------------|------|
| 1× (Wn/Ln) | 148 ps | 71 ps | 0.99 V |
| 2× (Wn/Ln) | 80 ps | 76 ps | 1.20 V |
| 3× (Wn/Ln) | 57 ps | 80 ps | 1.25 V |
| 4× (Wn/Ln) | 45 ps | 84 ps | 1.35 V |
| 5× (Wn/Ln) | 37 ps | 88 ps | 1.40 V |

The 2× sizing achieves near-symmetrical rise/fall delays, which is the standard choice for **clock tree buffers and inverters**. Wider PMOS ratios shift Vm further right and imbalance delays — useful in data-path contexts where asymmetric drive strength is acceptable. The intrinsic mobility mismatch (Ron_PMOS ≈ 2.5 × Ron_NMOS) explains why PMOS must be widened to achieve equivalent drive.

---

## Day 4 — Noise Margin Robustness

### Noise Margin Theory

On-chip power lines are never perfectly quiet — capacitive and inductive coupling inject noise onto logic signals. The CMOS inverter must correctly interpret its input despite this noise. **Noise margin** quantifies how much corruption the gate can absorb before misreading a logic level.

**Ideal vs. real VTC:**

An ideal inverter would flip instantaneously at exactly Vdd/2, producing a vertical-drop transfer curve with infinite gain. The physically realised characteristic transitions over a finite voltage range, and its slope in this region is the small-signal voltage gain.

![Ideal Inverter Transfer Characteristic](YOUR_IMAGE_URL — friend's image: ideal inverter VTC)

![Real Inverter Transfer Characteristic](YOUR_IMAGE_URL — friend's image: real inverter VTC with finite slope)

Four boundary voltages extracted from the real VTC define the noise margins:

| Voltage | Description | Logic Interpretation |
|---------|-------------|----------------------|
| Vol | Output Low | Maximum output still guaranteed to be logic '0' |
| Vil | Input Low | Maximum input reliably decoded as '0' |
| Vih | Input High | Minimum input reliably decoded as '1' |
| Voh | Output High | Minimum output still guaranteed to be logic '1' |

![VTC with Noise Margin Bands Marked](YOUR_IMAGE_URL — friend's image: VTC with Vol/Vil/Vih/Voh marked)

![Noise Margin Scale Diagram](YOUR_IMAGE_URL — friend's image: NMH and NML bands diagram)

The two margin values are:

```
NMH = Voh − Vih    ← headroom against noise on a logic '1'
NML = Vil − Vol    ← headroom against noise on a logic '0'
```

![NMH and NML Equations](YOUR_IMAGE_URL — friend's image: NMH NML equations)

**Noise hazard classification:**

![Noise Bump Severity Levels](YOUR_IMAGE_URL — friend's image: noise bump hazard diagram)

A glitch that stays within NML is absorbed safely — still decoded as '0'. A glitch pushing into the undefined transition band risks a logic error. One crossing the NMH band will definitely be decoded as '1' and represents a failure condition.

**Noise margins across PMOS sizing:**

| PMOS size | NMH | NML | Vm |
|-----------|-----|-----|----|
| 1× (Wn/Ln) | 0.30 V | 0.30 V | 0.99 V |
| 2× (Wn/Ln) | 0.35 V | 0.30 V | 1.20 V |
| 3× (Wn/Ln) | 0.40 V | 0.30 V | 1.25 V |
| 4× (Wn/Ln) | 0.42 V | 0.27 V | 1.35 V |
| 5× (Wn/Ln) | 0.42 V | 0.27 V | 1.40 V |

Widening PMOS raises Voh and thus NMH (stronger pull-up sustains the high output level). At large PMOS ratios NML dips slightly because the relatively weaker NMOS can no longer pull the output firmly to GND. The NMH span of roughly 120 mV across all sizing options sits comfortably within design margins — an explicit demonstration of CMOS inverter robustness.

![Digital vs Analog Operating Zones on VTC](YOUR_IMAGE_URL — friend's image: digital vs analog regions on VTC)

---

### Lab — Day 4

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0   0   sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in  0 1.8V

* ─── Simulation ──────────────────────────────────────────
.op
.dc Vin 0 1.8 0.01

.control
  run
  setplot dc1
  display
.endc
.end
```

![Day 4 — NGSpice Terminal](YOUR_IMAGE_URL — friend's image: Day4 NGSpice terminal output)

![Day 4 — VTC Output for Noise Margin Extraction](YOUR_IMAGE_URL — friend's image: Day4 VTC plot with cursor)

**Extracting NMH and NML manually:**

1. Click the curve near the top where the slope first approaches −1 → record `(x0, y0)` — these give **Vil = x0** and **Voh = y0**
2. Click near the bottom where slope again approaches −1 → record `(x1, y1)` — these give **Vih = x1** and **Vol = y1**
3. `NMH = Voh − Vih`; `NML = Vil − Vol`

For this run: `x0=0.767, y0=1.714, x1=0.977, y1=0.111` → **NMH = 0.737 V, NML = 0.656 V**

---

## Day 5 — Supply & Device Variation

### Power Supply Scaling

Technology scaling reduces both transistor dimensions and supply voltage hand-in-hand. Moving from 250 nm (Vdd ≈ 2.5 V) to the SKY130 130 nm node (Vdd ≈ 1.8 V) is not just geometry — it reshapes the inverter's electrical characteristics. Pushing Vdd even lower (e.g., 0.5 V) introduces a trade-off:

| Aspect | Low Vdd advantage | Low Vdd disadvantage |
|--------|-------------------|----------------------|
| Gain | ~50% higher | — |
| Energy | ~90% lower | — |
| Speed | — | Limited voltage swing, insufficient to fully charge/discharge load |

---

### Lab — Day 5 (Supply)

The following script sweeps Vdd from 1.8 V down to 0.8 V in 0.2 V steps and overlays all six VTC curves on one plot:

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0   0   sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in  0 1.8V

.control
  let powersupply = 1.8
  alter Vdd = powersupply
  let voltagesupplyvariation = 0
  dowhile voltagesupplyvariation < 6
    dc Vin 0 1.8 0.01
    let powersupply = powersupply - 0.2
    alter Vdd = powersupply
    let voltagesupplyvariation = voltagesupplyvariation + 1
  end
  plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in
  + xlabel "input voltage(V)" ylabel "output voltage(V)"
  + title "Inverter VTC vs Supply Voltage"
.endc
.end
```

![Day 5 Supply — NGSpice Terminal](YOUR_IMAGE_URL — friend's image: Day5 supply variation NGSpice terminal)

![Day 5 Supply — VTC Family of Curves](YOUR_IMAGE_URL — friend's image: Day5 supply variation VTC overlay plot)

**Gain calculation (Vdd = 1.8 V curve):**

Click two points where the VTC slope is steepest (near the −1 slope transitions):
- Top point: `(x0, y0) = (0.767, 1.714)`
- Bottom point: `(x1, y1) = (0.983, 0.100)`
- Gain = |Δy/Δx| = |1.614 / −0.216| ≈ **7.47**

---

### Manufacturing Variation

Even when a circuit is designed for a single nominal operating point, fabricated devices will drift from their drawn specifications due to two unavoidable process imperfections:

**1. Lithographic and etch variation:** The steps that define polysilicon gate width and diffusion geometry are never perfectly repeatable. Each run shifts the effective W/L of every transistor slightly from its drawn value, directly modulating Id and hence switching delay.

**2. Gate oxide thickness variation:** Thermal oxidation produces a nominally uniform tox, but in practice tox varies spatially across the wafer. Since Cox = εox/tox, and Id ∝ Cox, thickness fluctuations translate directly into drain current variation — and therefore timing variation.

**Impact in an inverter chain:**

![Single Inverter Layout](YOUR_IMAGE_URL — friend's image: single inverter layout/cell view)

![Inverter Chain Layout](YOUR_IMAGE_URL — friend's image: inverter chain layout)

Gates in the middle of a chain are flanked by identical structures, so their fabrication distortions tend to cancel. Edge gates connect to dissimilar structures and may experience different systematic shifts.

**Extreme process corners:**

| Corner | PMOS | NMOS | Vm |
|--------|------|------|----|
| Weak PMOS / Strong NMOS | Narrow, high-R | Wide, low-R | ~0.7 V |
| Strong PMOS / Weak NMOS | Wide, low-R | Narrow, high-R | ~1.4 V |

Even at these extremes, Vm shifts only within the 0.7 V–1.4 V range — the inverter continues to invert correctly across the full span. Noise margins at the corners: NMH spans ≈400 mV, NML spans ≈300 mV — both well within acceptable bounds, confirming CMOS's intrinsic robustness to process variation.

---

### Lab — Day 5 (Device)

This simulation uses an extremely wide PMOS (w=7) against a minimum-width NMOS (w=0.42) to emulate the **Strong PMOS / Weak NMOS** corner:

```spice
* ─── Model ───────────────────────────────────────────────
.param temp=27
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* ─── Netlist ─────────────────────────────────────────────
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7    l=0.15
XM2 out in 0   0   sky130_fd_pr__nfet_01v8 w=0.42 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in  0 1.8V

* ─── Simulation ──────────────────────────────────────────
.op
.dc Vin 0 1.8 0.01

.control
  run
  setplot dc1
  display
.endc
.end
```

![Day 5 Device — NGSpice Terminal](YOUR_IMAGE_URL — friend's image: Day5 device variation NGSpice terminal)

![Day 5 Device — VTC Shifted Right](YOUR_IMAGE_URL — friend's image: Day5 device corner VTC shifted plot)

The heavily oversized PMOS pulls the VTC rightward. Zooming in near the Vout = Vin crossing yields: `x0 ≈ y0 ≈ 0.988 V` — confirming Vm ≈ **0.988 V** for this corner.

---
