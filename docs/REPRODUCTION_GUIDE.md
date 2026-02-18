# IsoCompiler Reproduction Guide

This guide describes how an independent reviewer can verify the claims made in the IsoCompiler white paper. It covers what can be verified from the public repository alone, and what would require access to the proprietary codebase.

---

## 1. What This Repository Contains

| Directory | Contents |
|-----------|----------|
| `README.md` | White paper with all technical claims |
| `CLAIMS_SUMMARY.md` | 72 patent claims across 10 frequency bands |
| `HONEST_DISCLOSURES.md` | Complete disclosure of limitations |
| `LICENSE` | CC BY-NC-ND 4.0 |
| `evidence/key_results.json` | Machine-readable summary of all validated results |
| `verification/verify_claims.py` | Standalone verification script |
| `verification/reference_data/canonical_values.json` | Canonical reference values |
| `verification/requirements.txt` | Python dependencies for verification |
| `docs/SOLVER_OVERVIEW.md` | Non-proprietary solver physics description |
| `docs/REPRODUCTION_GUIDE.md` | This file |

---

## 2. Running the Verification Script

### Prerequisites

- Python 3.10 or later
- No external packages required (uses standard library only)

### Basic Verification

```bash
cd verification/
python verify_claims.py
```

This runs all verification checks and produces human-readable output showing pass/fail status for each check.

### JSON Output

```bash
python verify_claims.py --json
```

Produces machine-readable JSON for CI integration or automated analysis.

### Custom Canonical Values

```bash
python verify_claims.py --canonical /path/to/custom_values.json
```

---

## 3. What the Verification Script Checks

### Check 1: FDTD Convergence

For each of the 10 frequency bands, verifies:

- Grid spacing satisfies >= 20 cells per wavelength in the substrate material (eps_r = 4.4).
- The Courant-Friedrichs-Lewy (CFL) stability condition is satisfiable at the computed grid spacing.

**Physics basis:** The FDTD method requires sufficient spatial sampling to resolve electromagnetic waves (Taflove & Hagness, 2005). The standard minimum is 10-20 cells per wavelength; IsoCompiler uses 20 for conservative accuracy.

### Check 2: S21 Extraction Validity

For each band, verifies:

- 2D isolation improvement is within the plausible range (10-80 dB).
- 3D isolation improvement is within the plausible range.
- 2D improvement >= 3D improvement (2D overpredicts because it lacks out-of-plane leakage paths).
- Hanning window function is specified (not rectangular).

**Physics basis:** Isolation improvements beyond 80 dB are implausible for substrate-coupled structures at these frequencies. The 2D-to-3D gap (16-24 dB) is expected and well-documented.

### Check 3: Binarization

For each band, verifies:

- Optimized density field achieves >= 99% binary output (all pixels within 1% of 0.0 or 1.0).

**Physics basis:** Intermediate density values have no physical meaning and cannot be fabricated. The SIMP + Heaviside projection scheme is designed to produce 100% binary output.

### Check 4: Frequency Band Completeness

Verifies:

- All 10 frequency bands are present in the reference data.
- Center frequencies match the specification.
- All required data fields are present for each band.

### Check 5: Canonical Values Consistency

Cross-checks for internal consistency:

- Patent claims = (10 bands x 7 claims/band) + 2 cross-band = 72.
- Isolation improvement ranges match per-band data.
- Synthesis family count matches the listed families.
- DRC violations = 0.

### Check 6: Physics Plausibility

Checks fundamental physics relationships:

- Courant limits are correct (1/sqrt(2) for 2D, 1/sqrt(3) for 3D).
- PML reflection specification (-60 dB) is achievable with 20-cell polynomial grading.
- Sievenpiper mushroom EBG bandgap frequency falls in a plausible range (1-100 GHz for typical parameters).
- Parallel-plate waveguide TE1 cutoff frequency is above all 10 bands (confirming TEM dominance).

---

## 4. Independent Reproduction of Key Results

### What Can Be Reproduced Without the Proprietary Codebase

1. **Physics formulas.** All governing equations, stability conditions, and analytical models described in `docs/SOLVER_OVERVIEW.md` can be verified independently from standard textbooks.

2. **Sievenpiper EBG bandgap.** The mushroom EBG bandgap formula (f = 1/(2*pi*sqrt(LC))) can be computed for the stated parameters and compared to the claimed frequency ranges.

3. **Parallel-plate waveguide attenuation.** For a 500 um corridor in FR4 (eps_r=4.4, tan_delta=0.02), the TEM mode attenuation can be computed analytically:
   ```
   alpha = pi * f * tan_delta * sqrt(eps_r) / c_0
   S21 = -20*log10(exp(-alpha*d))  [dB]
   ```

4. **CFL stability.** The Courant stability condition can be verified for any grid spacing and material combination.

