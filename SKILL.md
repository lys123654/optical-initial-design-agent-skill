# Optical Initial Design Agent

## Purpose

Use this skill when a user wants to turn optical-system performance requirements into a reviewable initial lens structure by searching prescription libraries and patents, auditing candidate data, writing a Zemax OpticStudio file through ZOS-API, and producing an initial focus and review report.

This skill is for **starting-point generation and audit**, not for claiming a finished production optical design.

## Typical Requests

Use this skill for requests like:

- "Given these lens requirements, find an initial structure and build it in Zemax."
- "Search prescription libraries and patents for a compact telephoto seed."
- "Audit a patent lens prescription before entering it into Zemax."
- "Generate a ZMX starting point, quick focus it, and write a first review."
- "Create a repeatable workflow from requirements to Zemax initial structure."

## Workflow

### 1. Clarify Requirements

Collect or infer the following fields. If one or two are missing, make conservative assumptions and mark them in the report.

- Application and system type: telephoto, wide angle, telescope, relay, objective, etc.
- Effective focal length or magnification.
- F-number or NA.
- Image circle, sensor size, field angle, or object/image height.
- Wavelengths and weights.
- Total track length and back focal length constraints.
- Lens count range.
- Materials, plastic/glass limits, asphere limits, cemented-group preference.
- Cover glass, filters, window thickness, and detector distance.
- Must-have analysis outputs: layout, spot, MTF, PSF, CRA, distortion, relative illumination.

Write the clarified requirements to `specs.json` before building the Zemax file.

For microscope objectives and other modular systems, also clarify the boundary of the requested subsystem:

- Whether the requested focal length belongs to the objective itself, the tube lens, or the combined objective-plus-tube system.
- Whether the object is finite-conjugate, infinity-corrected, or a reverse-use reproduction of a patent.
- Whether the tube lens may be paraxial. A paraxial tube lens can be acceptable for evaluation, but do not replace a requested real objective prescription with a paraxial objective unless the user explicitly asks for a first-order scaffold.
- How field is specified. For an infinity-corrected microscope, a 20 mm image circle belongs to the tube-lens image plane; convert it to objective-side field angle or object height only after the tube-lens focal length is defined.
- How working distance is defined. Surgical microscope and long-working-distance objective patents often report the object-side air space; do not confuse it with back focal length in the printed prescription direction.

For remote-sensing or pushbroom hyperspectral systems, clarify the fore-optics boundary before searching:

- Whether the task is only the fore telescope or the full telescope-plus-spectrometer chain.
- Pixel pitch, GSD, orbit altitude, and whether the swath is one-pass optical coverage or a later scanning/mosaicking target.
- Whether the image plane is a detector, a slit, an intermediate image, or a relay into a spectrometer.
- Spectral range: visible, VNIR, SWIR, MWIR, LWIR, or mixed bands. This strongly affects reflective versus refractive architecture.
- Whether the user needs slit telecentricity, distortion, smile/keystone limits, relative illumination, stray-light constraints, or only a first Zemax starting structure.

### 2. Search Candidate Sources

Search in this order unless the user asks otherwise:

1. Existing prescription libraries and public optical benches.
2. Public lens-design examples, textbooks, or vendor examples.
3. Patents through Google Patents, Justia Patents, Espacenet, The Lens, WIPO, and national patent offices.
4. Broader web search for related exact example numbers, patent family members, PDFs, and table reproductions.

Do not stop at the first plausible match. Screen at least three candidates when the search space allows it.

For patent or library screening, rank candidates by application match first, then by first-order scale:

- For surgical microscopes, prefer sources that explicitly say surgical microscope, operating microscope, large objective, long working distance, apochromat, or Galilean zoom interface.
- If the target working distance is approximate, choose the closest real prescription in the correct application class instead of forcing an unrelated patent to the requested WD.
- Keep original-scale reproduction separate from scaled or optimized variants. Name files so it is obvious whether they are `original`, `scaled`, `reversed`, `paraxial_scaffold`, or `optimized`.
- Do not use an unrelated patent only because its focal length ratio is convenient. A weak application match should be rejected even if it can be scaled numerically.

For remote-sensing fore telescopes:

- Prefer sources that explicitly match the architecture class: TMA, Korsch, four-mirror anastigmat, RC/Cassegrain, Dyson/Offner, or pushbroom hyperspectral fore optics.
- For large swath and long EFL, prefer off-axis TMA/Korsch-family sources over on-axis RC unless the user only wants a rough comparison baseline.
- Public examples may provide enough geometry to build a Zemax file but not enough for performance. If freeform/polynomial coefficients are missing or not mapped, label the file as a geometry seed.
- A slower but complete published TMA prescription is often a better starting point than a faster but incomplete patent claim.

### 3. Audit Candidate Prescriptions

For each serious candidate, create a candidate audit note. Check:

- Complete radius, thickness, material, and aperture data.
- Whether dimensions are radius, diameter, semi-diameter, full image height, or half image height.
- Whether the prescription includes stops, coordinate breaks, mirrors, prisms, folded geometry, dummy surfaces, filters, and image plane.
- Whether asphere definitions match the target software convention.
- Whether glass names or model-glass Nd/Vd values can be reproduced.
- Whether the reported EFL, F-number, field, TTL, BFL, and image circle can be independently estimated.
- Whether source values are already scaled, normalized, or expressed as ratios.
- Whether the candidate is likely a patent example, a final product, or only a broad embodiment.
- Whether the printed surface order matches the intended use direction. Some patents list surfaces from image side to object side, or describe the "first" surface from the opposite side of the drawing.
- Whether radius-ratio, focal length, working distance, and group-position claims are internally consistent with the prescription table.
- Whether a plane surface, stop, field stop, cover glass, prism, or fold mirror is mentioned in text/figures but missing from the prescription table.
- Whether table values are OCR text or image-read values. If a table is image-only, mark that provenance and cross-check against a catalog or another patent family member.

Never present a patent table as fully reliable until it has passed a numerical and modeling audit.

Minimum numerical audit before Zemax entry:

- Recompute obvious ratios from the source, such as `R/f`, WD/f, F/# from pupil diameter, and image height from field angle.
- Run a paraxial ABCD or first-order estimate for the printed prescription where possible. Use it only as a plausibility check, not as proof of full optical quality.
- Compare reported EFL, BFL, WD, and total track against the matrix result in the same propagation direction. If they disagree, consider reversed use, sign convention, or a missing surface before modifying the prescription.
- Confirm glass names and basic Nd/Vd values against the local catalog when available. If using model glass, say so explicitly.

Remote-sensing first-order audit:

- Compute effective focal length from `EFL = orbit_altitude * pixel_pitch / GSD`, with consistent units.
- Compute required full field from swath and altitude: `full_fov = 2 * atan((swath / 2) / altitude)`.
- Compute focal-plane slit length from `slit_length = 2 * EFL * tan(full_fov / 2)`.
- Compute aperture from `EPD = EFL / F_number`. If a near-F/2 target implies a very large aperture, report it before building.
- If the implied slit length is unusually large, reduce the seed field for first inspection only after documenting the original requirement and the reduced seed field.

### 4. Convert To Zemax

Use ZOS-API through MATLAB, Python, or C# depending on the user's environment. Follow these rules:

- Use a two-surface cover glass or filter so Quick Focus adjusts the air gap after the glass, not the glass thickness.
- Set non-stop, non-image surface semi-diameters to Automatic solve when the initial aperture data is uncertain.
- Fix the aperture stop semi-diameter from the design F-number when appropriate.
- Set fields consistently from the user's definition: angle, object height, image height, or image circle.
- Use correct wavelength units and weights.
- For model glass, write Nd/Vd values explicitly and note that this is not catalog glass matching.
- For folded systems, either model coordinate breaks and mirrors explicitly or label the model as an unfolded approximation.
- Preserve all intentional deviations from the source prescription in the report.
- For mirror systems, use coordinate breaks and mirror materials explicitly. Record coordinate-break sign conventions and verify the layout visually.
- For off-axis TMA/Korsch systems, do not drop decenter/tilt data. If the source includes polynomial, Forbes, XY polynomial, or freeform coefficients, either map them to an appropriate Zemax surface type or state that only the base geometry was reproduced.

Field-setting rules:

- If the source provides a field table, reproduce it first.
- If the user gives image diameter and a tube lens is defined, set angle fields using `atan(image_semi_diameter / tube_lens_focal_length)`.
- If the user gives finite object size, set object-height fields and document magnification assumptions.
- If the source is an infinity-corrected objective without a tube lens, do not silently leave only the 0-degree field when the user requested an image circle. Either add a paraxial tube lens for evaluation or convert the field through the stated tube lens and label it as an evaluation field.
- Make the Zemax field header count match the number of field values. A file with three `YFLN` values but an `FTYP` declaration for one field will show only the on-axis field.

Infinity-objective and tube-lens rules:

- A real objective prescription should be modeled with real surfaces. A paraxial tube lens may be used to form the evaluation image plane when the user permits it.
- For a same-focal-length tube lens assumption, document it. Example: a 20 mm image diameter with a 250 mm tube lens gives a half-field angle `atan(10/250) = 2.290610 deg`.
- Verify collimation after the objective before the tube lens. Ray bundles that visibly converge or diverge after the objective do not satisfy an infinity-corrected objective setup.
- Do not tune the tube lens focal length to hide an objective focal-length mismatch. Report the mismatch first, then decide whether to optimize, reverse, or scale.

### 5. Focus And Analyze

After building the first Zemax file:

- Run Quick Focus or an equivalent image-plane focus step.
- Save both the build status and the focused file.
- Run at least one spot analysis and one MTF or first-order analysis if the license supports it.
- Save text, CSV, and image outputs when possible.
- Open the final Zemax file for visual inspection if the user asks.

Visual and first-order sanity checks:

- Inspect whether the ray direction and working plane match the intended object side. If a patent reproduction looks backwards, create a separate reverse-use file instead of silently flipping the original.
- Check that all requested fields appear in the layout, not just the 0-degree field.
- For infinity-corrected objectives, confirm that objective output is approximately parallel before the tube lens.
- If OpticStudio or ZOS-API is unavailable, say that only static text and paraxial checks were completed. Do not imply Quick Focus, spot, MTF, or layout verification.

### 6. Review And Report

Write a Markdown report with:

- Source candidate and provenance links.
- Requirement pass/fail table.
- Conversion assumptions.
- Known deviations from source prescription.
- 2D layout and ray-path sanity observations.
- EFL, F-number intent, image circle, TTL, BFL, lens count.
- Spot/MTF summary, with warning if the result is only a rough first pass.
- Explicit next steps: original-scale reproduction, scaled reproduction, optimization, glass matching, folded-geometry modeling, or merit-function setup.

When a source contains inconsistencies, state them plainly. For example, if a patent's printed `R1/f` fails its own stated range but the object-side surface satisfies it, report this as a surface-order or wording ambiguity rather than "fixing" the table silently.

## Lens-Type Notes

### Infinity-Corrected Microscope Objective

- Treat the objective as an afocal-output subsystem when the object is at its focal plane.
- Distinguish objective EFL, tube-lens focal length, and combined magnification.
- Image circle is normally evaluated after the tube lens, not at the objective exit by itself.
- Working distance is object plane to first objective surface, not image-side BFL unless the system is being used in reverse.
- A paraxial tube lens is acceptable for first-pass evaluation if declared; a paraxial objective is not acceptable when the user asked for a real patent/library objective.

### Surgical Microscope Large Objective

- Prioritize long-working-distance, apochromatic, large-objective patents and designs over generic microscope objectives.
- WD can be a class match rather than an exact forced value if the user says "close to" or "near".
- Many designs interface to a Galilean zoom or binocular system, so printed surface order may begin from the zoom side. Audit direction before interpreting WD and BFL.
- Field may be specified by downstream image diameter. Use the tube lens or relay focal length to convert this into angular fields for the objective model.

### Finite-Conjugate Microscope Objective

- Use object and image distances directly; do not assume collimated output.
- Magnification, NA, cover glass, and tube length are usually more important than standalone EFL.
- Verify cover-glass thickness and immersion medium before trusting performance.

### Photographic / Imaging Lens

- Image circle and sensor diagonal normally define field directly.
- F/# is tied to entrance pupil diameter; do not confuse with marginal ray cone at an arbitrary internal stop.
- BFL, flange distance, chief-ray angle, distortion, and relative illumination are often hard constraints.

### Telescope / Afocal Relay

