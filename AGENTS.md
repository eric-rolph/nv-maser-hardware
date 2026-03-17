# AGENTS.md — NV Maser Hardware Build

> Companion hardware repo for the [`nv-maser-twin`](https://github.com/eric-rolph/nv-maser-twin) digital twin.

---

## 📚 Essential References

| Document | Purpose | When to Consult |
|----------|---------|-----------------|
| **`README.md`** | Full spec table & subsystem breakdown | Start here |
| **nv-maser-twin `README.md`** | Digital twin, physics models, ML controller | Understanding simulation-derived specs |
| **nv-maser-twin `config/default.yaml`** | Canonical parameter values | Matching hardware to sim |
| **`docs/research/`** | Literature notes, supplier comparisons | Component selection |
| **`docs/plans/`** | Build phase plans | Before starting any fabrication |

---

## Project Context

This repo covers the **physical construction** of a room-temperature NV diamond maser with active magnetic shimming. The digital twin (separate repo) provides:
- Target B₀ field uniformity and Halbach design
- Microwave cavity specs (Q, mode, coupling)
- Trained shimming controller weights (ONNX)
- Thermal budget and optical pump parameters

**Our job here**: Turn those specs into PCBs, machined parts, firmware, and a working device.

---

## Repository Structure

| Directory | Contents | Toolchain |
|-----------|----------|-----------|
| `pcb/` | KiCad schematics & layouts | KiCad 8 |
| `cad/` | Mechanical designs (Halbach jigs, cavity, mounts) | FreeCAD / Fusion 360 |
| `firmware/` | MCU source code (shimming controller) | PlatformIO (C/C++) |
| `bom/` | Bills of materials per subsystem | CSV / spreadsheet |
| `assembly/` | Build instructions, wiring diagrams, photos | Markdown / images |
| `calibration/` | Measured field maps, Q measurements | `.npz`, `.csv` |
| `docs/research/` | Literature summaries, vendor comparisons | Markdown |
| `docs/plans/` | Build phase plans, decision logs | Markdown |

---

## Workflow

### ACE Workflow (same as twin repo)

1. **Research** (`/research`): Gather datasheets, literature, supplier quotes. Output to `docs/research/`.
2. **Plan** (`/plan`): Create detailed build plan. Output to `docs/plans/`.
3. **Build** (`/build`): Execute one step at a time, updating the plan as we go.
4. **Compact** (`/compact`): Summarize progress, clear context if needed.

### Hardware-Specific Phases

| Phase | Goal | Key Outputs |
|-------|------|-------------|
| **Phase 0** | Component research & selection | `docs/research/`, `bom/` |
| **Phase 1** | Halbach magnet array | `cad/halbach/`, assembly photos |
| **Phase 2** | Shimming electronics | `pcb/`, `firmware/`, bench test data |
| **Phase 3** | Microwave cavity | `cad/cavity/`, measured Q |
| **Phase 4** | Optical pump setup | `bom/optical/`, safety docs |
| **Phase 5** | Diamond integration | ODMR data in `calibration/` |
| **Phase 6** | First masing attempt | Signal chain data |
| **Phase 7** | Active shimming | Controller deployment |
| **Phase 8** | Calibration loop | Data → digital twin retraining |

---

## File Conventions

### PCB (KiCad)
- One KiCad project per board: `pcb/<board-name>/<board-name>.kicad_pro`
- Include schematic (`.kicad_sch`) and layout (`.kicad_pcb`)
- Export Gerbers to `pcb/<board-name>/gerbers/` before ordering
- Version schematic symbols and footprints in `pcb/libs/`

### CAD
- Prefer STEP (`.step`) for interchange, native format for working files
- Export STL for 3D printing to `cad/<part>/stl/`
- Include dimensioned drawings as PDF in `cad/<part>/drawings/`

### Firmware
- PlatformIO project structure: `firmware/<project>/platformio.ini`
- Target: ESP32-S3 or STM32H7 (TBD in Phase 0)
- ONNX Runtime Micro for ML inference

### BOM
- CSV format: `Part, Manufacturer, MPN, Qty, Source, Unit Price, Notes`
- One BOM file per subsystem: `bom/optical/bom.csv`, `bom/electronics/bom.csv`, etc.

### Calibration Data
- NumPy `.npz` for field maps (compatible with digital twin's data loaders)
- CSV for single measurements (Q, T₂*, linewidth)
- Always include measurement date and conditions in header/filename

---

## Key Design Constraints

| Constraint | Value | Why |
|-----------|-------|-----|
| B₀ = 50 mT | NV ground state ν₋ ≈ 1.47 GHz | Matches cavity design frequency |
| Diamond T_max < 400 K | T₂* degrades above ~400 K | Thermal budget for optical pump |
| Cavity optical access | ≥ 2 mm port | 532 nm pump beam must reach diamond |
| Shim coil current | < 500 mA per coil | Power dissipation & electronics limits |
| Controller latency | < 1 ms inference | Real-time shimming loop |
| Q₀ ≥ 10,000 | Sufficient cooperativity for masing | Cavity QED requirement |

---

## Safety

- **Laser**: Class 4 (532 nm, 2–4 W). Requires interlock, goggles, posted signage.
- **Magnets**: NdFeB N52 — strong enough to pinch fingers, attract tools. Keep away from electronics, credit cards, pacemakers.
- **Microwave**: Low power (~mW level inside cavity), but proper shielding needed.
- **Electrical**: Low voltage (5–24 V), but double-check coil current capacity.

---

## See Also

| Repo | Purpose |
|------|---------|
| [`nv-maser-twin`](https://github.com/eric-rolph/nv-maser-twin) | Digital twin: physics simulation, ML controller, validation |
| This repo | Physical hardware build |
