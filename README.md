# NV Maser "Tricorder" — Handheld MRI Probe Hardware

Physical implementation of a **handheld, maser-enhanced medical imaging probe** — a room-temperature NV-diamond maser combined with a single-sided permanent magnet, producing MRI-contrast tissue images in an ultrasound-like form factor.

> **Digital twin companion**: Simulation, ML controller, and physics validation live in → [`nv-maser-twin`](https://github.com/eric-rolph/nv-maser-twin).  
> The design review lives at → `nv-maser-twin/docs/research/handheld-maser-probe-architecture.md`

---

## Product Vision

**"MRI contrast in the palm of your hand."**

A handheld probe placed on the patient's body — like an ultrasound transducer — produces tissue-contrast images (T1, T2, proton density) within 30–120 seconds on a connected tablet. No bore, no shielded room, no cryogenics. Designed for emergency medical triage: hemorrhage detection, fracture assessment, compartment syndrome, tissue characterization.

---

## System Overview

The device consists of a **handheld probe** connected by cable to a **tablet** (with optional cloud processing). The probe contains two magnet systems: a single-sided imaging magnet and the NV maser module.

```
┌───────────── PROBE HEAD (< 1.5 kg) ──────────────────────┐
│                                                            │
│  ┌─── Maser Module (~50 mm cube) ─────────────────────┐   │
│  │  Mini Halbach (50 mT, 14 mm bore)                   │   │
│  │  NV diamond + TE₀₁₁ microwave cavity                │   │
│  │  532 nm DPSS laser (2W) + optics                     │   │
│  │  8 shimming coils + control MCU                      │   │
│  │  Up-conversion mixer (2 MHz → 1.47 GHz)              │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                            │
│  ┌─── Single-Sided Imaging Magnet ────────────────────┐   │
│  │  Sweet-spot barrel array (NdFeB N52)                │   │
│  │  B₀ = 50 mT at 20 mm depth sweet spot              │   │
│  │  Built-in gradient ~5-15 T/m (depth encoding)       │   │
│  │  Lateral gradient coils (2D in-plane encoding)      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                            │
│  ┌─── Patient-Facing RF Coil ─────────────────────────┐   │
│  │  Flat spiral surface coil (30 mm dia, 5 turns)      │   │
│  │  T/R switch + matching network                      │   │
│  │  Optional Peltier cooler (→ 200 K for SNR boost)    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                            │
│  ┌─── Probe Electronics ──────────────────────────────┐   │
│  │  RF transmitter (~10 W peak), maser control MCU     │   │
│  │  ADC, pulse sequence controller                     │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────────┬─────────────────────────────────┘
                           │ USB-C cable
                    ┌──────▼──────┐
                    │   TABLET    │ → App: scan, reconstruct, AI assist
                    └──────┬──────┘
                           │ WiFi (optional)
                    ┌──────▼──────┐
                    │    CLOUD    │ → ML recon, AI diagnostics, remote review
                    └─────────────┘
```

---

## Target Specifications

### Probe-Level Targets

| Parameter | Target | Rationale |
|---|---|---|
| **Probe weight** | < 1.5 kg | One-hand operation |
| **Probe dimensions** | ~8 × 8 × 12 cm | Ultrasound-transducer size class |
| **Imaging depth** | 5–25 mm (diagnostic to 15 mm) | Subcutaneous tissue, tendons, bone interface |
| **Lateral FOV** | 3–8 cm | Region of clinical interest |
| **Resolution** | 1–3 mm in-plane, 1–5 mm depth | Hemorrhage, fracture, edema detection |
| **Acquisition time** | 15–120 s per view | Emergency triage pace |
| **Contrast** | T1, T2, proton density | Tissue typing (fluid vs. solid) |
| **Power** | < 100 W total | Battery-operable |
| **Connectivity** | USB-C to tablet | Standard clinical workflow |
| **Cryogenics** | None | Room-temperature maser advantage |

### Maser Module Specs (from Digital Twin)

| Parameter | Target | Source |
|---|---|---|
| **B₀ field** | 50 mT (NV lower branch ν₋ ≈ 1.47 GHz) | Default config |
| **Field uniformity** | < 100 ppm over active zone | Shimming spec |
| **Cavity Q (unloaded)** | ≥ 10,000 | Wang 2024 baseline |
| **Cavity Q (Q-boosted)** | ≥ 100,000 (target ~650,000) | Wang 2024 |
| **Cavity mode** | TE₀₁₁ at ~1.47 GHz | Matched to NV transition |
| **Mode volume** | ~0.5 cm³ | Typical for 12 mm bore |
| **Diamond** | CVD, 10 ppm NV⁻, 0.5 mm thick | Kersten 2026 params |
| **T₂\*** | ≥ 1 µs | Required for gain |
| **Cooperativity** | C > 1 (masing threshold) | Cavity QED |
| **Optical pump** | 532 nm, 2–4 W CW | Breeze 2018, ISC pathway |
| **Shim coils** | 8 independent gradient coils | Digital twin model |
| **Maser gain** | ≥ 20 dB (regenerative: 30–60 dB near threshold) | Primary SNR advantage |
| **Noise temperature** | < 10 K (quantum limit: 0.07 K) | Room-temp quantum amplifier |
| **Gain bandwidth** | ≥ 50 kHz | Matches NMR readout BW |

### Single-Sided Imaging Magnet Specs

| Parameter | Target | Rationale |
|---|---|---|
| **Magnet type** | Sweet-spot barrel (concentric NdFeB rings) | Quasi-homogeneous region for T2 sequences |
| **B₀ at sweet spot** | 50 mT | Matches proton Larmor ~2.13 MHz |
| **Sweet spot depth** | 15–25 mm from surface | Clinically useful tissue depth |
| **Sweet spot size** | ~10 mm dia × 5 mm deep | Uniform region for multi-echo |
| **Homogeneity in sweet spot** | < 500 ppm | Enables CPMG sequences |
| **Built-in gradient** | ~5–15 T/m | Depth encoding |
| **Outer diameter** | ~6–8 cm | Handheld form factor |
| **Magnet mass** | < 1.0 kg | Weight budget |
| **Material** | N52 NdFeB | Maximum remanence |

---

## Subsystems & Work Packages

### 1. NV Maser Module

The maser module is the core technology differentiator — a room-temperature quantum-limited amplifier.

#### 1a. Maser Halbach Permanent Magnet Array

**Goal**: Generate a uniform 50 mT static field across the diamond volume.

- **Design**: K=2 (dipolar) Halbach cylinder, NdFeB N52 segments
- **Geometry**: 8–16 segments, ~7 mm bore, ~15 mm outer radius
- **Isolation**: Mu-metal shield to reject stray field from imaging magnet
- **Simulations**: Multipole harmonics from digital twin (`halbach.py`)
- **CAD**: `cad/maser-halbach/`
- **Status**: 🔲 Not started

#### 1b. Microwave Cavity

**Goal**: TE₀₁₁ resonator at ~1.47 GHz with Q ≥ 10,000, optical access.

- **Material**: Oxygen-free copper (OFHC) or sapphire dielectric
- **Design**: Cylindrical, with optical port (532 nm) and microwave coupling port
- **Q-boost**: Optional electronic feedback loop (Wang 2024 design)
- **CAD**: `cad/cavity/`
- **Status**: 🔲 Not started

#### 1c. Optical Pump System

**Goal**: 532 nm CW laser delivering 2–4 W to the diamond face.

- **Laser**: Miniature DPSS 532 nm, fiber-coupled for probe integration
- **Thermal**: Heat sink for quantum defect (~60% of absorbed power)
- **Safety**: Fully enclosed in probe housing; interlock switches
- **BOM**: `bom/optical/`
- **Status**: 🔲 Not started

#### 1d. Active Shimming Electronics

**Goal**: 8-channel current driver for gradient shim coils on the maser Halbach.

- **MCU**: ESP32-S3 or STM32H7 (runs ONNX-exported CNN at < 1 ms)
- **DAC**: 16-bit, 8-channel (e.g., AD5668 or DAC8568)
- **ADC**: 16-bit, for field sensing (e.g., ADS1115 or AD7606)
- **PCB**: `pcb/shimming-controller/`
- **Firmware**: `firmware/shimming-controller/`
- **Status**: 🔲 Not started

### 2. Single-Sided Imaging Magnet

**Goal**: Generate a 50 mT sweet-spot field at 20 mm depth outside the probe surface.

- **Design**: Barrel-type concentric NdFeB ring array optimized for sweet spot
- **Built-in gradient**: ~5-15 T/m provides depth encoding
- **Optimization**: Digital twin `single_sided_magnet.py` module (to be built)
- **Delivery**: Custom magnet assembly — 3D-printed alignment jig + commercial N52 magnets
- **Key challenge**: Magnetic isolation from maser Halbach (distance + mu-metal)
- **CAD**: `cad/imaging-magnet/`
- **Status**: 🔲 Not started

### 3. Surface RF Coil & Receive Chain

**Goal**: Transmit RF excitation and receive NMR signals from tissue through probe surface.

- **Coil**: Flat spiral PCB coil, ~30 mm diameter, 5 turns
- **T/R switch**: PIN diode or MEMS switch (>60 dB isolation)
- **Up-conversion**: Passive mixer upconverts ~2 MHz NMR signal to 1.47 GHz for maser input
- **LO source**: DDS or PLL locked to maser carrier frequency
- **Optional Peltier**: TEC cooler to ~200 K for 1.35× SNR boost
- **PCB**: `pcb/rf-coil/`
- **Status**: 🔲 Not started

### 4. Lateral Gradient Coils

**Goal**: Planar gradient coils on the probe face for 2D in-plane spatial encoding.

- **Design**: 2-axis flat gradient coils (Golay-type or planar saddle)
- **Gradient strength**: ≤ 50 mT/m per axis
- **Driver**: 3-channel current amplifier (~5A, ~50W per channel)
- **Location**: Integrated into imaging magnet face or behind RF coil
- **PCB/wound**: `cad/gradient-coils/`
- **Status**: 🔲 Not started

### 5. Signal Chain & Readout

**Goal**: Digitize the maser-amplified NMR signal and stream to tablet.

- **Maser output**: ~1.47 GHz, post-amplification
- **Down-conversion**: Mixer to baseband (DC – 50 kHz)
- **ADC**: 16-bit, ≥ 200 kHz sampling
- **Interface**: USB-C data stream to tablet
- **Sequence controller**: FPGA or MCU with µs timing precision
- **Status**: 🔲 Not started

### 6. NV Diamond

**Goal**: Source a CVD diamond with appropriate NV⁻ concentration.

- **Spec**: Type IIa CVD, [NV⁻] ~ 10 ppm (~1.76×10¹⁷/cm³), [N_s] < 50 ppm
- **Dimensions**: ~3×3×0.5 mm (thickness along cavity axis)
- **Suppliers**: Element Six, Applied Diamond, Sumitomo, Delaware Diamond Knives
- **Characterization**: ODMR linewidth → T₂*, PL intensity → NV concentration
- **Status**: 🔲 Not started — requires sourcing quotes

### 7. Probe Housing & Integration

**Goal**: Package all subsystems into a handheld ergonomic enclosure.

- **Material**: 3D-printed (SLA/MJF) or injection-molded polycarbonate
- **Thermal management**: Heat sinks for laser and maser, forced air if needed
- **Ergonomics**: Pistol-grip or transducer-style handle
- **Cable**: USB-C (power + data), possibly with separate power connector for gradient amps
- **CAD**: `cad/probe-housing/`
- **Status**: 🔲 Not started

---

## Repository Structure

```
nv-maser-hardware/
├── README.md               This file
├── AGENTS.md               AI collaboration guide
├── docs/
│   ├── research/           Literature notes, supplier comparisons
│   └── plans/              Build plans, phase breakdown
├── pcb/                    KiCad projects
│   ├── shimming-controller/
│   ├── rf-coil/            Surface coil PCB
│   └── signal-chain/       Readout electronics
├── cad/                    FreeCAD / Fusion 360 / STEP files
│   ├── maser-halbach/      Maser module Halbach array
│   ├── imaging-magnet/     Single-sided sweet-spot magnet
│   ├── cavity/             Microwave cavity
│   ├── gradient-coils/     Planar lateral gradient coils
│   └── probe-housing/      Handheld enclosure
├── firmware/               MCU firmware (C/C++, PlatformIO)
│   └── shimming-controller/
├── bom/                    Bills of materials (CSV/spreadsheet)
│   ├── optical/
│   ├── electronics/
│   └── mechanical/
├── assembly/               Assembly instructions, photos
└── calibration/            Measured data fed back to digital twin
```

---

## Interface with Digital Twin

The two repos communicate through well-defined interfaces:

| Direction | What | Format |
|---|---|---|
| **Twin → Hardware** | Target specs (field, Q, thermal) | Documented in this README |
| **Twin → Hardware** | Trained controller weights | ONNX file (`checkpoints/model.onnx`) |
| **Twin → Hardware** | Coil geometry parameters | `config/default.yaml` → coil section |
| **Hardware → Twin** | Measured B₀ field maps | `.npz` arrays in `calibration/` |
| **Hardware → Twin** | Measured cavity Q, NV T₂* | Updated `config/` YAML overrides |
| **Hardware → Twin** | Calibration data | Feeds `nv-maser-twin` re-training |

When hardware measurements are available, the digital twin can be re-trained on real data instead of synthetic physics — closing the simulation-reality gap.

---

## Build Phases (Revised for Handheld Probe)

| Phase | Scope | Deliverable | Dependencies |
|---|---|---|---|
| **Phase 0: Research & Simulation** | Literature, sweet-spot magnet optimization in digital twin | Validated magnet design, SNR predictions | None |
| **Phase 1: Maser Module** | Build NV maser (Halbach + cavity + diamond + laser + shims) | Working maser oscillation at 1.47 GHz | Phase 0 |
| **Phase 2: Single-Sided Magnet** | Design barrel array, order NdFeB, 3D-print jig, assemble, field-map | Measured sweet-spot at target depth | Phase 0 |
| **Phase 3: Surface Coil & Receive** | PCB coil, T/R switch, up-conversion mixer, maser integration | NMR signal detected from water phantom | Phases 1, 2 |
| **Phase 4: First Depth Profile** | Combine all, acquire from layered phantom | 1D T2 depth profile on screen | Phase 3 |
| **Phase 5: Lateral Gradients** | Planar gradient coils, pulse sequencer | 2D encoded raw k-space data | Phase 4 |
| **Phase 6: Reconstruction** | NUFFT + compressed sensing on tablet app | First 2D image displayed | Phases 4, 5 |
| **Phase 7: Probe Integration** | 3D-printed housing, cable, thermal management | Handheld prototype | Phase 6 |
| **Phase 8: Phantom Validation** | Tissue-mimicking phantoms, SNR/resolution measurements | Validated performance metrics | Phase 7 |
| **Phase 9: Tissue Imaging** | Ex vivo tissue, then in vivo (forearm skin) | First human tissue image | Phase 8 |

---

## Getting Started

```bash
# Clone
git clone https://github.com/eric-rolph/nv-maser-hardware.git
cd nv-maser-hardware

# Initialize git
git init  # if starting fresh locally
```

### Prerequisites

- [KiCad 8+](https://www.kicad.org/) — PCB design
- [FreeCAD](https://www.freecadweb.org/) or Fusion 360 — mechanical CAD
- [PlatformIO](https://platformio.org/) — firmware build system
- [Python 3.10+](https://python.org/) — scripting, data analysis

---

## References

### Source Papers (shared with digital twin)
- **Breeze et al.** (2018), Nature 555, 493 — First room-temperature maser with NV diamond
- **Kollarics et al.** (2024), Science Advances, PMC11135399 — NV spin T₁ characterization
- **Wang et al.** (2024), Advanced Science, PMC11425272 — Electronic Q-boosting, noise temperature
- **Long et al.** (2025), Communications Engineering, PMC12241473 — Pulsed pump masing
- **Kersten et al.** (2026), Nature Physics, PMC12811124 — Spectral dynamics, dipolar refilling

### Hardware References
- **Jin et al.** (2015), Nat. Commun. 6, 8251 — NV maser concept and diamond cavity QED
- **Halbach** (1980), NIM 169 — Halbach permanent magnet arrays
- Various application notes for DAC/ADC selection (TI, Analog Devices)

### Single-Sided NMR/MRI References
- **Blümich et al.** (2008), Progress in NMR Spectroscopy 52, 197 — NMR-MOUSE review
- **Casanova et al.** (2011), Single-Sided NMR (Springer) — Comprehensive textbook
- **Marble et al.** (2007), J. Magn. Reson. 186, 100 — Optimized single-sided magnet for NMR
- **Cooley et al.** (2021), Sci. Adv. 7, eabf8467 — Portable single-sided MRI
- **Prado** (2003), J. Magn. Reson. 166, 228 — Sweet-spot magnet design

---

## License

MIT
