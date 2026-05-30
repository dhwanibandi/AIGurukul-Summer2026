**Task:** Build a two-agent agentic system that designs the outer outline of a house on a given plot and verifies it against logical and geometric constraints, iterating until all constraints pass.

---

## 1. Approach Overview

The system uses two cooperating LLM agents in a generate-verify loop:

| Agent | Role | Job |
|-------|------|-----|
| **Agent 1 (Generator)** | Architect | Generates Python (`ezdxf`) code that draws the plot, the protected tree, and the house outline, and outputs the house-corner coordinates. |
| **Agent 2 (Verifier)** | Inspector | Checks Agent 1's output against the fixed constraint set, reports pass/fail per constraint, and returns specific feedback on what to fix. |

The loop runs as: **A1 generates → A2 verifies → if FAIL, feedback goes back to A1 → A1 regenerates → repeat until PASS.**

This run was the **manual / conceptual version** described in the problem statement's "Simplified First Attempt" - the agents were driven by hand (copy-paste between them) rather than via API automation. The next step is to replace Agent 2 with a Z3 solver for formal verification.

### Models used

- **Agent 1:** DeepSeek, standard chat mode (deep-thinking / reasoning mode **disabled**).
- **Agent 2:** Claude Opus 4.8, **medium** effort setting.

### Key design principle

Both agents were given an **identical, locked constraint specification** every round. This prevents "spec drift" - if the verifier checks against even slightly different numbers than the generator was given, the loop becomes meaningless. The constraint block was treated as the single source of truth.

---

## 2. The Plot

The plot is an irregular, fully rectilinear (all edges horizontal/vertical) polygon. Units are in feet, origin at bottom-left, North = +Y.

**Boundary vertices (in order, closed):**

```
(0, 8) → (35, 8) → (35, 0) → (75, 0) → (75, 8) → (80, 8)
→ (80, 88) → (75, 88) → (75, 84) → (0, 84) → back to (0, 8)
```
<img width="2000" height="2000" alt="desmos-graph (1)-1" src="https://github.com/user-attachments/assets/79baee8c-4bfd-4ab2-a758-8ebcce349299" />

---

## 3. Constraints (the locked spec)

1. The house outline lies **entirely inside the plot**.
2. The house respects the **setbacks** from every boundary edge.
3. The house does **not enter the protected tree** keep-out circle.
4. The house **maximizes its footprint** within the legal area.

---

## 4. Assumptions Taken on the Seattle Code (SBC / SMC)

The setback source provided was **Seattle Municipal Code §23.45.518** (residential setbacks). The following assumptions and simplifications were made to turn that code into usable numbers:

1. **Code section used:** Applied §23.45.518, which governs the **multifamily** residential zones (LR / MR / HR). A single detached house technically falls under a different SMC chapter, but §23.45.518 was the provided source, so it was used for consistency.

2. **Zone chosen - LR (Lowrise):** Among LR / MR / HR, the **LR** zone was selected because it is the lowest-density and most house-like, and it yields the largest buildable area (smallest required setbacks).

3. **"7 ft average, 5 ft minimum" → uniform 7 ft.** The LR front and rear setback is stated as "7 ft average, 5 ft minimum." For a simple rectangular house with a *uniform* setback, the value must satisfy the **average** (7 ft), not just the minimum (5 ft). So a flat **7 ft** was used for front and rear. Sides are a flat **5 ft**.

5. **Final setback values used:**
   - Front (South): **7 ft**
   - Rear (North): **7 ft**
   - Sides (East / West): **5 ft each**

7. **Tree radius simplified to √3 ≈ 1.732 ft.** Real Seattle tree protection uses a **Critical Root Zone (CRZ)** sized from the trunk diameter (~1 ft of radius per 1 inch of DBH). We did **not** use the CRZ formula; instead a fixed small radius of **√3 ft** was assumed for this exercise. The tree center was taken as **(77.5, 86)**, sitting inside the 4×5 ft top-right notch.

