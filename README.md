# CMOS Circuit Design and SPICE Simulation using SKY130 Technology

---

## 📅 Day 1 — Basics of NMOS: Drain Current (Id) vs Drain-to-Source Voltage (Vds)

### 🎯 Objective
Understand NMOS device physics from threshold voltage to saturation. Build the SPICE netlist for an NFET, run Id–Vds simulation using SKY130 models in ngspice, and verify resistive and saturation region behavior.

---

## Section 1 — Introduction to Circuit Design and SPICE Simulations

### L1 — Why Do We Need SPICE Simulations?

In digital circuit design, the fundamental building block is PMOS/NMOS connected to perform specific logic functions (Inverters, Buffers, Logic Gates).

**The Core Problem:**
- Circuit delay depends on the **W/L ratio** of PMOS/NMOS
- W/L ratio controls the drain current → drain current controls the delay profile
- To tune delay → tune W/L → requires SPICE Simulation

**Delay Table Use Case:**
> CTS buffer driving different capacitive loads. Goal: build a **delay table** to avoid running SPICE every time.
> - Rows → Input Slew | Columns → Output Capacitive Load | Each cell = Delay value
> - Interpolation used when exact (slew, load) pair is not in table

**3 Key Questions answered by this course:**
1. Where do delay table values come from?
2. How accurate are they?
3. How do you validate and trust the static timing values?

**Answer:** SPICE Simulation + Circuit Design + Characterization of PMOS/NMOS.

---

### L2 — Introduction to Basic Element: NMOS

**NMOS Structure:** 4 terminals — Source (S), Drain (D), Gate (G), Body (B) — built on P-type substrate with N+ doped source/drain regions.

**Initial Condition (VGS = 0):** Both P-N junctions are reverse biased. No conducting path between Source and Drain. Channel resistance → very high.

---

### L3 — Strong Inversion and Threshold Voltage

| VGS Level | What Happens |
|---|---|
| VGS = 0 | No channel |
| Small +VGS | Depletion region forms under gate oxide |
| VGS increases | Depletion widens; minority carriers attracted to surface |
| VGS = **Vt** | Surface inversion — P-type becomes N-type → **channel forms** |
| VGS > Vt | Depletion width stops; channel width grows with VGS |

**Threshold Voltage (Vt):** Minimum VGS required to create a conducting inversion channel between Source and Drain.

---

### L4 — Threshold Voltage with Positive Substrate Potential (Body Effect)

| Condition | Effect |
|---|---|
| VSB = 0V | Normal inversion — channel forms at Vt |
| VSB = +VE | Wider depletion near source — higher VGS required → **Vt increases** |

**Threshold Voltage Equation:**

<img width="816" height="886" alt="image" src="https://github.com/user-attachments/assets/062d2ab8-8dbd-495f-bd10-54381d0c6747" />


*Figure 1: Vt = Vto + γ(√|−2ΦF + Vsb| − √|−2ΦF|). Body effect coefficient γ = √(2qNA·εsi)/Cox. Linear and saturation Id equations shown below.*

All constants (γ, φF, Cox, Vt0) are **foundry-provided** — fed into the SPICE model file.

---

## Section 2 — NMOS Resistive and Saturation Region of Operation

### L5 — Resistive Region of Operation

**MOSFET Operating Regions:**

| Condition | Region |
|---|---|
| VGS < Vt | Cut-off — Id = 0 |
| VGS > Vt, VDS ≤ VGS−Vt | Linear / Resistive |
| VGS > Vt, VDS = VGS−Vt | Pinch-off boundary |
| VGS > Vt, VDS > VGS−Vt | Saturation |

**Channel voltage at any point x: Vgs − V(x)**
- Source end: V(x) = 0 → effective channel voltage = Vgs
- Drain end: V(x) = Vds → effective channel voltage = Vgs − Vds

<img width="2136" height="1039" alt="image" src="https://github.com/user-attachments/assets/0baa2499-4107-4cce-8cb7-072137b8e85d" />

*Figure 2: NMOS cross-section with channel voltage V(x) concept. Table for Vgs=1V, Vt=0.45V — red rows = linear region where Vgs−Vds > Vt.*

---

### L2 — Drift Current Theory

Current in channel = **Drift current** due to potential difference across channel ends. Carrier velocity is non-constant — varies with position due to voltage gradient.

---

### L3 — Drain Current Model: Linear Region

```
Id = µn · Cox · (W/L) · [(Vgs - Vt)·Vds − Vds²/2]
   = kn · [(Vgs - Vt)·Vds − Vds²/2]

kn' = µn · Cox           (process transconductance)
kn  = kn' · (W/L)        (gain factor)

Condition: Vds ≤ (Vgs - Vt)

For small Vds → Vds² ≈ 0:
Id ≈ kn · (Vgs - Vt) · Vds    [linear in Vds]
```

<!-- 
=============================================================
📌 PASTE SCREENSHOT HERE → Figure 3
   File: NgspiceSky130_Day_1.docx
   What to paste: Slide showing worked Id calculation
                  Id = kn·[(1−0.45)·0.05 − 0.05²/2] = kn·[0.0275 − 0.00125]
                  Vds²/2 shown ≈ 0, confirming linear region
                  Vgs sweep values: 0.5V, 1V, 1.5V, 2V, 2.5V listed below
                  
<img width="2160" height="852" alt="image" src="https://github.com/user-attachments/assets/6f531a44-ffd9-4f06-b892-ded1b21d97e8" />

*Figure 3: Worked example — Vgs=1V, Vt=0.45V. Vds²/2 shown ≈ 0. Confirms Id is linear function of Vds for small Vds.*

---

### L4 — SPICE Conclusion to Resistive Operation

To observe Id–Vds in linear region: fix VGS at multiple values, sweep VDS from 0 to (VGS−Vt), verify Id–VDS curves with SPICE output.

---

### L5 — Pinch-Off Region Condition

```
Pinch-off condition:   Vgs − Vds = Vt
                       VDS(sat)  = VGS − Vt
```

<!-- 
=============================================================
📌 PASTE SCREENSHOT HERE → Figure 4
   File: NgspiceSky130_Day_1.docx
   What to paste: Slide showing NMOS cross-section + table
                  Rows where (Vgs−Vds) < Vt highlighted (red box bottom section)
                  Label: "Pinch-Off region — no channel near drain region"

<img width="651" height="319" alt="image" src="https://github.com/user-attachments/assets/87efe229-0022-4447-9e0d-dbdb409a2d5f" />


*Figure 4: Pinch-off identified — rows where (Vgs−Vds) < Vt. Condition: Vgs − Vds ≤ Vt.*

<img width="949" height="949" alt="image" src="https://github.com/user-attachments/assets/d768fc1a-1950-4956-bd35-08364613a795" />


*Figure 5: NMOS cross-section at pinch-off — channel begins to disappear at drain end. Current does not stop but loses VDS linearity.*

---

### L6 — Drain Current Model: Saturation Region

**Post pinch-off — channel voltage clamped at Vgs − Vt:**

<img width="1256" height="937" alt="image" src="https://github.com/user-attachments/assets/0acda7e1-3556-4939-a92c-b22e9fc08271" />


*Figure 6: In saturation, voltage over channel remains constant = Vgs − Vt, independent of Vds.*

<img width="333" height="337" alt="image" src="https://github.com/user-attachments/assets/d2f0990c-babd-4c23-b78c-3e6e9d48916d" />


*Figure 7: Replacing VDS with (Vgs−Vt) in linear equation gives saturation Id = (kn/2)·(Vgs−Vt)².*

```
Ideal (1st order):
  Id = (Kn'/2) · (W/L) · (Vgs − Vt)²

With channel length modulation:
  Id = (Kn'/2) · (W/L) · (Vgs − Vt)² · [1 + λ·Vds]
```

