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

![NMOS Cross-Section](https://private-user-images.githubusercontent.com/261861039/551139383-ae7298a9-9096-41dc-8ace-e13aaa7b9e25.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxMTM5MzgzLWFlNzI5OGE5LTkwOTYtNDFkYy04YWNlLWUxM2FhYTdiOWUyNS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1lOTkzYzQwYjVmYjg5YWM1ZTNmYTk5ZjRhMWE5NTNjM2ZlODZhNmFkN2M2MzEzMTNlN2IwODY1MzZmOGQ0NTNkJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.u_9EHw1bksF-XXCMGPLwoSua0XHfskj_I-LxLaGsBjE)

The ratio of channel **W**idth to **L**ength (W/L) is the single most important sizing parameter — it sets drive current, switching speed, and input capacitance.

---

### Strong Inversion & Threshold Voltage

**At rest (Vgs = 0 V):** With drain, source, and body all at ground, the B-S and B-D junctions are unbiased reverse-junction diodes. No channel forms and S-D resistance is extremely high.

**Ramping up Vgs:** The positive gate voltage attracts electrons from the n+ regions toward the surface under the gate oxide, gradually creating a thin electron-rich layer.

![Effect of Increasing Vgs](https://private-user-images.githubusercontent.com/261861039/551142878-d76c1ebc-d734-4a2c-b6d9-db48ea69f849.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxMTQyODc4LWQ3NmMxZWJjLWQ3MzQtNGEyYy1iNmQ5LWRiNDhlYTY5Zjg0OS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT00MjU2NTQxNDJiNjFkMGU0NzE1MjJkMzg3MGU2NzFmMzI4MTBlMGEyNWZmYzViYmEyODU5YmE3MmViYmVmNzA4JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.2yaWuwDX-P3nrxayic6xHpc978ujN_04PBBIaR4GoSw)

**Strong Inversion:** Once Vgs crosses a critical level, the surface beneath the gate flips from p-type to n-type — forming a complete conducting channel between source and drain. This transition is called **strong inversion**, and the gate voltage at which it occurs is the **Threshold Voltage (Vt)**.

![Channel Formation at Strong Inversion](https://private-user-images.githubusercontent.com/261861039/551156645-85ae483b-047e-44d9-b0e5-321dc044f7cc.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxMTU2NjQ1LTg1YWU0ODNiLTA0N2UtNDRkOS1iMGU1LTMyMWRjMDQ0ZjdjYy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1lYTEzMGNlMjYwZjJkODc1Yjk5NGI0NDJkYmUxMjdmNjRmMzExNGIxODhlMzQ0MjM3ZTcwNDhjZDIxMmEyMWMzJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.t-HszDQaE7P894e_qreDJ7V9SjndDL6PcTB6hmXVIA8)

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

![Depletion Region with Body Bias](https://private-user-images.githubusercontent.com/261861039/551164619-5c67337a-9a60-4b19-8bdf-65f6ac4c241c.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxMTY0NjE5LTVjNjczMzdhLTlhNjAtNGIxOS04YmRmLTY1ZjZhYzRjMjQxYy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1hYThmZTY4YWI2Zjk3NzQ1ZTZhM2M2MGY0YzcxZmY2YzE3MWFlNDFkNWI5OGUxMjQyMzUwZDA4YzNmYjkyMzhmJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.4PfML1OJ4EY0w2tAnQWnKnfvevMTMiKWtqu5DSPjDGE)

The wider depletion region pulls electrons away from the channel-forming surface, so **more gate voltage is needed** to achieve inversion. In other words, Vt increases with Vsb — this is the **body effect** (also called the back-gate effect).

![Threshold Shift Due to Body Bias](https://private-user-images.githubusercontent.com/261861039/551336288-3932925f-0ae3-4710-b48d-48f3ca2b74ce.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxMzM2Mjg4LTM5MzI5MjVmLTBhZTMtNDcxMC1iNDhkLTQ4ZjNjYTJiNzRjZS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT04MDhjNTRhYWVlNzQxZDAwMDVkMjU2ODcyMThjYzZhNjNiYmMyNjBhM2E5MzRhZDVkZWMyZGE1MGZkZWQxYjMzJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.8kgGbpA-TRa9mRD-rYTqXM1TAiXbtOW5isB7UNMybs8)

![Threshold Voltage Equation with Vsb](https://private-user-images.githubusercontent.com/261861039/551337452-39fc86c7-b6cb-4913-81c7-b2e8931d0f08.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxMzM3NDUyLTM5ZmM4NmM3LWI2Y2ItNDkxMy04MWM3LWIyZTg5MzFkMGYwOC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1mOWY5MjA3YWE1NmFmMzgwMDhlYjQ2MWFkOWUxZmU0ZmRiMzU5ODg2NmU4YjI2YmY5N2JlMTY5ZWFiNjljMTVmJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.gmD9VeY92dEX-86fy2x53GK01wp1I39jUtseqtV37d8)

The body-effect coefficient γ and Fermi potential φf are not design choices — they come from foundry characterization. SKY130's SPICE model files encode these values precisely.

![Foundry-Supplied SPICE Parameters](https://private-user-images.githubusercontent.com/261861039/551339632-f294e8af-d5e9-4937-9a81-0286b7a673c5.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxMzM5NjMyLWYyOTRlOGFmLWQ1ZTktNDkzNy05YTgxLTAyODZiN2E2NzNjNS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1lNDJlYzkzMDIyNGZmZWM1ZDZiZTkxYmUwM2FiOTE5NjI3ODdhN2VlZDFhYmM1N2VmNmE2YzI5MDY3NDczNjQ0JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.Fm_0gQAhn2TTnZUth0QNRjE8_ji1x-StDNoZjXgx61o)

---

### Linear Region of Operation

The MOSFET has three primary regions: **cutoff** (OFF), **linear/resistive**, and **saturation**.

In the **linear region** (Vgs > Vt, small Vds), a conducting channel spans the full length from source to drain. As Vgs increases, the channel thickens and its conductance rises.

![MOSFET in Linear Region](https://private-user-images.githubusercontent.com/261861039/551424597-11e7e4f6-9632-4c1e-83b0-548751e68575.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxNDI0NTk3LTExZTdlNGY2LTk2MzItNGMxZS04M2IwLTU0ODc1MWU2ODU3NS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0wOTlhODE5MWZjNzU5MzI0MmI3OTgxMGE3YzJlMzFlNDljMGU2YTc0MTRmODBlODBmNDc4YTAxYWQ0MDZlOGVmJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.bVr_d3f8I7Lf1uijq1ysSMOMv-98EFUrLXudPdmq7qw)

With Vds applied, the channel is no longer uniform — a voltage gradient V(x) builds along its length (x-axis). Using Vgs = 1 V, Vt = 0.45 V, Vds = 0.05 V as an example:
- At the source end (x=0): effective gate overdrive = Vgs − V(0) = 1.0 V
- At the drain end (x=L): effective gate overdrive = Vgs − V(L) = 0.95 V

The local induced charge at any point x is therefore: `Qi(x) = −Cox · (Vgs − V(x) − Vt)`

![Voltage Gradient Along the Channel](https://private-user-images.githubusercontent.com/261861039/551426229-33bbd97c-3bd1-415e-a632-85c6f3c3d70e.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxNDI2MjI5LTMzYmJkOTdjLTNiZDEtNDE1ZS1hNjMyLTg1YzZmM2MzZDcwZS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1jOWE4ODI4NDhlOTU4YWQ5M2NlODc2ZDBlZTY2ODFmNTlmN2M2YzhhMjE3MzAyOWVjNDYyMDBlZGFkMzhiYzM1JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.v316jOzWTBbGiK7-cX5stpmGfR57y6ddx0KFw4lKNEk)

![Current Equation in Linear Region](https://private-user-images.githubusercontent.com/261861039/551437238-77a3cccc-7699-4145-a0a0-f864fedc6522.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxNDM3MjM4LTc3YTNjY2NjLTc2OTktNDE0NS1hMGEwLWY4NjRmZWRjNjUyMi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT01OTY5Nzk2MTc4ZTFkZTVmM2MzMGU2ZGVjZjdkNmUzNzE3NzVkMmRiZGE3N2M2MzRiNzlmODZhZDhlNjMwZmU0JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.tPm_yxNyGCO4BhaMFLp3tG8MWkh60ErpTtONMWOTFFE)

**Operating region boundary:** Vds ≤ (Vgs − Vt) → linear; once Vds crosses this threshold, the device enters saturation.

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

![Drift Current Formula](https://private-user-images.githubusercontent.com/261861039/551460541-170b037c-2adc-4538-966c-f208cf52ebbd.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NzIwNjIxMjQsIm5iZiI6MTc3MjA2MTgyNCwicGF0aCI6Ii8yNjE4NjEwMzkvNTUxNDYwNTQxLTE3MGIwMzdjLTJhZGMtNDUzOC05NjZjLWYyMDhjZjUyZWJiZC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjYwMjI1JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI2MDIyNVQyMzIzNDRaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0wMzc0ODVjYmUwZDVhNmIwYjNmMTM5N2I1MjJkYzFiYzY2MTY4NmQ5YzYyZWQyNGViMTYxZWMzYzcyNDQwMWJlJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.4KmP98Zsr2Xa1AdYXOaeQBs6nt1itQ95ZAI3aQr2G_s)

---

### Saturation & Pinch-Off

As Vds increases toward (Vgs − Vt), the overdrive at the drain end approaches zero. At exactly Vgs − Vds = Vt, the channel "pinches off" at the drain. Beyond this point, the channel adopts a triangular shape — still connected at the source, absent at the drain.

The current no longer grows with Vds; it saturates at:

```
Id(sat) = (kn/2) · (Vgs − Vt)² · (1 + λ·Vds)
```

The `(1 + λ·Vds)` term captures **channel-length modulation** — as Vds pushes beyond pinch-off, the depletion region at the drain expands, effectively shortening the channel and allowing Id to increase very slowly. λ is a process-dependent parameter provided in SPICE model files.

---

### SPICE Primer

SPICE reads a **netlist** describing the circuit topology, device models, and simulation commands. Key workflow elements:

> **SPICE Simulation Flow**

![SPICE Workflow Diagram](https://user-images.githubusercontent.com/89193562/132533155-7affa537-beb3-4aa4-8eab-b4ff3aaab64d.JPG)

SKY130 provides models for five statistical **process corners**:

| Corner Code | Meaning |
|-------------|---------|
| `tt` | Typical NMOS / Typical PMOS |
| `ff` | Fast NMOS / Fast PMOS |
| `ss` | Slow NMOS / Slow PMOS |
| `sf` | Slow NMOS / Fast PMOS |
| `fs` | Fast NMOS / Slow PMOS |

A **node** is any wire junction connecting two or more device terminals. The SPICE netlist assigns a name to every node; all unlabelled nodes default to 0 (GND).

![NMOS SPICE Netlist Example](https://user-images.githubusercontent.com/89193562/132711027-1aa941dc-56bc-4be9-af32-5a96b76d9c09.jpg)

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

![Day 1 — NGSpice Terminal](https://user-images.githubusercontent.com/89193562/132533223-85fac5e7-3073-43fc-9d14-a248e9116a2e.JPG)

![Day 1 — Id vs Vds Output Plot](https://user-images.githubusercontent.com/89193562/132533338-e2298388-5d87-49a2-b5e2-6156ce69c46d.JPG)

> 💡 **Reading a value:** Left-click any point on the curve. The terminal prints `x0` (Vds) and `y0` (Id in amps).

---

## Day 2 — Velocity Saturation & CMOS VTC

### Short-Channel Effects

**Regions on the Id–Vds plane:**

The Id–Vds family of curves maps out all three regions at once:

![Id vs Vds — All Regions](https://user-images.githubusercontent.com/89193562/132864852-2f667ae5-a71c-4e67-975a-e4c137843114.png)

The Vgs = 0 trace hugs the x-axis because the device is fully off. Higher Vgs traces show clear linear and saturation regions. The transition between the two occurs at the pinch-off boundary (Vds = Vgs − Vt).

**Why velocity saturates at short channel lengths:**

In long devices, carrier velocity scales linearly with electric field. In short-channel devices (L < 250 nm), the field becomes high enough to push carriers into a **velocity saturation** regime — velocity stops increasing and plateaus at a critical value.

![Velocity Saturation Equation](https://user-images.githubusercontent.com/89193562/132674315-002da47e-65d4-4976-b2c0-b309dee76df7.JPG)

![Velocity vs. Electric Field](https://user-images.githubusercontent.com/89193562/132679374-baa32830-fcca-49c3-be54-10b5caf2c5d3.png)

This adds a **fourth operating region** to short-channel devices (L < 250 nm):

| Channel Length | Regions |
|----------------|---------|
| Long (> 250 nm) | Cutoff → Linear → Saturation |
| Short (< 250 nm) | Cutoff → Linear → **Velocity Saturation** → Saturation |

The unified drain current model uses `Vmin = min(Vgt, Vds, Vdsat)`:
```
Id = µn·Cox·(W/L)·(Vmin²/2)·(1 + λ·Vds)
```

`Vdsat` is a **process-only parameter** — it is the drain voltage at which carriers saturate, independent of bias conditions. Devices at smaller nodes therefore saturate at lower Vds values, making velocity saturation a dominant effect in modern process nodes.

---

### Lab — Day 2

**Part A — Id vs Vds for short-channel NMOS (w=0.39 µm, l=0.15 µm):**

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

![Day 2A — NGSpice Terminal (Id vs Vds)](https://user-images.githubusercontent.com/89193562/132675164-206b1eeb-8cba-44a8-af4f-bf4322e37550.JPG)

![Day 2A — Id vs Vds Short Channel](https://user-images.githubusercontent.com/89193562/132675399-e8f69dc7-f222-4e91-81fc-4cb2639213d4.JPG)

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

![Day 2B — NGSpice Terminal (Id vs Vgs)](https://user-images.githubusercontent.com/89193562/132675473-18cd0d22-a956-4c4a-978b-e4837c292d70.JPG)

![Day 2B — Id vs Vgs (Vt extraction)](https://user-images.githubusercontent.com/89193562/132675655-f779b9be-bcee-4d31-8a62-6204bc0bca40.JPG)

> 💡 **Finding Vt:** Extend the straight-line portion of the Id–Vgs curve until it crosses the x-axis. That x-intercept is Vt.

---

### CMOS Inverter & VTC

**Transistor switching model:** Each transistor can be approximated as a voltage-controlled switch — infinite impedance below Vt, finite resistance above it.

![CMOS Inverter Circuit](https://user-images.githubusercontent.com/89193562/132681895-fe353e35-c49a-4fcf-a822-640a20898861.jpg)

When Vin = 0 V, the PMOS gate is pulled low (|Vgsp| > |Vtp|) turning it on; the NMOS gate is at zero (Vgsn < Vtn) keeping it off — output charges to Vdd. When Vin = Vdd the roles reverse and the output discharges to GND.

The voltage relationships used to draw the load curves are:

![NMOS Voltage Equations](https://user-images.githubusercontent.com/89193562/132678807-2bcbfa75-4081-46cb-899d-7b3915c62688.JPG)

![PMOS Voltage Equations](https://user-images.githubusercontent.com/89193562/132678852-68fd02e7-f396-45f5-8ea3-3b2645c71372.JPG)

![KCL Current Constraint](https://user-images.githubusercontent.com/89193562/132678900-8087d81b-0588-46f8-8fc9-11e51725ebdd.JPG)

Plotting the PMOS and NMOS load curves separately first:

![PMOS Load Curve](https://user-images.githubusercontent.com/89193562/132907643-9423cede-796e-40b7-a8bb-36fde16d533f.jpg)

![NMOS Load Curve](https://user-images.githubusercontent.com/89193562/132907688-664d7f0e-da4e-4c36-9270-fca25f24b945.jpg)

Superimposing both curves and tracing the intersection points at each Vin value produces the **Voltage Transfer Characteristic (VTC)**:

![Superimposed Load Curves](https://user-images.githubusercontent.com/89193562/132946435-09010251-dd7e-4af4-b791-ea5655dd171f.jpg)

![Vout vs Vin — Complete VTC](https://user-images.githubusercontent.com/89193562/132859275-865ad5a6-d174-4bb0-9615-77cc5deb9992.jpg)

---

## Day 3 — Switching Threshold & Transient Analysis

### SPICE Deck for the Inverter

A complete SPICE deck needs four sections: component connectivity, component values, node names, and simulation commands.

![Reference CMOS Inverter Netlist](https://user-images.githubusercontent.com/89193562/132860957-9e9492b5-c646-4866-add5-279064e73c82.jpg)

---

### Lab — Day 3

**VTC Simulation:**

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

![Day 3 VTC — Terminal](https://user-images.githubusercontent.com/89193562/132863150-3d5b53a2-a802-43cd-aaca-1d28bca51945.JPG)

![Day 3 VTC — Output Plot](https://user-images.githubusercontent.com/89193562/132863366-e80c7b36-42af-45bb-a140-c75502008046.JPG)

> 💡 **Finding Vm:** Zoom into the region where Vout ≈ Vin. Left-click there — since Vm sits at the Vout = Vin diagonal, x0 ≈ y0 ≈ Vm.

**Transient Analysis — Rise/Fall Delay:**

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

![Day 3 Transient — Terminal](https://user-images.githubusercontent.com/89193562/132863836-2348a72e-a7e9-4ee3-9d3c-bb5c63d64183.JPG)

![Day 3 Transient — Waveform](https://user-images.githubusercontent.com/89193562/132863939-9f777f44-e10e-4bc3-a9a2-41e833dd465b.JPG)

**Delay measurement procedure:**
- **Rise delay** — Zoom near Vdd/2 on the output rising edge. Measure Δx between the output crossing Vdd/2 and the input falling through Vdd/2.
- **Fall delay** — Same approach, but for the output falling edge against the input rising edge.

---

### Switching Threshold (Vm)

Vm is the input voltage at which Vout = Vin — the midpoint of the inverter's transition. Graphically it is the intersection of the VTC with a 45° line from the origin.

At Vm, both transistors are simultaneously active: `Vgs = Vds` for each, and KCL requires `IdP + IdN = 0`.

![IdN and IdP at Vm](https://user-images.githubusercontent.com/89193562/132867914-dc5b1ac9-a4d9-452b-9352-4872c94264fe.JPG)

![Setting IdP + IdN = 0](https://user-images.githubusercontent.com/89193562/132868215-8eb427a0-1a5a-4c3f-8cd5-76c41d49d947.JPG)

Solving for Vm:

![Vm Closed-Form Expression](https://user-images.githubusercontent.com/89193562/132869184-bd60ea34-e16f-4c7b-9be7-05386549329a.JPG)

Working backwards, if a specific Vm is required, the needed PMOS-to-NMOS sizing ratio is:

![W/L Ratio for Target Vm](https://user-images.githubusercontent.com/89193562/132872201-eba019d8-5a72-480a-a44d-f4d66f718da1.JPG)

**Simulated results across PMOS widths:**

| PMOS sizing | Rise delay | Fall delay | Vm |
|-------------|-----------|------------|------|
| 1× (Wn/Ln) | 148 ps | 71 ps | 0.99 V |
| 2× (Wn/Ln) | 80 ps | 76 ps | 1.20 V |
| 3× (Wn/Ln) | 57 ps | 80 ps | 1.25 V |
| 4× (Wn/Ln) | 45 ps | 84 ps | 1.35 V |
| 5× (Wn/Ln) | 37 ps | 88 ps | 1.40 V |

Notable takeaways: the 2× sizing achieves nearly symmetrical rise/fall delays, making it the standard choice for **clock tree inverters/buffers**. Larger ratios progressively shift Vm rightward while unbalancing delays — still valid for data-path applications. Ron(PMOS) ≈ 2.5 × Ron(NMOS) explains why PMOS must be wider to match drive strength.

---

## Day 4 — Noise Margin Robustness

### Noise Margin Theory

Real digital systems ride on power supplies that carry noise. The CMOS inverter must tolerate this noise without misinterpreting a logic level. This tolerance is quantified as **noise margin**.

**Ideal vs. real transfer characteristics:**

An ideal inverter flips instantaneously at Vdd/2 with infinite slope. The real curve has a finite-slope transition region — defining four boundary voltages:

![Ideal Inverter Characteristics](https://user-images.githubusercontent.com/89193562/132940629-2c46f46d-d9a6-4c3b-9615-45f88bac7944.jpg)

![Real Inverter Characteristics](https://user-images.githubusercontent.com/89193562/132940634-1688436a-cf57-41d7-82d8-70e0512a1154.jpg)

| Voltage | Full Name | Logic Meaning |
|---------|-----------|---------------|
| Vol | Output Low | Maximum output voltage still read as '0' |
| Vil | Input Low | Maximum input voltage treated as '0' |
| Vih | Input High | Minimum input voltage treated as '1' |
| Voh | Output High | Minimum output voltage still read as '1' |

![I/O Curve with Noise Margin Bands](https://user-images.githubusercontent.com/89193562/132940893-994a25d6-d89f-401f-b98f-868a4f438ef1.jpg)

![Noise Margin Scale Plot](https://user-images.githubusercontent.com/89193562/132940899-e462e34a-643c-4864-9024-73bc997474c1.jpg)

The two margins:
```
NMH = Voh − Vih    ← tolerance for noise on a logic '1'
NML = Vil − Vol    ← tolerance for noise on a logic '0'
```

![NMH and NML Equations](https://user-images.githubusercontent.com/89193562/132941048-4d1a35eb-24bc-44dd-ba25-5a5abecd73bb.JPG)

**Noise bump hazard classification:**

![Noise Bump at Different Levels](https://user-images.githubusercontent.com/89193562/132951411-2ce7e449-b8bb-4e24-90b3-932e3344533d.jpg)

A noise glitch that stays within NML is harmless — it is still decoded as '0'. One that enters the undefined transition band may or may not flip the logic. One that reaches the NMH band will certainly be decoded as '1' and must be eliminated.

**Measured noise margins across PMOS sizing:**

| PMOS size | NMH | NML | Vm |
|-----------|-----|-----|----|
| 1× (Wn/Ln) | 0.30 V | 0.30 V | 0.99 V |
| 2× (Wn/Ln) | 0.35 V | 0.30 V | 1.20 V |
| 3× (Wn/Ln) | 0.40 V | 0.30 V | 1.25 V |
| 4× (Wn/Ln) | 0.42 V | 0.27 V | 1.35 V |
| 5× (Wn/Ln) | 0.42 V | 0.27 V | 1.40 V |

Increasing PMOS width improves NMH (stronger pull-up holds Voh high) but slightly degrades NML at large ratios (NMOS becomes relatively weaker). The total NMH swing of ~120 mV across all sizings is entirely within acceptable bounds — confirming CMOS inverter robustness.

![Digital vs Analog Applicable Regions on VTC](https://user-images.githubusercontent.com/89193562/132952201-c0a046b8-1c7e-45f3-bd8e-590efba51f59.jpg)

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

![Day 4 — NGSpice Terminal](https://user-images.githubusercontent.com/89193562/132941122-635e2440-e1a6-4df2-9218-480b45ddf61d.JPG)

![Day 4 — VTC Output for Noise Margin Extraction](https://user-images.githubusercontent.com/89193562/132941130-48c2c93d-3ad2-4a9c-acd1-c7caec2bf87c.JPG)

**Extracting NMH and NML manually:**

1. Click the curve near the top where the slope first approaches −1 → record `(x0, y0)` — these give **Vil = x0** and **Voh = y0**
2. Click near the bottom where slope again approaches −1 → record `(x1, y1)` — these give **Vih = x1** and **Vol = y1**
3. `NMH = Voh − Vih`; `NML = Vil − Vol`

For this run: `x0=0.767, y0=1.714, x1=0.977, y1=0.111` → **NMH = 0.737 V, NML = 0.656 V**

---

## Day 5 — Supply & Device Variation

### Power Supply Scaling

Technology scaling reduces both transistor dimensions and operating voltage. Moving from 250 nm (Vdd ≈ 2.5 V) to 130 nm (SKY130, Vdd ≈ 1.8 V) and lower is not just a size reduction — it changes the inverter's electrical characteristics.

Running at a reduced supply (e.g., 0.5 V) has two competing effects:

| Aspect | Low Vdd advantage | Low Vdd disadvantage |
|--------|-------------------|----------------------|
| Gain | ~50% higher | — |
| Energy | ~90% lower | — |
| Speed | — | Insufficient swing to fully charge/discharge load |

---

### Lab — Day 5 (Supply)

The following script sweeps Vdd from 1.8 V down to 0.8 V in 0.2 V steps and overlays all six VTC curves:

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

![Day 5 Supply — NGSpice Terminal](https://user-images.githubusercontent.com/89193562/132985847-4bd15226-3483-47e8-9961-1080e21564f8.JPG)

![Day 5 Supply — VTC Family of Curves](https://user-images.githubusercontent.com/89193562/132985866-6915c943-8cf2-4c48-a524-c4b4c1169147.JPG)

**Gain calculation (Vdd = 1.8 V curve):**

Click two points where slope is steepest (near −1 transitions):
- Top: `(x0, y0) = (0.767, 1.714)`
- Bottom: `(x1, y1) = (0.983, 0.100)`
- Gain = |Δy/Δx| = |1.614 / −0.216| ≈ **7.47**

---

### Manufacturing Variation

Even when designing for a single operating point, the fabricated device will differ due to two unavoidable process imperfections:

**1. Etching variation:** The lithography and etch steps that define polysilicon gate width and diffusion width are never perfectly repeatable. This shifts the effective W/L of every transistor slightly from its drawn value, directly affecting Id and therefore delay.

**2. Gate oxide thickness variation:** Ideal oxidation grows a perfectly uniform tox. In practice, tox varies spatially across the wafer. Because Cox = εox/tox, and Id ∝ Cox, every thickness fluctuation modulates drain current.

**Inverter chain context:**

![Single Inverter Layout](https://user-images.githubusercontent.com/89193562/132986827-95402e8e-0b25-40ef-b1fa-8122c6dcba7b.jpg)

![Inverter Chain Layout](https://user-images.githubusercontent.com/89193562/132986894-f6d2a87b-acf9-4bd9-8b5d-a1f98e343a39.jpg)

Gates in the middle of a chain are flanked by identical structures — their distortions repeat. Gates at the ends connect to non-identical devices and may exhibit different parametric shifts.

**Extreme process corners:**

| Corner | PMOS | NMOS | Vm |
|--------|------|------|----|
| Weak PMOS / Strong NMOS | Small, high-R | Wide, low-R | ~0.7 V |
| Strong PMOS / Weak NMOS | Wide, low-R | Small, high-R | ~1.4 V |

Even at the extreme corners, Vm shifts only from ~0.7 V to ~1.4 V — the gate still correctly inverts across the entire range, confirming CMOS's fundamental robustness to process variation. Noise margin variations across corners: NMH spans ≈400 mV, NML spans ≈300 mV — both tolerable.

---

### Lab — Day 5 (Device)

This simulation uses an extremely wide PMOS (w=7) against a minimum-width NMOS (w=0.42) to emulate the Strong PMOS / Weak NMOS corner:

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

![Day 5 Device — NGSpice Terminal](https://user-images.githubusercontent.com/89193562/132985962-a05f4559-ce5d-4af5-bf2e-67e091c577dd.JPG)

![Day 5 Device — VTC Shifted Right](https://user-images.githubusercontent.com/89193562/132985983-6313769f-5ef4-432d-87c9-ef39f7e572f0.JPG)

The heavily oversized PMOS pulls the VTC rightward. Zooming in near the Vout = Vin crossing yields: `x0 ≈ y0 ≈ 0.988 V` — confirming Vm ≈ **0.988 V** for this corner.

---

## Conclusion

This five-day workshop built a complete understanding of CMOS circuit behavior starting from first principles. Beginning with MOSFET physics — threshold voltage, body effect, drift current — and progressing through SPICE-based simulation of the full CMOS inverter lifecycle, the course delivered both theoretical grounding and hands-on fluency with the SKY130 PDK.

The central lesson reinforced throughout is the **inherent robustness of the CMOS inverter**: its noise margins, switching threshold, and logic functionality remain intact across a wide range of W/L sizing choices, supply voltages, and manufacturing process corners. This robustness is not accidental — it is a direct consequence of the complementary NMOS/PMOS topology and is best understood (and verified) through SPICE simulation.

---

## References

- https://github.com/kunalg123/sky130CircuitDesignWorkshop
- https://www.vlsisystemdesign.com/cmos-circuit-design-spice-simulation-using-sky130-technology/
- https://skywater-pdk.readthedocs.io/en/main/
- https://www.vsdiat.com/
- https://github.com/kunalg123/vsdflow
- https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax
