# AI Verifiable Output — Autocad 2D drawing Generation and Verification

## Problem Statement

The goal was to use two AI agents — one to generate code and one to verify it — in a feedback loop. A1 generates Python code (using ezdxf) that produces a .dxf layout for a given property plot. A2 verifies the output against geometric constraints and flags any errors. A1 then uses A2's feedback to fix the code. This loop continues until A2 gives a full pass. The objective is to maximize the buildable area within the plot while satisfying all SBC setback rules.

**Agent roles:**
- **A1 (ChatGPT)** — acts as an expert architectural CAD drafter and Python developer. Generates the ezdxf code, traces the property boundary, applies setbacks, and outputs the building footprint that maximizes buildable area.
- **A2 (Gemini)** — acts as a geometry verifier. Uses Z3-style mathematical reasoning to check the generated code against 5 constraints — boundary accuracy, tree placement, setback offsets, entry acknowledgment, and area calculation. Returns a structured PASS/FAIL report with specific coordinate-level feedback.

**Plot (irregular polygon, all dimensions in feet):**

| Segment | Direction | Length | To |
|---|---|---|---|
| Start | — | — | (0, 0) |
| West wall | North | 76 ft | (0, 76) |
| North wall (lower) | East | 75 ft | (75, 76) |
| Step up | North | 4 ft | (75, 80) |
| North wall (upper) | East | 5 ft | (80, 80) |
| East wall | South | 80 ft | (80, 0) |
| Step in | West | 5 ft | (75, 0) |
| Right notch wall | South | 8 ft | (75, -8) |
| South entry wall | West | 40 ft | (35, -8) |
| Left notch wall | North | 8 ft | (35, 0) |
| South base wall | West | 35 ft | (0, 0) |

**Physical constraints:**
- Entry on the 40ft south-facing wall between (35, −8) and (75, −8), to be marked in the drawing
- Existing tree at center (77.5, 78), radius 1.5ft — cannot be removed or built over

**SBC setbacks:**
- Front: 20ft inward from the south entry wall
- Rear: 25ft inward from the north wall
- Sides: 5ft inward from all remaining walls

**Output required:** Python/ezdxf code with layers `PROPERTY_LINE`, `BUILDING_FOOTPRINT`, `TREE`, `ENTRY`, `DIRECTIONS`. Must print buildable area and % utilization. A sketch of the plot is attached.

### Task Flowchart

```
        ┌─────────────────────────┐
        │  A1 (ChatGPT) generates │
        │      Python/DXF code    │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │  A2 (Gemini) verifies   │
        │  5 geometric constraints│
        └────────────┬────────────┘
                     │
           ┌─────────┴──────────┐
           │                    │
         FAIL                  PASS
           │                    │
           ▼                    ▼
  ┌─────────────────┐   ┌───────────────┐
  │ A2 feedback sent│   │  Final output │
  │   back to A1    │   │    accepted   │
  └────────┬────────┘   └───────────────┘
           │
           ▼
     A1 revises code
           │
           └──────► [A2 verifies again]
```

---

## Approach

A1 was given the full plot data and constraints in the prompt. Its output was passed directly to A2. If A2 returned any FAIL, that feedback was forwarded back to A1 to fix and regenerate. The loop ran until A2 gave a full PASS on all 5 constraints.

---

## Prompts

### A1 Prompt (sent to ChatGPT)