<img width="1383" height="264" alt="image" src="https://github.com/user-attachments/assets/5c55a884-5bd4-4749-bd5f-d5d6f259cbc8" />


*Figure 8: More accurate saturation equation including λ (channel length modulation). λ is a foundry-provided constant.*

---

## Section 3 — Introduction to SPICE

### L1 — Basic SPICE Setup

| SPICE Input | Description |
|---|---|
| Model equations | MOSFET device behavior |
| Technological constants | λ, γ, Vt0, kn', Cox — from foundry |
| Model files (.mod / .lib) | All device constants packaged |
| SPICE Netlist | Circuit connectivity |

<img width="2241" height="1008" alt="image" src="https://github.com/user-attachments/assets/2d4d0166-3a9c-4d37-ad09-b45073b9f977" />


*Figure 9: SPICE flow — netlist + model parameters → SPICE engine → Id–Vds family of curves.*

---

### L2 — Circuit Description in SPICE Syntax

| Prefix | Component |
|---|---|
| `M` | MOSFET → order: Drain Gate Source Substrate |
| `R` | Resistor |
| `V` | Voltage Source (positive terminal first) |
| `C` | Capacitor |

<img width="641" height="243" alt="image" src="https://github.com/user-attachments/assets/a9e4dfbc-a94f-4082-a997-2eb98da95ad7" />


*Figure 10: NMOS structure mapped to SPICE circuit. Nodes: vdd, n1, in, 0. Protection resistor R1 at gate.*

<img width="899" height="370" alt="image" src="https://github.com/user-attachments/assets/889ab338-20ed-4ae0-b3e7-5add190f3794" />


*Figure 11: SPICE netlist (left) linked to model parameter equations (right). Model name in netlist must exactly match `.MODEL` definition.*

```spice
M1 vdd n1 0 0 nmos W=1.8u L=1.2u
R1 in n1 55
Vdd vdd 0 2.5
Vin in 0 2.5
```

---

### L3 — Define Technology Parameters

<img width="1126" height="350" alt="image" src="https://github.com/user-attachments/assets/a4a03e42-7625-4722-98b7-ad0e17a0ec00" />


*Figure 12: `.MODEL` blocks for NMOS and PMOS — TOX, VTH0, U0, GAMMA1 parameters listed.*

<img width="891" height="555" alt="image" src="https://github.com/user-attachments/assets/c618b484-95d1-47c5-a039-f1506488c460" />


*Figure 13: Model parameters packaged inside `.lib cmos_models` → saved as `xxxx_025um_model.mod`.*

<!-- 
=============================================================
📌 PASTE SCREENSHOT HERE → Figure 14
   File: NgspiceSky130_Day_1.docx
   What to paste: Black terminal slide showing full netlist with:
                  *** NETLIST Description ***
                  M1 vdd n1 0 0 nmos W=1.8u L=1.2u
                  R1 in n1 55
                  .LIB "xxxx_025um_model.mod" CMOS_MODELS
                  Label: "Model file description"
=============================================================
-->
![Figure 14 – Netlist with .LIB include](paste_figure_14_here)

*Figure 14: Complete netlist with `.LIB "xxxx_025um_model.mod" CMOS_MODELS` inclusion.*

```spice
.LIB "xxxx_025um_model.mod" CMOS_MODELS
```

---

### L4 — First SPICE Simulation — Environment Setup

<img width="1279" height="1112" alt="image" src="https://github.com/user-attachments/assets/a7c603e4-d513-4ca8-9a86-1dcdf656f5ff" />


*Figure 15: VSDWorkshop VM — Ubuntu 18.04, 4GB RAM, SATA: 7nm.vdi (25GB). VM password: vsdiat.*

<img width="1780" height="193" alt="image" src="https://github.com/user-attachments/assets/e1bd24ac-08f9-431b-923c-4524e883f7f5" />


