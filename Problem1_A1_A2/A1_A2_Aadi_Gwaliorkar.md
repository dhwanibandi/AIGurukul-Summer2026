**Task:** Build a two-agent agentic system that designs the outer outline of a house on a given plot and verifies it against logical and geometric constraints, iterating until all constraints pass.

---

## 1. Approach Overview

The system uses two cooperating LLM agents in a generate-verify loop:

| Agent | Role | Job |
|-------|------|-----|
| **Agent 1 (Generator)** | Architect | Generates Python (`ezdxf`) code that draws the plot, the protected tree, and the house outline, and outputs the house-corner coordinates. |
| **Agent 2 (Verifier)** | Inspector | Checks Agent 1's output against the fixed constraint set, recomputes every value, reports pass/fail per constraint, and returns specific feedback on what to fix. |

The loop runs as: **A1 generates → A2 verifies → if FAIL, feedback goes back to A1 → A1 regenerates → repeat until PASS.**

This run is the **manual / conceptual version** described in the problem statement's "Simplified First Attempt" — the agents were driven by hand (copy-paste between them) rather than via API automation. The next step is to replace Agent 2's checks with a Z3 solver for formal verification.

### Models used

- **Agent 1 (Generator):** Claude.
- **Agent 2 (Verifier):** Moonshot.

### Key design principle

Both agents were given an **identical, locked constraint specification** every round. This prevents "spec drift" — if the verifier checks against even slightly different numbers than the generator was given, the loop becomes meaningless. The constraint block was treated as the single source of truth, and the buildable area was derived from the **real boundary edges** before either agent ran.

---

## 2. The Plot

The plot is an irregular, fully rectilinear (all edges horizontal/vertical) polygon. Units are in feet, origin at bottom-left, North = +Y.

![Plot diagram with dimensions, the protected tree keep-out, and the final house footprint](plot_diagram.png)

**Boundary vertices (in order, closed):**

```
(0, 8) → (35, 8) → (35, 0) → (75, 0) → (75, 8) → (80, 8)
→ (80, 88) → (75, 88) → (75, 84) → (0, 84) → back to (0, 8)
```

Features: a 40-ft-wide central notch drops to y = 0 at the bottom (x ∈ [35, 75], the street/entry side), and a small 4 × 5 ft step sits at the top-right (x ∈ [75, 80], y ∈ [84, 88]). The plot encloses 6,420 sq ft.

---

## 3. Constraints (the locked spec)

1. The house outline lies **entirely inside the plot**.
2. The house respects the **setbacks** from every boundary edge.
3. The house does **not enter the protected tree** keep-out circle.
4. The house **maximizes its footprint** within the legal area.
5. There is exactly **one entry, on the South side**.
6. Only the **outer periphery** is drawn — one single closed polyline, no interior rooms.

---

## 4. Assumptions Taken on the Seattle Code (SBC / SMC)

The following assumptions were made to turn the Seattle code into usable numbers:

1. **Code section used — §23.44 (single-family).** Applied **Seattle Municipal Code §23.44**, which governs **single-family / Neighborhood Residential** zones. A detached single house falls under this chapter, so it is the more correct reading than the multifamily §23.45.518. (This is the main point of difference from a multifamily reading, which would use smaller setbacks and yield a larger box.)

2. **Final setback values used:**
   - Front (South): **20 ft**
   - Rear (North): **25 ft**
   - Sides (East / West): **5 ft each**

3. **Setbacks measured from the ACTUAL boundary edges, not the bounding box.** This is the key geometric subtlety on an irregular lot (see §5).

4. **Tree radius simplified to √3 ≈ 1.732 ft.** Real Seattle tree protection uses a **Critical Root Zone (CRZ)** sized from trunk diameter; the CRZ formula was **not** used. Instead a fixed small radius of **√3 ft** was assumed, with the tree centre at **(77.5, 86)**, sitting inside the 4 × 5 ft top-right notch.

5. **Geometry simplifications.** Treated as 2D, axis-aligned, with the house footprint allowed to be a **single rectangle** (rather than an L-shape) for the maximize-area objective.

---

## 5. Derived Legal Envelope

From the actual boundary edges (not the bounding box) plus the setbacks:

| Boundary | Real lot line | Setback | Resulting limit |
|----------|---------------|---------|-----------------|
| West | x = 0 | 5 ft | x ≥ **5** |
| East | x = 80 | 5 ft | x ≤ **75** |
| North (rear) | y = 84 (for x ≤ 75) | 25 ft | y ≤ **59** |
| South (front) | y = 8 (outside central notch) | 20 ft | y ≥ **28** |

> The South limit is **28**, not 20, because a single full-width rectangle spans x < 35, where the plot floor is y = 8 (the y = 0 floor only exists in the central notch, x ∈ [35, 75]). Using the bounding-box floor (y = 0) or the bounding-box ceiling (y = 88) is the classic trap on this lot.

**Largest legal single rectangle:** `(5, 28) → (75, 28) → (75, 59) → (5, 59)` = **70 ft × 31 ft = 2,170 sq ft.**

---

## 6. Iteration Log

### Iteration 1 — PASS (6 / 6)

**Agent 1 output:** `(5, 28), (75, 28), (75, 59), (5, 59)`

| # | Constraint | Result |
|---|-----------|--------|
| 1 | Inside plot | **PASS** — all four corners and edges within the boundary |
| 2 | Setbacks | **PASS** — front 20 (from y = 8), rear 25 (from y = 84), sides 5 (from x = 0 / 80) |
| 3 | Clears tree | **PASS** — closest approach ≈ 25.38 ft vs keep-out radius √3 ≈ 1.73 ft |
| 4 | Maximize footprint | **PASS** — 2,170 sq ft, the largest legal rectangle |
| 5 | Single south entry | **PASS** — one marker on the south face at x ∈ [53, 57], y = 28 |
| 6 | Outer periphery only | **PASS** — one closed LWPOLYLINE, no interior geometry |

**The loop converged on the first iteration.**

Because the buildable envelope was derived from the real edges and locked into the shared spec, Agent 1 did not fall into the bounding-box trap, and the verifier confirmed every value by recomputation.

---

## 7. Observations

- **The hard part lived in the spec, not the loop.** The one genuinely error-prone step on this lot is turning the irregular boundary into setback limits (real edges y = 8 / y = 84, not the bounding box y = 0 / y = 88). Front-loading the derived envelope into the locked spec is what produced a first-pass convergence.
- **Chapter choice changes the answer.** Using single-family §23.44 (20 / 25 / 5) yields **2,170 sq ft**. A multifamily reading (§23.45.518, 7 / 7 / 5) would yield a larger box (~4,340 sq ft). The result is therefore an assumption-driven number, and §23.44 is the defensible chapter for a detached house.
- **The tree constraint was non-binding** — it sits in the corner notch, ~25 ft from the buildable area, so it passed for free.
- **The single-rectangle simplification leaves area unused.** The notch's lower strip (x ∈ [35, 75], y ∈ [20, 28], ~320 sq ft of the 2,490 sq ft envelope) cannot be captured by one rectangle; an L-shaped outline would recover it.


---

*Models: Agent 1 — Claude; Agent 2 — Moonshot. Verification this round was manual / LLM-based; Z3 formalization is pending.*
