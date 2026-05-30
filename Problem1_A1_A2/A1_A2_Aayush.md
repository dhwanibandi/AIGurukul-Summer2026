# Neuro-Symbolic Multi-Agent CAD Verification for Seattle-Compliant Site Planning

## Problem Statement

In this experiment, the goal was to automatically generate a residential site plan that:

* Fits inside a predefined irregular property boundary.
* Preserves a protected tree.
* Complies with Seattle building regulations.
* Complies with Seattle tree protection requirements.
* Maximizes buildable carpet area.

The experiment uses a multi-agent workflow where one LLM generates python code and another LLM independently verifies compliance and constraints.

The Seattle Building Code and Seattle Site Plan Guidelines were provided as reference documents. These documents contain requirements related to building constraints and site plan preparation.

---

# Architecture

## Agent 1 — DeepSeek

Role: CAD Drawing Generator

Responsibilities:

* Generate Python code using the `ezdxf` library.
* Draw the site boundary.
* Draw the protected tree.
* Generate a house footprint.
* Attempt to satisfy Seattle code requirements.
* Maximize buildable area.

---

## Agent 2 — Kimi

Role: Constraint Verification Agent

Responsibilities:

1. Extract constraints from the specification.
2. Classify constraints as:

   * Logical Constraints
   * Semantic Constraints
3. Verify compliance.
4. Generate structured feedback.
5. Return only failed constraints.

---

# Workflow

```text
Specification
      ↓
DeepSeek (Generator)
      ↓
Python ezdxf Code
      ↓
Kimi (Verifier)
      ↓
Constraint Extraction
      ↓
Constraint Classification
      ↓
Verification
      ↓
Feedback
      ↓
DeepSeek Revision
      ↓
Repeat Until PASS
```

---

# Initial Prompt to Agent 1 (DeepSeek)

``` ROLE: CAD DRAWING GENERATOR
You are an expert mechanical CAD engineer.
Your task is to generate Python code using the ezdxf library that creates a DXF drawing satisfying ALL constraints.
IMPORTANT RULES
1.	Output ONLY Python code.
2.	Do NOT provide explanations.
3.	Use ezdxf only.
4.	Units are feet.
5.	The generated code must execute without syntax errors.
6.	Preserve all previously satisfied constraints.
7.	If verifier feedback is provided, modify only what is necessary to fix failed constraints.
8.	The DXF drawing is the final artifact. The geometry must satisfy the constraints.
ORIGINAL SPECIFICATION
Site Information
Coordinate System:
•	Units are feet.
•	Origin (0,0) is the southwest corner of the property.
•	Positive X points east.
•	Positive Y points north.
Assume that the front of the house is facing south.
Property Boundary:
P0 = (0,0)
P1 = (0,76)
P2 = (75,76)
P3 = (75,80)
P4 = (80,80)
P5 = (80,0)
P6 = (75,0)
P7 = (75,-8)
P8 = (35,-8)
P9 = (35,0)
P10 = (0,0)
The property boundary shall be drawn as a closed polyline.
Protected Tree:
•	Tree center = (77.5,78)
•	Tree diameter = 1 ft
•	Tree radius = 0.5 ft
•	The tree must be shown in the drawing.
•	The tree cannot be removed.
•	The proposed house footprint shall not intersect the tree protection area required by Seattle regulations.
House Requirements:
•	Create a single contiguous house footprint polygon.
•	The house footprint must lie entirely inside the property boundary.
•	The house footprint must comply with all applicable Seattle building and zoning setback requirements.
•	The house footprint must comply with all Seattle tree protection requirements.
•	The house footprint should maximize buildable carpet area while remaining fully compliant with all constraints.
•	The footprint should use a practical residential layout and avoid unnecessary geometric complexity.
•	The footprint must be represented as a closed polygon.
Drawing Requirements:
•	Draw the property boundary.
•	Draw the protected tree.
•	Draw the proposed house footprint.
•	Clearly separate site elements using appropriate DXF layers.
•	Include labels identifying:
o	Property Boundary
o	Protected Tree
o	Proposed House Footprint
I have attached the seattle building code and tree site plan by seattle government for you to follow and verify if all the requirements are satisfied.
Output only executable Python code using ezdxf.
```

---

