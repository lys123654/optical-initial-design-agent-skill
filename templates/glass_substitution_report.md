# Glass Substitution Report Template

## Source Prescription

```text
Project:
Candidate:
Source:
Date:
```

## Material Matching Results

For each material surface in the candidate prescription:

| Surface | Source Nd | Source Vd | Matched Glass | Catalog | Match Nd | Match Vd | Distance | Tolerance | Notes |
|---------|-----------|-----------|---------------|---------|----------|----------|----------|-----------|-------|
| S1 | 1.69799 | 55.5 | H-LAK51A | CDGM | 1.69680 | 55.5264 | 0.059 | close | Nd diff -0.001 |
| S3 | 1.68159 | 57.5 | H-BAK8 | CDGM | 1.57250 | 57.4868 | 0.113 | approximate | Large Nd gap; consider HOYA |

## Tolerance Legend

| Level | Criterion | Action |
|-------|-----------|--------|
| `exact` | Nd diff <0.001 AND Vd diff <0.1 | Use catalog glass, no flag needed |
| `close` | distance <0.05 | Use catalog glass, note in deviations |
| `approximate` | distance 0.05–0.20 | Present top-3 for user review |
| `no_match` | distance >0.20 | Keep model glass, mark high-risk |

## Substitution Summary

| Tolerance Level | Count | Surfaces |
|-----------------|-------|----------|
| exact | | |
| close | | |
| approximate | | |
| no_match | | |

## High-Risk Substitutions

List all surfaces with `no_match` or `approximate` tolerance. For each:
- Explain why no good match exists (unusual Nd/Vd combination, obsolete glass type, etc.)
- Suggest mitigation: multi-material optimization, relaxing Vd constraint, or accepting model glass

## Verified Glass Availability

```text
All matched catalog glasses confirmed present and active in local AGF library.
CDGM: [list]
HOYA: [list]
PLASTIC: [list]
```
