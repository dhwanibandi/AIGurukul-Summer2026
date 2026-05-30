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

**Agent 1 code (Iteration 1):**

```python
import ezdxf
from ezdxf.math import Vec2
import math

def create_house_dxf():
    doc = ezdxf.new()
    msp = doc.modelspace()

    # ========== PLOT BOUNDARY ==========
    plot_vertices = [
        (0, 8), (35, 8), (35, 0), (75, 0),
        (75, 8), (80, 8), (80, 88), (75, 88),
        (75, 84), (0, 84)
    ]
    # Close the polyline by adding first point at end
    plot_vertices.append(plot_vertices[0])

    msp.add_lwpolyline(
        plot_vertices,
        dxfattribs={'layer': 'PLOT', 'color': 1}
    )

    # ========== TREE KEEP-OUT CIRCLE ==========
    tree_center = (77.5, 86)
    tree_radius = math.sqrt(3)  # ≈ 1.732 ft
    msp.add_circle(
        tree_center, tree_radius,
        dxfattribs={'layer': 'TREE', 'color': 2}
    )

    # ========== SETBACKS & MAXIMUM RECTANGLE ==========
    # Plot extreme points:
    x_min_plot = 0
    x_max_plot = 80
    y_min_plot = 0
    y_max_plot = 88

    # Setback distances
    setback_front = 7.0   # South (y=0)
    setback_rear = 7.0    # North (y=88)
    setback_west = 5.0    # West (x=0)
    setback_east = 5.0    # East (x=80)

    # Initial feasible rectangle from setbacks
    house_x_min = x_min_plot + setback_west
    house_x_max = x_max_plot - setback_east
    house_y_min = y_min_plot + setback_front
    house_y_max = y_max_plot - setback_rear

    # Adjust for tree (must not enter circle)
    # Tree center: (77.5, 86), radius 1.732
    # The tree is near the northeast corner.
    # We'll find the largest rectangle that fits entirely within the plot,
    # respects setbacks, and does not intersect the tree circle.

    # Since the tree is at top right, we reduce right side or top side if needed.
    # We'll test shifting the rectangle's top or right edge until no intersection.
    # Because we must maximize area, we will try to keep rightmost edge at x_max_house_possible.

    # First, allow rectangle to extend to setback boundaries
    # Then check collision with circle.

    # Use a helper to test if a rectangle (xmin, xmax, ymin, ymax) intersects circle
    def rect_intersects_circle(xmin, xmax, ymin, ymax, cx, cy, r):
        # Find closest point in rectangle to circle center
        closest_x = max(xmin, min(cx, xmax))
        closest_y = max(ymin, min(cy, ymax))
        dist_sq = (closest_x - cx)**2 + (closest_y - cy)**2
        return dist_sq < r**2

    # Start with largest possible (by setbacks) and adjust if needed
    trial_xmin = house_x_min
    trial_xmax = house_x_max
    trial_ymin = house_y_min
    trial_ymax = house_y_max

    # We will only trim if necessary, prioritizing area.
    # The tree is at top-right, so we may need to shrink from right or top.
    if rect_intersects_circle(trial_xmin, trial_xmax, trial_ymin, trial_ymax,
                              tree_center[0], tree_center[1], tree_radius):
        # Two strategies: move left edge right? No, left edge is fixed by setback.
        # Better: move right edge left or top edge down.
        # We try both and pick larger area.

        # Option A: move right edge left until no collision
        xmax_opt = trial_xmax
        for x in range(int(trial_xmax*10), int(trial_xmin*10), -1):
            cand_xmax = x / 10.0
            if not rect_intersects_circle(trial_xmin, cand_xmax, trial_ymin, trial_ymax,
                                          tree_center[0], tree_center[1], tree_radius):
                xmax_opt = cand_xmax
                break
        area_A = (xmax_opt - trial_xmin) * (trial_ymax - trial_ymin)

        # Option B: move top edge down until no collision
        ymax_opt = trial_ymax
        for y in range(int(trial_ymax*10), int(trial_ymin*10), -1):
            cand_ymax = y / 10.0
            if not rect_intersects_circle(trial_xmin, trial_xmax, trial_ymin, cand_ymax,
                                          tree_center[0], tree_center[1], tree_radius):
                ymax_opt = cand_ymax
                break
        area_B = (trial_xmax - trial_xmin) * (ymax_opt - trial_ymin)

        # Choose best option
        if area_A >= area_B:
            trial_xmax = xmax_opt
        else:
            trial_ymax = ymax_opt

    # Final house rectangle (closed polyline)
    house_rect = [
        (trial_xmin, trial_ymin),  # bottom-left
        (trial_xmax, trial_ymin),  # bottom-right
        (trial_xmax, trial_ymax),  # top-right
        (trial_xmin, trial_ymax),  # top-left
        (trial_xmin, trial_ymin)   # close
    ]

    msp.add_lwpolyline(
        house_rect,
        dxfattribs={'layer': 'HOUSE', 'color': 3}
    )

    # ========== ENTRY MARKER (south edge, 35 to 75 on y=0) ==========
    entry_start = (35, 0)
    entry_end = (75, 0)
    entry_mid = ((35+75)/2, 0)
    inward_length = 2.0
    entry_line_end = (entry_mid[0], entry_mid[1] + inward_length)
    msp.add_line(
        entry_mid, entry_line_end,
        dxfattribs={'layer': 'HOUSE', 'color': 4}
    )
    msp.add_circle(
        entry_mid, 0.2,
        dxfattribs={'layer': 'HOUSE', 'color': 4}
    )

    # Save the DXF
    doc.saveas("house.dxf")
    print("DXF file 'house.dxf' generated successfully.")

    return house_rect[:-1]  # exclude the closing duplicate

# ========== EXECUTE ==========
house_corners = create_house_dxf()

print("\nFinal list of house-corner coordinates in order (feet, x,y):")
for corner in house_corners:
    print(f"({corner[0]:.2f}, {corner[1]:.2f})")
```

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

