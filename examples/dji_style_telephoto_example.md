# Example: Compact Telephoto Seed

This example shows the type of input this skill is designed for.

## Requirement

```text
EFL: medium-to-long focal length
F-number: fast imaging lens class
Image circle: large sensor class
TTL: constrained compact package
BFL: constrained detector package, including cover glass
Lens count: user-defined practical range
Field definition: angle or image circle
Wavelengths: user-defined visible or multispectral bands
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
