# NV Maser "Tricorder" — Hardware Build

Physical implementation of a room-temperature **Nitrogen-Vacancy (NV) center diamond maser** with active magnetic shimming.

> **Digital twin companion**: Simulation, ML controller, and physics validation live in → [`nv-maser-twin`](https://github.com/ericrobinson-dev/nv-maser-twin).  
> That repo defines the target specifications; this repo implements them in hardware.

---

## System Overview

A functioning NV diamond maser requires five integrated subsystems:

```
┌──────────────────────────────────────────────────────────────────┐
│                        NV DIAMOND MASER                         │
│                                                                  │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │   HALBACH    │  │  MICROWAVE   │  │    OPTICAL PUMP      │   │
│  │   MAGNET     │→ │  CAVITY      │← │    (532 nm laser)    │   │
│  │   (50 mT)   │  │  (TE₀₁₁)    │  │                      │   │
│  └──────┬──────┘  └──────┬───────┘  └──────────────────────┘   │
│         │                │                                       │
│  ┌──────▼──────┐  ┌──────▼───────┐  ┌──────────────────────┐   │
│  │   ACTIVE     │  │  READOUT     │  │    CONTROL           │   │
│  │   SHIMMING   │  │  & SIGNAL    │  │    ELECTRONICS       │   │
│  │   (8 coils)  │  │  CHAIN       │  │    (MCU + DAC/ADC)   │   │
│  └─────────────┘  └──────────────┘  └──────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Target Specifications (from Digital Twin)

These specs are derived from the simulation and published literature:

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
| **Pump laser** | DPSS or diode, fiber-coupled option | |
| **Shim coils** | 8 independent gradient coils | Digital twin model |
| **Controller** | MCU with trained CNN (ONNX-exported) | nv-maser-twin |
| **DAC resolution** | ≥ 16-bit for current control | Signal chain SNR budget |
| **ADC resolution** | ≥ 16-bit for field sensing | Signal chain SNR budget |
| **Thermal budget** | Diamond stays < 400 K under pump | Thermal model |

---

## Subsystems & Work Packages

### 1. Halbach Permanent Magnet Array

**Goal**: Generate a uniform 50 mT static field across the diamond volume.

- **Design**: K=2 (dipolar) Halbach cylinder, NdFeB N52 segments
- **Geometry**: 8–16 segments, ~7 mm bore, ~15 mm outer radius
- **Simulations**: Multipole harmonics from digital twin (`halbach.py`) predict the field distribution
- **Tolerance**: Magnetization variation < 1% per segment
- **CAD**: `cad/halbach/`
- **Status**: 🔲 Not started

### 2. Microwave Cavity

**Goal**: TE₀₁₁ resonator at ~1.47 GHz with Q ≥ 10,000, optical access.

- **Material**: Oxygen-free copper (OFHC) or sapphire dielectric
- **Design**: Cylindrical, with optical port (532 nm access) and microwave coupling port
- **Coupling**: Adjustable loop/probe for critical coupling (β ≈ 0.5)
- **Q-boost**: Optional electronic feedback loop (Wang 2024 design)
- **Simulations**: COMSOL or CST for eigenmode analysis (cross-validate with `cavity.py`)
- **CAD**: `cad/cavity/`
- **Status**: 🔲 Not started

### 3. Optical Pump System

**Goal**: 532 nm CW laser delivering 2–4 W to the diamond face.

- **Laser**: DPSS 532 nm, TEM₀₀, beam waist ~1.5 mm
- **Optics**: Focusing lens, optional dichroic mirror for fluorescence monitoring
- **Thermal**: Heat sink for quantum defect (~60% of absorbed power)
- **Safety**: Laser class 4, interlocks required
- **BOM**: `bom/optical/`
- **Status**: 🔲 Not started

### 4. Active Shimming Electronics

**Goal**: 8-channel current driver for gradient shim coils, controlled by trained ML model.

- **MCU**: ESP32-S3 or STM32H7 (runs ONNX-exported CNN at < 1 ms)
- **DAC**: 16-bit, 8-channel (e.g., AD5668 or DAC8568)
- **ADC**: 16-bit, for field sensing (e.g., ADS1115 or AD7606)
- **Current driver**: Low-noise op-amp + MOSFET per coil channel
- **Interface**: USB/UART to host PC; optional WiFi for remote monitoring
- **PCB**: `pcb/shimming-controller/`
- **Firmware**: `firmware/shimming-controller/`
- **Status**: 🔲 Not started

### 5. Signal Chain & Readout

**Goal**: Detect and amplify the ~1.47 GHz maser emission.

- **LNA**: Low-noise amplifier (noise figure < 1 dB)
- **Mixer**: Downconvert to IF or baseband
- **ADC**: High-speed digitizer for power measurement
- **SNR budget**: Validated by digital twin `signal_chain.py`
- **Status**: 🔲 Not started

### 6. NV Diamond

**Goal**: Source a CVD diamond with appropriate NV⁻ concentration.

- **Spec**: Type IIa CVD, [NV⁻] ~ 10 ppm (~1.76×10¹⁷/cm³), [N_s] < 50 ppm
- **Dimensions**: ~3×3×0.5 mm (thickness along cavity axis)
- **Suppliers**: Element Six, Applied Diamond, Sumitomo, Delaware Diamond Knives
- **Characterization**: ODMR linewidth → T₂*, PL intensity → NV concentration
- **Status**: 🔲 Not started — requires sourcing quotes

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
│   └── shimming-controller/
├── cad/                    FreeCAD / Fusion 360 / STEP files
│   ├── halbach/
│   └── cavity/
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

## Build Phases (Proposed)

| Phase | Scope | Dependencies |
|---|---|---|
| **Phase 0: Research** | Literature deep-dive, supplier quotes, component selection | None |
| **Phase 1: Halbach Array** | Design, order magnets, 3D-print jig, assemble, measure B₀ | Phase 0 |
| **Phase 2: Shimming Electronics** | PCB design, fabricate, firmware, bench-test coils | Phase 0 |
| **Phase 3: Microwave Cavity** | Design, machine/print, characterize Q | Phase 1 (need field) |
| **Phase 4: Optical Pump** | Laser sourcing, optics bench, thermal management | Phase 0 |
| **Phase 5: Diamond Integration** | Mount diamond in cavity, measure ODMR, verify T₂* | Phases 1-4 |
| **Phase 6: First Masing** | Assemble everything, attempt maser oscillation | Phase 5 |
| **Phase 7: Active Shimming** | Deploy ONNX controller, closed-loop field correction | Phase 6 |
| **Phase 8: Calibration Loop** | Feed measured data back to digital twin, re-train | Phase 7 |

---

## Getting Started

```bash
# Clone
git clone https://github.com/ericrobinson-dev/nv-maser-hardware.git
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

---

## License

MIT
