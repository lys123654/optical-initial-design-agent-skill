# Remote-Sensing Hyperspectral Fore-Telescope Notes

Use this note when the requested system is a satellite, aircraft, or UAV pushbroom hyperspectral imager and the immediate task is to create a fore-telescope starting structure.

## First-Order Checks

Before searching patents or writing Zemax files, compute:

```text
EFL = orbit_altitude * pixel_pitch / GSD
full_fov = 2 * atan((swath / 2) / orbit_altitude)
slit_length = 2 * EFL * tan(full_fov / 2)
EPD = EFL / F_number
```

Keep units consistent. Report these values in `specs.json` and the candidate report.

## Architecture Guidance

- Use RC/Cassegrain only as a quick reflective baseline or software workflow check.
- Prefer off-axis TMA, Korsch, or other unobscured multi-mirror anastigmat layouts for broad spectral range and wide field.
- For VNIR/SWIR, all-reflective designs avoid chromatic correction pressure.
- A complete slower public TMA prescription is often more useful than an incomplete faster patent.
- If a source includes freeform or polynomial surfaces, do not silently omit those coefficients. Either map them to Zemax or call the file a geometry seed.

## Swath And Slit

Pushbroom systems can make the swath requirement look deceptively simple. A large swath at long EFL maps into a long focal-plane slit. If the slit length is very large:

- Keep the original swath in `specs.json`.
- Build a reduced-field seed for first inspection if needed.
- Say clearly that the reduced seed is not the final swath architecture.
- Consider scanning, multiple modules, relay demagnification, or a different optical architecture.

## Zemax Modeling Rules

- Use coordinate breaks for off-axis mirrors.
- Preserve decenter and tilt from the source prescription.
- Use mirror materials explicitly.
- Verify 2D and 3D layouts before trusting spot or MTF numbers.
- Keep original-scale, scaled, and optimized files separate.
- For scaled mirror prescriptions, radii, thicknesses, decenter, and apertures scale linearly; conic constants do not.

## Minimum Report Content

The first review should include:

- Derived EFL from GSD.
- Requested full FOV and slit length.
- Seed full FOV and slit length if reduced.
- EPD and F-number gap.
- Architecture rationale.
- Source prescription URL.
- What was reproduced exactly and what was omitted.
- Spot/MTF only as a first sanity check.

