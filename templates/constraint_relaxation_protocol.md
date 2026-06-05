# Constraint Relaxation Protocol

When no candidate prescription passes all strict requirements, use this protocol to systematically relax constraints and find the best achievable compromise.

## Priority Levels

Each constraint in `specs.json` carries a priority:

| Level | Weight | Meaning | Example |
|-------|--------|---------|---------|
| `hard` | 1000 | Must pass. Cannot be relaxed. | Application type, system class |
| `high` | 100 | Strong preference. Relax only within defined bounds. | EFL, F/#, image circle |
| `medium` | 10 | Can trade off if needed. | TTL, BFL, lens count |
| `low` | 1 | Can be deprioritized or dropped. | CRA, distortion, RI |

## Default Relaxation Order

Relax in this order (user may override):

### Step 1: Relax F/#
F/# is often the highest-impact relaxation because many patent lenses exist at slightly slower speeds.
- Strict: F/2.8
- Step 1a: F/3.5 (half-stop slower)
- Step 1b: F/4.0 (one stop slower)
- Step 1c: F/4.5

### Step 2: Relax Packaging Constraints
- TTL: +10% → +20% → +30% → drop TTL check
- BFL: -10% → -20% → -30% → drop BFL check

### Step 3: Relax Optical Performance
- EFL: ±5% → ±10% → ±20%
- Image circle: -5% → -10% → -20%

### Step 4: Relax Structural Constraints
- Lens count: +1 element → +2 elements → +4 elements → drop check
- Asphere count: +1 surface → +2 surfaces → drop check
- Cemented groups: allow one more → drop limit

### Step 5: Relax Application Match
- Accept similar application class (e.g., folded phone telephoto as compact telephoto proxy)
- Accept zoom long-end prescription as fixed-focal-length seed
- Accept scaled prescription (original f differs by >20% but can be uniformly scaled)

## Compromise Scoring

For each candidate, compute:
```
score = Σ(w_i × pass_i)
```
where `w_i` is the priority weight and `pass_i` is 1 if the constraint is met, 0 if not.

Present results as a ranked table:

| Rank | Candidate | EFL | F/# | TTL | BFL | IC | Count | Score | Relaxations |
|------|-----------|-----|-----|-----|-----|-----|-------|-------|-------------|
| 1 | KR102742776B1 Ex7 (scaled) | PASS | PASS | PASS | FAIL | PASS | 10 | 110 | BFL relaxed to 4.9mm |
| 2 | US20260029627 Ex1 (scaled) | PASS | PASS | FAIL | PASS | PASS | 6 | 101 | TTL relaxed to 83mm |

## Reporting

Always report:
1. Which constraints were relaxed and by how much (absolute and percentage)
2. The relaxation step that produced each candidate
3. Whether the relaxed candidate still serves as a viable starting point for optimization
4. Which relaxed constraints are likely recoverable through optimization (e.g., BFL can often be increased with merit function operands) vs. fundamental (e.g., TTL/f ratio is structural)

## Human Review Gate

Before committing any relaxed candidate to Zemax:
- Present the compromise table.
- Flag the best-scoring candidate(s).
- Ask the user to confirm relaxation decisions.
- Never silently accept a candidate that fails a hard constraint.