```
Role: You are an expert Architectural CAD Drafter and Python Developer specializing in the ezdxf library.

Task: I want you to generate a layout for a given property plot in the form of Python code that outputs a valid .dxf file. Your task is to strictly follow the plot dimensions, respect the entry and tree locations, apply the specified open space setback constraints, and draw an inner building footprint that maximizes the allowable buildable area.

1. Plot Dimensions (in feet):

Please map the x and y axes according to standard compass directions (North = +y, East = +x). Assume the starting point (bottom-left corner of the main left wall) is at coordinate (0,0). Trace the property boundary clockwise using the following dimensions:

Start at (0,0).
Go North: 76 ft to (0, 76).
Go East: 75 ft to (75, 76).
Go North: 4 ft to (75, 80).
Go East: 5 ft to (80, 80).
Go South: 80 ft to (80, 0).
Go West: 5 ft to (75, 0).
Go South: 8 ft to (75, -8).
Go West: 40 ft to (35, -8). (Note: This 40ft south-facing wall is the primary Entry).
Go North: 8 ft to (35, 0).
Go West: 35 ft.

2. Physical Constraints:

Entry: A single entry must be placed on the 40 ft South wall (between X=35 and X=75 at Y=-8). Ensure this is shown in the drawing.

Tree: According to SBC rules, the existing tree cannot be cut. Draw a circle representing the tree.
Tree Center Coordinate: (77.5, 78) — which places it perfectly in the top-right alcove.
Tree Radius: 1.5 ft.
Ensure the building footprint does not overlap the tree.

3. SBC Building Offset Constraints:

Calculate the inner building footprint by offsetting the outer boundary inward according to these specific requirements:

Front Setback: Apply a 20-foot setback inward from the 40 ft South entry wall.
Rear Setback: Apply a 25-foot setback inward from the primary North wall.
Side Setbacks: Apply a standard 5-foot setback inward from all remaining exterior property lines (East walls, West walls, and any other walls).

4. Goal - Maximize Area:
Among all the layout configurations that satisfy the physical boundaries, the tree preservation, and the offset constraints, calculate and draw the building footprint that yields the maximum possible buildable area.

Output Requirements:
Output only the final, complete Python code using ezdxf, the maximum buildable area achieved and the percentage utilisation.
Include clear comments in the code explaining how the coordinates were resolved and how the 20ft, 25ft, and 5ft offsets were applied.
Draw the outer property boundary on one layer ("PROPERTY_LINE"), the maximum building footprint on another ("BUILDING_FOOTPRINT"), and the Tree on a third ("TREE"). Also mention the entry ("ENTRY") and the directions.
```

---

### A2 Prompt (sent to Gemini)

```
Role: You are an expert Geometry Verifier. Your task is to verify a CAD drawing (written in Python using the ezdxf library) with respect to strict physical and legal parameters.

Methodology: You must apply Z3 solver logic. You must mathematically simulate the coordinate geometry step-by-step. Define the boundary lines, apply the setback offsets mathematically, check for coordinate intersections, and verify that the constraints are satisfied without logical contradictions.

Input:
Below, I will attach a Python script that generates a .dxf layout.

Constraints to Verify:

1. Outer Dimensions (The Property Line):
Does the code trace this exact clockwise closed polygon starting at (0,0)?
(0,0) → (0,76) → (75,76) → (75,80) → (80,80) → (80,0) → (75,0) → (75,-8) → (35,-8) → (35,0) → (0,0)

2. Tree Preservation:
Is there a circle at Center: (77.5, 78) with Radius: 1.5?
Does any part of the BUILDING_FOOTPRINT enter the bounding box X:75 to 80, Y:76 to 80?

3. SBC Building Offsets (The Building Footprint):
Front Setback: Is the footprint offset exactly 20 feet North from the South entry wall?
Rear Setback: Is the footprint offset exactly 25 feet South from the primary North wall?
Side Setbacks: Are all other footprint lines offset exactly 5 feet inward from their corresponding property lines?

4. Entry Constraint:
Does the code logically acknowledge the Entry on the South wall (X=35 to 75, Y=-8)?

5. Max Area Calculation and Percentage Coverage:
Did the script calculate and output the area of the generated footprint?
Based on your geometric verification, is the calculated area and coverage mathematically accurate and maximized?

Output Requirements:
Do not write or rewrite the drawing code. Your output must strictly follow this format:

[Constraint 1: Outer Dimensions] - PASS / FAIL
Feedback: (If FAIL, state exactly which coordinate or vector is wrong and why).

[Constraint 2: Tree Preservation] - PASS / FAIL
Feedback: ...

[Constraint 3: SBC Building Offsets] - PASS / FAIL
Feedback: ...

[Constraint 4: Entry] - PASS / FAIL
Feedback: ...

[Constraint 5: Max Area] - PASS / FAIL
Feedback: ...

FINAL VERIFICATION: [PASS / FAIL]
(Only grant a Final PASS if all 5 constraints pass.)
```

---

## Iterations

### Iteration 1

