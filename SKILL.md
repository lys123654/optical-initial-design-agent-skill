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

### 2a. Ratio-Based Screening (Tier 1 Quick Filter)

Before extracting full prescription data, screen candidates by dimensionless ratios computed from claimed or estimated values. This is fast, needs no API calls, and eliminates clearly incompatible candidates early.

Compute these ratios for each candidate:

| Ratio | Formula | Meaning | Typical tight target |
|---|---|---|---|
| TTL/f | TotalTrackLength / EFL | Compactness | <0.75 (telephoto), <1.0 (standard) |
| BFL/f | BackFocalLength / EFL | Detector packaging | >0.08 (typical camera) |
| WD/f | WorkingDistance / EFL | Microscope working distance | 0.80–1.05 (objective) |
| IH/f | ImageSemiHeight / EFL | Field of view proxy | = tan(half_FOV) |
| EP/f | EntrancePupil / EFL | F-number proxy | = 1/(2×F#) |
| Σt/f | SumGlassThickness / EFL | Element-count proxy | Application-specific |

Candidates that pass ratio screening proceed to Tier 2 (extraction and ABCD check). Candidates that fail ALL relaxed ratio bounds are rejected without full extraction. Record ratios in the candidate audit for traceability.

### 2b. Search Query Templates

Generate search queries systematically from specs.json. Use multiple query templates and multiple search engines in parallel when possible.

**Patent search templates (English):**
- `"focal length [EFL]mm F/[F#]" lens patent embodiment`
- `"compact telephoto [TTL/f target] TTL/f patent" lens optical system`
- `"[application] lens [EFL]mm patent radius thickness"`
- `"optical system" "f=[EFL]" "Fno=[F#]" patent numerical example`

**Patent search templates (Chinese):**
- `[EFL]mm [F#] 镜头 光学系统 专利 实施例`
- `短总长 长焦镜头 [EFL]mm 光学专利`
- `光学系统 曲率半径 厚度 阿贝数 [EFL]mm`

**Engine fallback order:**
1. Google Patents (English keywords)
2. Espacenet (advanced search, CPC/IPC classes)
3. The Lens (fielded search, patent families)
4. Google Patents (Chinese keywords for CNIPA patents)
5. Broader web search for prescription reproductions

**Integration with deep-research:** When available, invoke a deep-research skill to fan out across patent databases with the candidate criteria. Pass it a structured brief: `{target_spec, ratio_bounds, application_class, rejection_criteria}`. Do not rely on general web search alone.

### 2c. Multi-Tier Screening

Structure candidate evaluation in three explicit tiers:

- **Tier 1 (Ratio Screen):** Compute dimensionless ratios from claimed/reported values. Fast, no extraction needed. Gate: keep candidates passing ≥60% of applicable ratios.
- **Tier 2 (ABCD + Numerical Audit):** Run paraxial ABCD or first-order estimate. Check reported vs. computed EFL, BFL, TTL for consistency (±15%). Gate: keep candidates where computed values are internally consistent.
- **Tier 3 (Full Entry):** Complete surface-by-surface prescription extraction, glass substitution check (see 3a), Zemax build, Quick Focus, and full analysis suite.

Only escalate to the next tier when a candidate passes the current tier.

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

### 3a. Glass Substitution Check

Before building the Zemax file, replace model glass (Nd/Vd only) with real catalog glass from the user's glass library. Run the AGF glass matcher (`optical_glass_tools`) against the local catalogs (CDGM, HOYA, PLASTIC) for each material surface in the candidate prescription.

**Matching tolerance levels:**
- **Exact match:** Nd difference <0.001 AND Vd difference <0.1 → use catalog glass directly. Mark as `exact`.
- **Close match:** distance <0.05 (weighted) → use catalog glass, note the substitution in the intentional-deviations table. Mark as `close`.
- **Approximate match:** distance 0.05–0.20 → present top-3 candidates to user for review before committing. Mark as `approximate`.
- **No match:** distance >0.20 → keep model glass, flag as high-risk. Note that the prescription may depend on unobtainable materials.

**Substitution rules:**
1. Prefer CDGM over HOYA over PLASTIC unless the user specifies otherwise. CDGM is the primary catalog for most Chinese optical design work.
2. Within the same catalog, prefer `is_preferred` (availability=1) glasses over standard ones.
3. When multiple glasses have similar distance, prefer lower relative cost.
4. Document every substitution in the intentional-deviations table with: source Nd/Vd, matched glass name, catalog, distance, and tolerance level.
5. If the candidate prescription explicitly names a glass (e.g., "N-BK7", "H-FK61"), verify the name exists in the catalog and use the catalog Nd/Vd values, not the patent's printed values.

**Tool invocation:**
```bash
python -m optical_glass_tools match --library glass_library.json --nd 1.51680 --vd 64.2 --top 3
```

For bulk matching of an entire prescription:
```python
from optical_glass_tools.glass_library import GlassLibrary
lib = GlassLibrary.load("glass_library.json")
for nd, vd, surface_label in prescription:
    matches = lib.match(nd=nd, vd=vd, top_n=3)
    # Report best match, tolerance level, and alternatives
```

Generate a `glass_substitution_report.md` with one row per surface showing: surface label, source Nd/Vd, matched glass, catalog, Nd/Vd diff, distance, tolerance level, and alternatives.

### 4. Convert To Zemax

Use ZOS-API through MATLAB, Python, or MCP depending on the user's environment. **Before editing ZMX files or calling ZOS-API, consult `ZEMAX_OPS.md` for file format details, GLAS line conventions, and common pitfalls.**

Follow these rules:

- Use a two-surface cover glass or filter so Quick Focus adjusts the air gap after the glass, not the glass thickness.
- Set non-stop, non-image surface semi-diameters to Automatic solve when the initial aperture data is uncertain.
- Fix the aperture stop semi-diameter from the design F-number when appropriate.
- Set fields consistently from the user's definition: angle, object height, image height, or image circle.
- Use correct wavelength units and weights.
- For model glass, write Nd/Vd values explicitly and note that this is not catalog glass matching.
- For folded systems, either model coordinate breaks and mirrors explicitly or label the model as an unfolded approximation.
- Preserve all intentional deviations from the source prescription in the report.

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
- **Constraint relaxation analysis:** When no candidate passes all strict requirements, apply the `constraint_relaxation_protocol.md` protocol. Produce a weighted score for each candidate: `score = Σ(w_i × pass_i)` where `w_i` = priority weight (hard=1000, high=100, medium=10, low=1). Present a compromise ranking table showing which candidate is the "least bad" option and what was relaxed.
- Conversion assumptions.
- Known deviations from source prescription.
- **Glass substitution table:** Show every material surface with source Nd/Vd, matched catalog glass, distance, and tolerance level.
- 2D layout and ray-path sanity observations.
- EFL, F-number intent, image circle, TTL, BFL, lens count.
- Spot/MTF summary, with warning if the result is only a rough first pass.
- Explicit next steps: original-scale reproduction, scaled reproduction, optimization, glass matching refinement, folded-geometry modeling, or merit-function setup.

**Default constraint relaxation order** (user may override):
1. Relax F/# first (e.g., F/2.8 → F/3.5 → F/4.0)
2. Relax packaging constraints: TTL → BFL
3. Relax optical performance: EFL tolerance, image circle
4. Relax structural constraints: lens count, asphere count, cemented groups
5. Relax application match: accept similar application class as proxy

When no candidate passes at the strict level:
- Run the relaxation protocol and produce a compromise table.
- Flag the best-scoring candidate(s) for user review before Zemax entry.
- Never silently accept a candidate that fails a hard constraint.

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
  glass_substitution_report.md
  constraint_relaxation_analysis.md
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
- Leaving a microscope objective at only 0-degree field when the user specified an image circle through a tube lens.
- Treating a paraxial tube lens as permission to use a paraxial objective.
- Treating patent working distance as BFL without checking propagation direction.
- Trusting patent prose when a simple ratio check contradicts the prescription table.
- Combining original reproduction, reverse-use interpretation, scaling, and optimization into one file without labels.
- Using model glass when a good catalog match exists within distance <0.05.
- Matching glass by Nd only, ignoring Vd — this breaks chromatic correction.
- Accepting a "best match" without checking whether the glass status (preferred vs. discontinued) matters for the project.
- Relaxing constraints without documenting what was relaxed and by how much.
- Presenting the first compromise candidate as "the" solution instead of showing the trade-off table.

## Relationship To Other Tools

General Zemax guidance skills can help with optimization, analysis selection, and tolerancing. They do not replace the source-search, patent-audit, and prescription-to-Zemax workflow in this skill.