*Figure 16: `cells/nfet_01v8/` directory — ff, fs, sf, ss, tt corner files. `tt.pm3.spice` selected for typical corner.*

<img width="1794" height="1125" alt="image" src="https://github.com/user-attachments/assets/63896135-6287-412b-ae72-8ebf5c133a5b" />


*Figure 17: `sky130_fd_pr__nfet_01v8__tt.pm3.spice` content — subckt `d g s b`, model params: lmin=2e-5, lmax=1e-4, wmin=7e-6, wmax=1e-4, level=54.*

---

### L5 — SPICE Lab with SKY130 Models

<img width="1534" height="128" alt="image" src="https://github.com/user-attachments/assets/c4a2af96-8ae0-4ae2-b8f3-b20ca9aca9b2" />


*Figure 18: `cd ../../models/` → `ls` shows: `all.spice`, `parameters`, `sky130.lib.spice`.*

<img width="1096" height="792" alt="image" src="https://github.com/user-attachments/assets/a467732d-e238-41fd-a58c-c2b86dc42289" />


*Figure 19: `sky130.lib.spice` — `tt` corner includes nfet/pfet corner + mismatch + all.spice. Pattern repeats for sf, ff, ss corners.*

<img width="1499" height="152" alt="image" src="https://github.com/user-attachments/assets/53be3854-7b65-47b9-bb73-2431bc8bb195" />


*Figure 20: Design folder — day1 through day5 SPICE files. `day1_nfet_idvds_L025_W065.spice` opened with vim.*

<img width="758" height="1056" alt="image" src="https://github.com/user-attachments/assets/99935a8f-d2db-4d4c-bd62-5e4e8fca4b4a" />


*Figure 21: `day1_nfet_idvds_L025_W065.spice` in vim — model include (tt corner), XM1 (w=0.65, l=0.25), R1=55Ω, Vdd=Vin=1.8V, DC sweep.*

**Exact SPICE netlist from the file:**

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.65 l=0.25

R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
run
display
setplot dc1
.endc

.end
```

**Run the simulation:**

```bash
ngspice day1_nfet_idvds_L025_W065.spice
```

**Plot inside ngspice:**

```bash
ngspice 1 -> plot -vdd#branch
```

<img width="1408" height="986" alt="image" src="https://github.com/user-attachments/assets/41f67c1e-66f0-4ee4-9088-5772fce36abc" />


*Figure 22: ngspice-27 startup. Simulation at TEMP=27°C. Active vectors: vdd#branch, vin#branch. 190 data rows generated.*

<img width="710" height="565" alt="image" src="https://github.com/user-attachments/assets/1df40dbf-5c61-4d2c-a3df-a68528cce50a" />


*Figure 23: **Key Result** — Id vs Vds family of curves. nfet_01v8, W=0.65µm, L=0.25µm, SKY130 tt corner. Each curve = different Vgs (0→1.8V, step 0.2V). Linear and saturation regions clearly visible.*

---

## 📌 Key Observations — Day 1

- NMOS has 3 operating regions: Cut-off, Linear, Saturation — each governed by Vgs and Vds relative to Vt
- Body effect (VSB > 0) widens depletion near source → requires higher Vgs to form channel → Vt increases
- In linear region, Vds² term ≈ 0 for small Vds → Id simplifies to linear function of Vds
- Pinch-off at VDS = VGS − Vt — channel disappears at drain end; Id plateaus into saturation region
- In saturation, first-order model shows Id independent of Vds — λ term captures real channel length modulation
- SKY130 `sky130.lib.spice` organizes all PVT corners (tt, ff, ss, sf, fs) — correct corner must be called in `.lib`
- Protection resistor R1 = 55Ω at gate node is mandatory in SPICE setup
- Pre-characterized W/L values must be used with SKY130 models — non-standard values produce errors

---
