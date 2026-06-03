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

### 2. Search Candidate Sources

Search in this order unless the user asks otherwise:

1. Existing prescription libraries and public optical benches.
2. Public lens-design examples, textbooks, or vendor examples.
3. Patents through Google Patents, Justia Patents, Espacenet, The Lens, WIPO, and national patent offices.
4. Broader web search for related exact example numbers, patent family members, PDFs, and table reproductions.

Do not stop at the first plausible match. Screen at least three candidates when the search space allows it.

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

Never present a patent table as fully reliable until it has passed a numerical and modeling audit.

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

### 5. Focus And Analyze

After building the first Zemax file:

- Run Quick Focus or an equivalent image-plane focus step.
- Save both the build status and the focused file.
- Run at least one spot analysis and one MTF or first-order analysis if the license supports it.
- Save text, CSV, and image outputs when possible.
- Open the final Zemax file for visual inspection if the user asks.

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
- Opening a patent F/3 design to F/2.8 and assuming it remains valid.
- Scaling asphere coefficients with the wrong powers.
- Ignoring coordinate breaks or folded optical paths in patent examples.
- Reporting "Zemax opens" as proof that the structure is correct.

## Relationship To Other Tools

General Zemax guidance skills can help with optimization, analysis selection, and tolerancing. They do not replace the source-search, patent-audit, and prescription-to-Zemax workflow in this skill.