# Initial Prompt to Agent 2 (Kimi)
```
ROLE: CAD CONSTRAINT VERIFICATION AGENT
You are an expert engineering reviewer responsible for validating whether a generated site plan satisfies the specification.
Your task is NOT to improve the drawing.
Your task is to:
1.	Extract constraints from the specification.
2.	Classify constraints as LOGICAL or SEMANTIC.
3.	Verify the generated solution.
4.	Produce structured feedback for failed constraints.
A constraint is:
LOGICAL:
●	Can be checked objectively.
●	Geometry
●	Coordinates
●	Dimensions
●	Setbacks
●	Areas
●	Layers
●	Labels
●	Polygon validity
●	Tree location
●	Boundary compliance
SEMANTIC:
●	Requires engineering judgement.
●	Practical residential layout.
●	Avoid unnecessary geometric complexity.
●	Maximizing usable area.
●	Drawing readability.
●	Design quality.
VERIFICATION PROCEDURE
STEP 1 — CONSTRAINT EXTRACTION
Extract all requirements from the specification.
Create atomic constraints.
Assign IDs:
C1
C2
C3
...
STEP 2 — CLASSIFICATION
For every constraint classify:
LOGICAL
or
SEMANTIC
STEP 3 — VERIFICATION
Evaluate every constraint independently.
For every constraint provide:
●	ID
●	Status
●	PASS or FAIL
●	Reason
Do NOT assume compliance.
Assume the drawing is incorrect until proven correct.
STEP 4 — FINAL DECISION
If every constraint passes:
OVERALL STATUS = PASS
Otherwise:
OVERALL STATUS = FAIL
STEP 5 — FEEDBACK GENERATION
Generate feedback ONLY for failed constraints.
For every failed constraint provide:
Constraint ID
Expected:
Actual:
Required Fix:
OUTPUT FORMAT
Return ONLY the following structure:
CONSTRAINTS
C1 ...
C2 ...
...
CLASSIFICATION
C1 LOGICAL
C2 SEMANTIC
...
VERIFICATION RESULTS
C1 PASS ...
C2 FAIL ...
OVERALL STATUS
PASS or FAIL
FEEDBACK FOR GENERATOR
Actual Constraint:
Expected:
Actual:
Required Fix:
INPUTS
=== ORIGINAL SPECIFICATION ===
Site Information
Coordinate System:
•	Units are feet.
•	Origin (0,0) is the southwest corner of the property.
•	Positive X points east.
•	Positive Y points north.
Assume that the front of the house is facing south.
Property Boundary:
P0 = (0,0)
P1 = (0,76)
P2 = (75,76)
P3 = (75,80)
P4 = (80,80)
P5 = (80,0)
P6 = (75,0)
P7 = (75,-8)
P8 = (35,-8)
P9 = (35,0)
P10 = (0,0)
The property boundary shall be drawn as a closed polyline.
Protected Tree:
•	Tree center = (77.5,78)
•	Tree diameter = 1 ft
•	Tree radius = 0.5 ft
•	The tree must be shown in the drawing.
•	The tree cannot be removed.
•	The proposed house footprint shall not intersect the tree protection area required by Seattle regulations.
House Requirements:
•	Create a single contiguous house footprint polygon.
•	The house footprint must lie entirely inside the property boundary.
•	The house footprint must comply with all applicable Seattle building and zoning setback requirements.
•	The house footprint must comply with all Seattle tree protection requirements.
•	The house footprint should maximize buildable carpet area while remaining fully compliant with all constraints.
•	The footprint should use a practical residential layout and avoid unnecessary geometric complexity.
•	The footprint must be represented as a closed polygon.
Drawing Requirements:
•	Draw the property boundary.
•	Draw the protected tree.
•	Draw the proposed house footprint.
•	Clearly separate site elements using appropriate DXF layers.
•	Include labels identifying:
o	Property Boundary
o	Protected Tree
o	Proposed House Footprint
I have attached the seattle building code and tree site plan by seattle government for you to follow and verify if all the requirements are satisfied.

=== GENERATED PYTHON CODE ===
import ezdxf
from ezdxf.math import Vec2

def create_drawing():
    doc = ezdxf.new('R2010')
    
    # Layers
    doc.layers.new('PROPERTY_BOUNDARY', dxfattribs={'color': 1, 'lineweight': 50})   # Red
    doc.layers.new('PROTECTED_TREE', dxfattribs={'color': 3, 'lineweight': 30})      # Green
    doc.layers.new('TREE_PROTECTION_ZONE', dxfattribs={'color': 4, 'lineweight': 20, 'linetype': 'DASHED'})  # Cyan, dashed
    doc.layers.new('HOUSE_FOOTPRINT', dxfattribs={'color': 5, 'lineweight': 40})      # Blue
    doc.layers.new('TEXT', dxfattribs={'color': 7, 'lineweight': 25})                 # White/Black
    
    msp = doc.modelspace()
    
    # Property boundary
    prop_points = [
        (0, 0), (0, 76), (75, 76), (75, 80), (80, 80),
        (80, 0), (75, 0), (75, -8), (35, -8), (35, 0), (0, 0)
    ]
    prop_poly = msp.add_lwpolyline(prop_points, dxfattribs={'layer': 'PROPERTY_BOUNDARY'})
    prop_poly.close()
    
    # Protected tree (trunk)
    tree_center = (77.5, 78)
    tree_radius = 0.5
    msp.add_circle(tree_center, tree_radius, dxfattribs={'layer': 'PROTECTED_TREE'})
    
    # Tree protection zone (Seattle SMC 25.11 - typical 6 ft radius for small tree)
    tpz_radius = 6.0
    msp.add_circle(tree_center, tpz_radius, dxfattribs={'layer': 'TREE_PROTECTION_ZONE'})
    
    # House footprint - maximized rectangle inside property and outside tree protection zone
    house_points = [
        (5, 5), (70, 5), (70, 71), (5, 71)
    ]
    house_poly = msp.add_lwpolyline(house_points, dxfattribs={'layer': 'HOUSE_FOOTPRINT'})
    house_poly.close()
    
    # Labels
    msp.add_text("Property Boundary", dxfattribs={'layer': 'TEXT', 'height': 2.5}) \
        .set_pos((40, 85), align='MIDDLE_CENTER')
    
    msp.add_text("Protected Tree", dxfattribs={'layer': 'TEXT', 'height': 2.5}) \
        .set_pos((77.5, 80.5), align='MIDDLE_CENTER')
    
    msp.add_text("Tree Protection Zone", dxfattribs={'layer': 'TEXT', 'height': 2.0}) \
        .set_pos((85, 78), align='MIDDLE_LEFT')
    
    msp.add_text("Proposed House Footprint", dxfattribs={'layer': 'TEXT', 'height': 2.5}) \
        .set_pos((37.5, 36), align='MIDDLE_CENTER')
    
    doc.saveas('site_plan.dxf')

if __name__ == '__main__':
    create_drawing()

Seattle building code and tree protection code are attached for reference.
OUTPUT ONLY THE FEEDBACK. FEEDBACK ONLY CONTAINS THE FAILED CONSTRAINT AND WHY IT FAILED. FEEDBACK SHOULD BE COPYABLE TEXT AND NOT A TABLE OR ANY OTHER FORMAT.
```
---