**A1 Output:** Rectangular footprint — `(5,12) → (75,12) → (75,51) → (5,51)`
Buildable: 2,730 sq ft | Plot area: 6,120 sq ft | Utilization: 44.61%

| Constraint | Result |
|---|---|
| 1 — Outer Dimensions | PASS |
| 2 — Tree Preservation | PASS |
| 3 — SBC Offsets | FAIL |
| 4 — Entry | PASS |
| 5 — Max Area | FAIL |
| **FINAL** | **FAIL** |

**Constraint 3 — A2 Feedback:**
> "The Side Setback for the South-Western property line is geometrically flawed. The property boundary from X=0 to X=35 rests at Y=0. Applying the 5ft inward side setback should place the bottom footprint boundary for this segment at Y=5. The code erroneously extends the 20ft front entry setback (Y=12) across the entire width of the plot, chopping off the valid buildable area on the west side."

**Constraint 5 — A2 Feedback:**
> "The area calculation is not maximized due to the logical error in Constraint 3. By flattening the entire southern footprint boundary to Y=12, the code creates a contiguous rectangular void of legally buildable space between X=5 to X=35 and Y=5 to Y=12. Your script calculates a maximum area of 2,730 sq ft, but the true mathematically maximized area is 2,940 sq ft."

---

### Iteration 2

**A1 Output:** L-shaped footprint — `(5,5) → (35,5) → (35,12) → (75,12) → (75,51) → (5,51)`
Buildable: 2,940 sq ft | Plot area: 6,120 sq ft (wrong) | Utilization: 48.04%

| Constraint | Result |
|---|---|
| 1 — Outer Dimensions | PASS |
| 2 — Tree Preservation | PASS |
| 3 — SBC Offsets | PASS |
| 4 — Entry | PASS |
| 5 — Max Area | FAIL |
| **FINAL** | **FAIL** |

**Constraint 5 — A2 Feedback:**
> "You nailed the absolute maximum building area at exactly 2,940 sq ft. However, the `property_area` calculation is mathematically flawed, which in turn invalidates the Utilization Percentage. Your formula `(80 * 80) - (35 * 8)` calculates 6,120 sq ft, assuming the property is an 80x80 square minus a southwest notch. The true area is the sum of its distinct blocks: Main Y=0 to 76 block (X=0 to 75): 75×76 = 5,700 | Eastern Y=0 to 80 block (X=75 to 80): 5×80 = 400 | Southern entry bump-out (X=35 to 75, Y=0 to -8): 40×8 = 320 | True Plot Area: 6,420 sq ft. This means your true utilization is 2,940 / 6,420 = 45.79%, not 48.04%."

---

### Iteration 3

**A1 Output:** Same L-shaped footprint, property area corrected to 6,420 sq ft.
Buildable: 2,940 sq ft | Plot area: 6,420 sq ft | Utilization: 45.79%

| Constraint | Result |
|---|---|
| 1 — Outer Dimensions | PASS |
| 2 — Tree Preservation | PASS |
| 3 — SBC Offsets | PASS |
| 4 — Entry | PASS |
| 5 — Max Area | PASS |
| **FINAL** | **PASS** |

---

## Final Code