- Report angular magnification, entrance/exit pupil, eye relief or relay pupil position.
- A finite image plane may be a test plane only; do not over-focus an afocal design unless an eyepiece or detector objective is included.

### Remote-Sensing Pushbroom / Hyperspectral Fore Telescope

- Treat the fore telescope as a subsystem feeding a slit or intermediate image unless the full spectrometer is explicitly included.
- Convert GSD/pixel/orbit into EFL before searching. This first-order result often dominates all later architecture choices.
- Convert swath into full FOV and focal-plane slit length before promising a single-instrument design.
- If one-pass swath implies an impractically long slit, create a reduced-field seed only as a documented first inspection model; keep the original swath in `specs.json`.
- RC/Cassegrain can be useful as a quick reflective baseline, but it should not be treated as the final architecture for wide-field high-spectral fore optics.
- Off-axis TMA or Korsch-family designs are usually stronger starting points for wide field, unobscured pupil, and reflective broadband performance.
- A public TMA prescription with F/10-F/12 can be a useful parent even if the user wants a faster F-number; label the F-number gap and plan a later redesign/optimization.
- Add slit telecentricity, distortion, smile/keystone, relative illumination, and stray-light checks as soon as the geometric seed is stable.

### Folded, Prism, or Mirror Systems

- Preserve coordinate breaks, mirror signs, prism materials, and unfolded/folded interpretation separately.
- If only an unfolded approximation is created, label it clearly and do not claim mechanical packaging is reproduced.

### Patent Reproduction

- Create the original-scale file first, even if the target spec requires scaling later.
- Keep candidate audit, original ZMX, scaled ZMX, and optimized ZMX as separate artifacts.
- Record every intentional deviation: model glass, guessed aperture, inferred field, missing stop, omitted plane, reversed surface order, or added paraxial tube lens.
- Never make the patent "fit" by changing radii, glass order, field, or working distance without creating a new derivative file and documenting the change.

## Required Output Structure

Create a project folder like:

```text
project_name/
  specs.json
  candidate_search_report.md
  candidate_audits/
    candidate_01.md
    candidate_02.md
  zemax/
    project_name_focused.zmx
    project_name_focused.ZDA
  analysis_outputs/
    spot.txt
    fft_mtf.txt
    fft_mtf.csv
    fft_mtf.png
  initial_review.md
  run_project_name.m
```

If only a skill package is being created, use the templates in `templates/`.

## Human Review Gates

Ask for user review before:

- Selecting a candidate when several plausible candidates exist.
- Trusting a patent with folded geometry, missing surfaces, or ambiguous asphere definitions.
- Deleting intermediate files.
- Claiming any requirement is strictly satisfied if it was only inferred or approximately estimated.

## Failure Modes To Watch

- Treating diameter as semi-diameter.
- Treating full image height as half image height.
- Adding stop or field-stop rows as extra distances instead of splitting adjacent air gaps.
- Letting Quick Focus change the cover-glass thickness.
- Forgetting automatic semi-diameter solves, causing misleading 2D layouts.
- Opening a source design to a faster F-number and assuming it remains valid.
- Scaling asphere coefficients with the wrong powers.
- Ignoring coordinate breaks or folded optical paths in patent examples.
- Reporting "Zemax opens" as proof that the structure is correct.
- Treating an on-axis RC/Cassegrain baseline as a final wide-swath hyperspectral fore telescope.
- Reducing a large swath for first inspection without documenting the original swath and the implied slit length.
- Importing an off-axis TMA source while dropping decenter/tilt or freeform coefficients without labeling it as a partial reproduction.
- Reporting mirror-system spot/MTF numbers before visually checking coordinate-break signs and ray path.
- Leaving a microscope objective at only 0-degree field when the user specified an image circle through a tube lens.
- Treating a paraxial tube lens as permission to use a paraxial objective.
- Treating patent working distance as BFL without checking propagation direction.
- Trusting patent prose when a simple ratio check contradicts the prescription table.
- Combining original reproduction, reverse-use interpretation, scaling, and optimization into one file without labels.

## Relationship To Other Tools

General Zemax guidance skills can help with optimization, analysis selection, and tolerancing. They do not replace the source-search, patent-audit, and prescription-to-Zemax workflow in this skill.