**Agent 1 code (Iteration 2 — corrected):**

```python
import ezdxf
import math

def create_house_dxf():
    doc = ezdxf.new()
    msp = doc.modelspace()

    # ========== PLOT BOUNDARY ==========
    plot_vertices = [
        (0, 8), (35, 8), (35, 0), (75, 0),
        (75, 8), (80, 8), (80, 88), (75, 88),
        (75, 84), (0, 84)
    ]
    plot_vertices.append(plot_vertices[0])  # close polyline
    msp.add_lwpolyline(
        plot_vertices,
        dxfattribs={'layer': 'PLOT', 'color': 1}
    )

    # ========== PROTECTED TREE ==========
    tree_center = (77.5, 86)
    tree_radius = math.sqrt(3)  # ≈ 1.732 ft
    msp.add_circle(
        tree_center, tree_radius,
        dxfattribs={'layer': 'TREE', 'color': 2}
    )

    # ========== CORRECTED HOUSE RECTANGLE ==========
    # Based on actual plot boundary, not bounding box:
    # West setback 5 ft  -> x_min = 0 + 5 = 5
    # East setback 5 ft  -> x_max = 80 - 5 = 75
    # Front (south) edge: binding y = 8 (for x < 35) + 7 = 15
    # Rear (north) edge:  y = 84 - 7 = 77
    house_corners = [
        (5, 15),   # bottom-left
        (75, 15),  # bottom-right
        (75, 77),  # top-right
        (5, 77),   # top-left
        (5, 15)    # close
    ]
    msp.add_lwpolyline(
        house_corners,
        dxfattribs={'layer': 'HOUSE', 'color': 3}
    )

    # ========== ENTRY MARKER (south edge, 35 to 75 on y=0) ==========
    entry_mid = ((35 + 75) / 2, 0)
    inward_length = 2.0
    entry_line_end = (entry_mid[0], entry_mid[1] + inward_length)
    msp.add_line(
        entry_mid, entry_line_end,
        dxfattribs={'layer': 'HOUSE', 'color': 4}
    )
    msp.add_circle(
        entry_mid, 0.2,
        dxfattribs={'layer': 'HOUSE', 'color': 4}
    )

    # ========== SAVE DXF ==========
    doc.saveas("house.dxf")
    print("DXF file 'house.dxf' generated successfully with corrected house rectangle.")

    return house_corners[:-1]  # exclude closing duplicate

# Run and display corners
final_corners = create_house_dxf()

print("\nFinal list of house-corner coordinates in order (feet, x, y):")
for corner in final_corners:
    print(f"({corner[0]:.2f}, {corner[1]:.2f})")
```

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