5. **Grid resolution.** For any frequency and substrate permittivity, the required grid spacing for 20 cells/wavelength can be computed.

### What Requires the Proprietary Codebase

1. **Actual FDTD simulation results.** Running the solver and comparing S21 outputs requires the IsoCompiler codebase.

2. **Adjoint-optimized structures.** Reproducing the specific optimized isolation patterns requires the inverse design engine.

3. **GDSII output verification.** Verifying that GDSII files pass DRC requires the synthesis and export pipeline.

4. **132-test suite.** Running the full test suite requires the complete codebase.

### What Requires Fabrication

1. **Measurement validation.** Confirming simulation predictions against VNA measurements requires fabricating test structures and measuring them.

2. **Foundry DRC.** Confirming compatibility with production foundry DRC decks requires submitting GDSII to a foundry or running a licensed DRC tool.

---

## 5. Reproducing Analytical Baselines

The following analytical results can be computed independently to verify the solver's baseline predictions.

### Parallel-Plate Waveguide S21

For a TEM mode in a parallel-plate waveguide with lossy dielectric fill:

```python
import numpy as np

c0 = 299792458.0       # m/s
eps_r = 4.4            # FR4
tan_delta = 0.02
d = 500e-6             # 500 um corridor

for f_ghz in [1, 3, 5, 10, 12, 20, 28, 39, 56, 77]:
    f = f_ghz * 1e9
    alpha = np.pi * f * tan_delta * np.sqrt(eps_r) / c0
    s21_db = -20 * np.log10(np.exp(alpha * d)) if alpha * d > 0 else 0.0
    print(f"Band {f_ghz:5.1f} GHz: S21 = {-s21_db:.2f} dB (baseline, no isolation)")
```

### Sievenpiper EBG Bandgap

```python
import numpy as np

mu0 = 4e-7 * np.pi
eps0 = 8.854e-12

# Typical mushroom EBG parameters
h = 200e-6    # via height (m)
w = 500e-6    # patch width (m)
eps_r = 4.4

L = mu0 * h
C = eps0 * eps_r * w**2 / h
f_bandgap = 1 / (2 * np.pi * np.sqrt(L * C))
print(f"Bandgap center: {f_bandgap/1e9:.2f} GHz")
```

### CFL Stability Check

```python
import numpy as np

c0 = 299792458.0
eps_r = 4.4

for f_ghz in [1, 3, 5, 10, 12, 20, 28, 39, 56, 77]:
    f = f_ghz * 1e9
    lam_sub = c0 / (f * np.sqrt(eps_r))
    dx = lam_sub / 20  # 20 cells per wavelength
    dt_max_2d = dx / (c0 * np.sqrt(2))
    dt_max_3d = dx / (c0 * np.sqrt(3))
    print(f"{f_ghz:5.1f} GHz: dx={dx*1e6:.1f} um, "
          f"dt_max_2d={dt_max_2d:.2e} s, dt_max_3d={dt_max_3d:.2e} s")
```

---

## 6. Verifying the Evidence Files

### evidence/key_results.json

This file contains machine-readable results for all 10 bands. Cross-check against:

1. The per-band table in README.md (values should match exactly).
2. The canonical_values.json reference data (should be consistent within tolerances).
3. The physics plausibility checks (all improvements in 10-80 dB range).

### verification/reference_data/canonical_values.json

This file defines the reference values that the verification script checks against. It can be audited for:

1. Internal consistency (patent claim arithmetic, isolation range matching per-band data).
2. Physical plausibility (Courant limits, PML specifications).
3. Completeness (all 10 bands present, all required fields defined).

---

## 7. Known Gaps in Reproducibility

| Gap | Impact | Mitigation |
|-----|--------|-----------|
| No solver source code | Cannot reproduce FDTD results | Physics described in SOLVER_OVERVIEW.md; standard algorithms |
| No fabrication data | Cannot confirm sim-to-measurement accuracy | Honest Disclosures document the gap; fabrication packages ready |
| No GDSII files | Cannot independently verify DRC | KLayout DRC results documented; process described |
| Single-developer codebase | Limited independent review | 132 automated tests; CI pipeline |

---

## 8. Recommended Review Procedure

For a technical reviewer evaluating IsoCompiler claims:

1. **Run verify_claims.py** and confirm all checks pass.
2. **Read HONEST_DISCLOSURES.md** to understand stated limitations.
3. **Compute analytical baselines** (Section 5 above) and compare to claimed values.
4. **Review SOLVER_OVERVIEW.md** for physics correctness.
5. **Check key_results.json** for internal consistency across all 10 bands.
6. **Assess the 2D-to-3D gap** (16-24 dB): is this consistent with your understanding of 2D vs. 3D FDTD accuracy for substrate isolation problems?

---

*This guide enables independent verification of IsoCompiler claims to the extent possible without access to the proprietary codebase. For full reproduction, access to the IsoCompiler platform is required.*
