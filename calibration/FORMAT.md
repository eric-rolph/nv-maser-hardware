# Calibration Field Map Format

This document defines the on-disk format for magnetostatic field maps exchanged
between `nv-maser-hardware` (measurement) and `nv-maser-twin` (simulation / ML).

---

## File Format: NumPy `.npz` archive

Each measurement session produces one `.npz` file. Files are created by the
measurement rig and consumed by the digital twin for training / comparison.

### Naming Convention

```
halbach_phase1_YYYYMMDD.npz          # initial Halbach characterisation
halbach_phase1_YYYYMMDD_shimmed.npz  # post-shimming
measurement_run_NNN.npz              # sequential daily measurement
```

---

## Required Arrays

| Key | dtype | shape | unit | Description |
|-----|-------|-------|------|-------------|
| `b_z` | `float32` | `(H, W)` | T | Z-component (axial) field values on a 2-D grid. Row index → y, column index → x. |
| `x_mm` | `float32` | `(W,)` | mm | X-axis coordinates (columns), monotonically increasing. |
| `y_mm` | `float32` | `(H,)` | mm | Y-axis coordinates (rows), monotonically increasing. |

---

## Required Metadata (0-dimensional arrays)

| Key | dtype | Example | Description |
|-----|-------|---------|-------------|
| `b0_nominal_tesla` | `float64` | `0.05` | Expected nominal field strength (needed for ppm calculation). |
| `active_radius_mm` | `float64` | `3.0` | Radius of the NV active zone for uniformity reporting. |
| `source` | `U` (unicode str) | `"measurement"` | Must be `"measurement"` for hardware files; `"simulation"` for twin exports. |
| `timestamp` | `U` (unicode str) | `"2025-07-15T14:30:00Z"` | ISO 8601 UTC acquisition timestamp. Empty string if unknown. |
| `notes` | `U` (unicode str) | `"Hall probe scan, 100 µm step"` | Free-form provenance notes. Empty string if none. |

---

## Grid Requirements

- **Extent**: cover at least the full NV active zone diameter, i.e. ±`active_radius_mm` mm in both X and Y.
- **Resolution**: any, but 32×32 or 64×64 is recommended to match the simulation grid. The twin's `regrid()` function will interpolate if sizes differ.
- **Alignment**: grid must be centred on the Halbach bore axis (x=0, y=0).

---

## Writing a Compliant File (Python)

Use the twin's calibration library directly — it handles all type coercions:

```python
import numpy as np
from nv_maser.calibration import FieldMap, save_field_map

fm = FieldMap(
    b_z=b_z_array.astype(np.float32),      # shape (H, W), in Tesla
    x_mm=x_mm_array.astype(np.float32),    # shape (W,), in mm
    y_mm=y_mm_array.astype(np.float32),    # shape (H,), in mm
    b0_nominal_tesla=0.05,
    active_radius_mm=3.0,
    source="measurement",
    timestamp="2025-07-15T14:30:00Z",
    notes="Hall probe scan, 100 µm step, room temp 24.1 °C",
)

save_field_map("calibration/halbach_phase1_20250715.npz", fm)
```

### Manual write (without the twin installed)

If the twin library is not available on the measurement machine, write the `.npz`
directly — the twin's `load_field_map()` will accept either approach:

```python
import numpy as np

np.savez(
    "halbach_phase1_20250715.npz",
    b_z=b_z_array.astype(np.float32),
    x_mm=x_mm_array.astype(np.float32),
    y_mm=y_mm_array.astype(np.float32),
    b0_nominal_tesla=np.float64(0.05),
    active_radius_mm=np.float64(3.0),
    source=np.array("measurement", dtype="U"),
    timestamp=np.array("2025-07-15T14:30:00Z", dtype="U"),
    notes=np.array("Hall probe scan", dtype="U"),
)
```

---

## Loading in the Digital Twin

```python
from nv_maser.calibration import (
    load_field_map,
    uniformity_ppm,
    compare_maps,
    simulated_field_map,
)
from nv_maser.config import Config

measured = load_field_map("calibration/halbach_phase1_20250715.npz")

# Uniformity of measured field
print(f"Uniformity: {uniformity_ppm(measured):.0f} ppm")

# Compare against simulation
cfg = Config.from_yaml("config/default.yaml")
reference = simulated_field_map(cfg)
result = compare_maps(measured, reference)
print(f"RMS residual: {result.rms_residual_ppm:.1f} ppm ({result.rms_residual_tesla*1e6:.3f} µT)")
print(f"Correlation:  {result.correlation:.4f}")
```

---

## Typical Values (Phase 1 Target)

| Metric | Target | Notes |
|--------|--------|-------|
| Uniformity (bare Halbach) | < 1 000 ppm | before shimming |
| Uniformity (shimmed) | < 100 ppm | after 3-iteration shim |
| RMS residual vs. simulation | < 500 ppm | validates digital twin |
| Correlation with simulation | > 0.95 | good spatial agreement |

---

## See Also

- [nv-maser-twin calibration module](https://github.com/eric-rolph/nv-maser-twin/tree/main/src/nv_maser/calibration)
- `src/nv_maser/calibration/field_map.py` — full Python API
- `config/default.yaml` → `calibration:` section — path configuration