10. **Geometry simplifications.** The problem was treated as 2D, axis-aligned, with the house footprint allowed to be a **single rectangle** (rather than an L-shape) for the maximize-area objective.

---

## 5. Derived Legal Envelope

From the actual boundary edges (not the bounding box) plus the setbacks:

| Boundary | Real lot line | Setback | Resulting limit |
|----------|---------------|---------|-----------------|
| West | x = 0 | 5 ft | x ≥ **5** |
| East | x = 80 | 5 ft | x ≤ **75** |
| North (rear) | y = 84 (for x ≤ 75) | 7 ft | y ≤ **77** |
| South (front) | y = 8 (outside central notch) | 7 ft | y ≥ **15** |

> The South limit is **15**, not 7, because the house spans x < 35, where the plot floor is y = 8 (the y = 0 floor only exists in the central notch). This single fact is what Agent 1 missed in iteration 1.

**Largest legal single rectangle:** `(5, 15) → (75, 15) → (75, 77) → (5, 77)` = **70 ft × 62 ft = 4,340 sq ft.**

---

## 6. Iteration Log

### Iteration 1 - FAIL (3 / 6)

**Agent 1 output:** `(5, 7), (75, 7), (75, 81), (5, 81)`

**Root cause:** Agent 1 computed setbacks from the plot's **bounding box** (`x: 0-80, y: 0-88`) instead of the actual irregular edges.

| # | Constraint | Result |
|---|-----------|--------|
| 1 | Inside plot | **FAIL** - bottom edge y = 7 is below the plot floor (y = 8) for x ∈ [5, 35) |
| 2 | Setbacks | **FAIL** - rear used y = 88 → top = 81 (real edge y = 84 → should be 77); front used y = 0 → bottom = 7 (real edge y = 8 → should be 15) |
| 3 | Clears tree | PASS |
| 4 | Maximize footprint | **FAIL** - invalid envelope |
| 5 | Single south entry | PASS |
| 6 | Outer periphery only | PASS |

**Feedback to Agent 1:** Stop using min/max bounding coordinates. North lot line is y = 84 (not 88); south lot line is y = 8 except the central notch. Corrected target: `(5, 15), (75, 15), (75, 77), (5, 77)`.

### Iteration 2 - PASS (6 / 6)

**Agent 1 output:** `(5, 15), (75, 15), (75, 77), (5, 77)`

| # | Constraint | Result |
|---|-----------|--------|
| 1 | Inside plot | **PASS** |
| 2 | Setbacks | **PASS** - W/E at x = 5/75, front y = 15, rear y = 77 |
| 3 | Clears tree | **PASS** - closest approach ≈ 9.3 ft vs √3 ≈ 1.73 ft |
| 4 | Maximize footprint | **PASS** - 4,340 sq ft, the largest legal rectangle |
| 5 | Single south entry | **PASS** - marker at (55, 0) |
| 6 | Outer periphery only | **PASS** |

**The loop converged in 2 iterations.**
<img width="2000" height="2000" alt="desmos-graph-1" src="https://github.com/user-attachments/assets/a20f67d7-c8ce-4c97-a232-5b9584f472c0" />


---

## 7. Observations

- **Agent 1's logic was correct; the bug was a wrong geometric assumption** (bounding box = plot). It was invisible inside the code itself - the collision/setback math worked fine - and only surfaced when checked against the *real* boundary. This is a strong argument for why a separate verifier is necessary.
- **Specific, numeric feedback fixed it in one shot.** Agent 1 was anchored to a simplification; vague feedback ("the setbacks are wrong") likely would not have broken that anchor, but exact corrected coordinates did.
- **A locked, identical spec to both agents was essential** to avoid drift between what was generated and what was checked.
- **The tree constraint was non-binding** (it sits in the corner notch, far from the buildable area), so it passed for free.

---

*Models: Agent 1 - DeepSeek (chat mode, reasoning disabled); Agent 2 - Claude Opus 4.8 (medium effort). Verification this round was manual/LLM-based; Z3 formalization is pending.*