```python
import ezdxf

# ============================================================
# SBC-Compliant Plot Layout (Maximum Area Version)
# Units: Feet
#
# Coordinate System:
#   North = +Y
#   East  = +X
#
# The footprint is generated by applying:
#   - 20 ft front setback only from the primary entry wall
#   - 25 ft rear setback from the primary north wall
#   - 5 ft side setback from all remaining property lines
#
# This yields the mathematically maximized SBC-compliant
# buildable footprint.
# ============================================================

doc = ezdxf.new("R2010")
msp = doc.modelspace()

# ------------------------------------------------------------
# Layers
# ------------------------------------------------------------
doc.layers.new("PROPERTY_LINE")
doc.layers.new("BUILDING_FOOTPRINT")
doc.layers.new("TREE")
doc.layers.new("ENTRY")
doc.layers.new("DIRECTIONS")

# ------------------------------------------------------------
# PROPERTY BOUNDARY
# ------------------------------------------------------------
property_boundary = [
    (0, 0),
    (0, 76),
    (75, 76),
    (75, 80),
    (80, 80),
    (80, 0),
    (75, 0),
    (75, -8),
    (35, -8),
    (35, 0),
    (0, 0)
]

msp.add_lwpolyline(
    property_boundary,
    close=True,
    dxfattribs={"layer": "PROPERTY_LINE"}
)

# ------------------------------------------------------------
# SBC OFFSET RESOLUTION
#
# Front Setback:
#   Entry wall = segment (35,-8) to (75,-8)
#   20 ft inward => Y = -8 + 20 = 12
#
# Rear Setback:
#   Primary north wall Y = 76
#   25 ft inward => Y = 76 - 25 = 51
#
# Side Setbacks:
#   West boundary X = 0  -> X = 5
#   East boundary X = 80 -> X = 75
#   South-west boundary Y = 0 -> Y = 5
#
# Maximum valid footprint polygon:
#   (5,5) -> (35,5) -> (35,12) -> (75,12) -> (75,51) -> (5,51)
#
# Area:
#   Main rectangle: 70 x 39 = 2730
#   Added west block: 30 x 7  = 210
#   Total = 2940 sq ft
# ------------------------------------------------------------

building_footprint = [
    (5, 5),
    (35, 5),
    (35, 12),
    (75, 12),
    (75, 51),
    (5, 51)
]

msp.add_lwpolyline(
    building_footprint,
    close=True,
    dxfattribs={"layer": "BUILDING_FOOTPRINT"}
)

# ------------------------------------------------------------
# TREE PRESERVATION
# ------------------------------------------------------------
tree_center = (77.5, 78)
tree_radius = 1.5

msp.add_circle(
    tree_center,
    tree_radius,
    dxfattribs={"layer": "TREE"}
)

msp.add_text(
    "TREE",
    height=1.5,
    dxfattribs={"layer": "TREE"}
).set_placement((74, 82))

# ------------------------------------------------------------
# ENTRY LOCATION
# ------------------------------------------------------------
entry_start = (52, -8)
entry_end = (58, -8)

msp.add_line(
    entry_start,
    entry_end,
    dxfattribs={"layer": "ENTRY"}
)

msp.add_text(
    "ENTRY",
    height=1.5,
    dxfattribs={"layer": "ENTRY"}
).set_placement((50, -12))

# ------------------------------------------------------------
# DIRECTION MARKERS
# ------------------------------------------------------------
msp.add_line(
    (90, 20),
    (90, 35),
    dxfattribs={"layer": "DIRECTIONS"}
)

msp.add_text(
    "N",
    height=2,
    dxfattribs={"layer": "DIRECTIONS"}
).set_placement((89, 37))

msp.add_text(
    "E",
    height=2,
    dxfattribs={"layer": "DIRECTIONS"}
).set_placement((95, 20))

# ------------------------------------------------------------
# AREA CALCULATIONS
# ------------------------------------------------------------
main_rectangle_area = (75 - 5) * (51 - 12)      # 2730
west_extension_area = (35 - 5) * (12 - 5)       # 210

max_buildable_area = main_rectangle_area + west_extension_area

# Property area — broken down by region:
#   Main block (X=0-75, Y=0-76):  75 x 76 = 5700
#   East strip (X=75-80, Y=0-80):  5 x 80 =  400
#   South bump-out (X=35-75, Y=-8-0): 40 x 8 = 320
#   Total: 6420 sq ft
property_area = 6420

utilization_percent = (max_buildable_area / property_area) * 100

print(f"Maximum Buildable Area: {max_buildable_area:.2f} sq ft")
print(f"Property Area: {property_area:.2f} sq ft")
print(f"Utilization: {utilization_percent:.2f}%")
```

**Output:**
```
Maximum Buildable Area: 2940.00 sq ft
Property Area: 6420.00 sq ft
Utilization: 45.79%
```

---

## Findings
 
- Both agents performed better when data and values were typed out directly — giving images or documents led to worse results
- A1 sometimes manipulated its calculations to match what was already in the code, even when that was incorrect
- Number of iterations was not fixed — in some runs the first output was already correct, in others it took more rounds
- A2 (verifier) was not always reliable — in some runs it passed an incorrect layout
- Most errors were in misinterpreting the plot boundaries, specifically which setback applies to which wall.