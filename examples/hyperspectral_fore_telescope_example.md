# Example: Pushbroom Hyperspectral Fore Telescope

This example is intentionally generic. Replace all placeholder values with the actual project values before running.

## Requirement Sketch

```text
Application: spaceborne or airborne pushbroom hyperspectral imager
Subsystem: fore telescope only
Spectral range: visible, VNIR, SWIR, or mixed bands
Pixel pitch: project-defined
GSD: project-defined
Altitude: project-defined
Swath: project-defined one-pass or scanned coverage
F-number: as fast as practical
Architecture preference: reflective allowed
```

## Required First-Order Conversion

```text
EFL = altitude * pixel_pitch / GSD
full_fov = 2 * atan((swath / 2) / altitude)
slit_length = 2 * EFL * tan(full_fov / 2)
EPD = EFL / F_number
```

## Candidate Strategy

1. Search for complete off-axis TMA/Korsch prescriptions first.
2. Use RC/Cassegrain only as a deprecated comparison baseline if needed.
3. If a public TMA is slower than desired but complete, keep it as a parent seed and document the F-number gap.
4. If a freeform source cannot be fully mapped to Zemax, label the file as a geometry seed.

## Output

```text
hyperspectral_fore_telescope/
  specs.json
  candidate_search_report.md
  candidate_audits/
  zemax/
  analysis_outputs/
  tma_initial_review.md
  run_hyperspectral_tma_seed.m
```