# Example Feedback Generated by Agent 2
```
FEEDBACK FOR GENERATOR
Failed Constraint: C10 - Maximize Buildable Carpet Area
Actual Constraint: The house footprint should maximize buildable carpet area while remaining fully compliant with all constraints.
Expected: The largest possible rectangular footprint within the buildable envelope defined by property boundaries, setbacks, and tree protection. For y in [20, 51] (satisfying 20-ft front and 25-ft rear setbacks), the property extends to x=80 on the east side, so with 5-ft side setback the maximum east extent is x=75. The maximum compliant rectangle is x=5 to x=75, y=20 to y=51, yielding 70 ft by 31 ft for 2170 sq ft.
Actual: House footprint is x=5 to x=70, y=20 to y=51, yielding 65 ft by 31 ft for 2092.5 sq ft. This is 77.5 sq ft less than the maximum possible compliant area.
Required Fix: Extend the house east edge from x=70 to x=75. This adds 5 ft of width across the full 31-ft depth, increasing area by 155 sq ft to achieve the maximum 2170 sq ft. The point (75, 51) is 25.02 ft from the tree center, well outside the 12-ft TPZ, and x=75 provides exactly 5 ft east setback from the x=80 property boundary for y<<76.
```

---

# Example Feedback Sent Back to Agent 1

```
The feedback loop used a minimal revision prompt:

Revise your previous solution based on the following feedback from the verifier.
FEEDBACK FOR GENERATOR
Failed Constraint: C10 - Maximize Buildable Carpet Area
Actual Constraint: The house footprint should maximize buildable carpet area while remaining fully compliant with all constraints.
Expected: The largest possible rectangular footprint within the buildable envelope defined by property boundaries, setbacks, and tree protection. For y in [20, 51] (satisfying 20-ft front and 25-ft rear setbacks), the property extends to x=80 on the east side, so with 5-ft side setback the maximum east extent is x=75. The maximum compliant rectangle is x=5 to x=75, y=20 to y=51, yielding 70 ft by 31 ft for 2170 sq ft.
Actual: House footprint is x=5 to x=70, y=20 to y=51, yielding 65 ft by 31 ft for 2092.5 sq ft. This is 77.5 sq ft less than the maximum possible compliant area.
Required Fix: Extend the house east edge from x=70 to x=75. This adds 5 ft of width across the full 31-ft depth, increasing area by 155 sq ft to achieve the maximum 2170 sq ft. The point (75, 51) is 25.02 ft from the tree center, well outside the 12-ft TPZ, and x=75 provides exactly 5 ft east setback from the x=80 property boundary for y<<76.
Requirements:
●	Fix only the failed constraints.
●	Preserve all previously satisfied constraints.
●	Output only the complete updated Python code.
```

---

# Findings

## Iterative Convergence

The system required **4 feedback iterations** before achieving a fully verified solution.

### Iteration 1

* Initial footprint generated.
* Multiple compliance failures identified.

### Iteration 2

* Tree protection issues corrected.
* Additional setback issues discovered.

### Iteration 3

* Setback violations resolved.
* Remaining optimization-related issues identified.

### Iteration 4

* Final footprint refined.
* All constraints satisfied.

---

## Observations

The experiment demonstrated that:

1. A generator LLM alone is insufficient for reliable regulatory compliance.
2. Constraint extraction significantly improves verification quality.
3. Separating generation and verification reduces hallucinated compliance.
4. Structured feedback enables iterative correction.
5. The approach resembles a neuro-symbolic workflow where:

   * DeepSeek acts as the generative component.
   * Kimi acts as the symbolic verifier.

---
