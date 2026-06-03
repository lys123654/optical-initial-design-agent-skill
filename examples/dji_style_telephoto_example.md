# Example: DJI-Style Compact Telephoto Seed

This example shows the type of input this skill is designed for.

## Requirement

```text
EFL: 74 mm
F-number: F2.8
Image circle: 16 mm
TTL: <55.5 mm
BFL: >6 mm, including 0.3 mm cover glass
Lens count: 5-15
Field definition: angle or image circle
Wavelengths: 435/486/546/587/656 nm plus 850 nm review
```

## Lessons From MVP

- A relaxed prescription-library seed can validate the workflow even when it fails strict packaging.
- A strict patent hit can still fail because BFL, folded geometry, or F-number changes break the model.
- Coefficient audit must be separate from optical-performance review.
- Layout inspection is mandatory; a saved ZMX file is not enough.

## Expected Output

```text
dji_style_telephoto/
  specs.json
  candidate_search_report.md
  candidate_audits/
  zemax/
  analysis_outputs/
  initial_review.md
```

