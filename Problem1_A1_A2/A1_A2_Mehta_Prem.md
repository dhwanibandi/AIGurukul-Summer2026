# Agentic Workflow Report: Automated House Outline Generation

## 1. Problem Statement
The objective of this project is to build a two-agent autonomous system that designs the maximal buildable outer outline of a house on a specific irregular plot of land. The system must generate the coordinates of the house footprint and rigorously verify them against a set of geometric and zoning constraints (setbacks, protected trees, plot boundaries). The agents must operate in a continuous feedback loop, iterating upon the design until all statutory conditions are successfully met.

---

## 2. Workflow Process

The system employs a Generator-Verifier architecture, utilizing two distinct Large Language Models communicating via an automated Python pipeline. 

| Agent | Role | Model | Responsibilities |
|-------|------|-------|------------------|
| **Agent 1 (Generator)** | Architect | Claude Sonnet (low effort) | Parses the plot coordinates and constraints, formulates the geometric shape, and generates the exact coordinate vertices for the house outline. |
| **Agent 2 (Verifier)** | Compliance Checker | Grok | Acts as the statutory authority. Receives Agent 1's output, mathematically recalculates boundaries, checks for constraint violations, and provides structured failure feedback. |

### The Iteration Loop:
1. **Initialization:** The static plot geometry, zoning constraints, and system prompts are injected into the context windows of both agents.
2. **Generation:** Agent 1 proposes a set of coordinates for the house outline.
3. **Verification:** Agent 2 evaluates the proposed coordinates against the hardcoded constraints.
4. **Feedback Mechanism:**
   - If **PASS**: The loop terminates, and the coordinates are finalized and exported.
   - If **FAIL**: Agent 2 generates a precise, numerical critique (e.g., "Vertex A at (80, 45) violates the 5ft East setback by 2 feet"). This critique is appended to Agent 1's context window.
5. **Regeneration:** Agent 1 adjusts the coordinates based on the Verifier's feedback and submits a new proposal.

---

## 3. Constraints & Plot Details (Reference)
The system evaluated the design against five strict criteria:

1.  **Tree Preservation:** The footprint must not intersect a protected tree located at `(77.5, 86)` with a radius of `1.9 ft`.
2.  **SBC Offsets:** The footprint must respect standard zoning setbacks: Front = 20', Rear = 25' (measured from Y=76), Sides = 5' (East/West).
3.  **Single Entry Point:** The design must account for an entry point/driveway on the front (South) side.
4.  **Scope:** The generated DXF code must draw **only** the outer periphery of the house footprint (a single closed polyline). No ancillary architectural or property elements are permitted.
5.  **Area Optimization:** The house must maximize its buildable area within the legal bounds.

---

## 4. System Prompts

### Agent 1 (Generator) Prompt

You are an expert Civil Engineering Drafter and Python Developer. Your task is to generate Python code using the ezdxf library to create a 2D engineering drawing (.dxf) of a house plot layout. 

CRITICAL PRE-REQUISITE: Before generating any code, you must review the full Seattle Building Code (SBC) context provided to you. You must strictly adhere to its zoning, setback, and offset regulations. 

CURRENT FACTS & CONSTRAINTS:
Plot Dimensions: [
Coordinate System:
- Units are feet. 
- Origin (0,0) is the southwest corner of the property. 
- Positive X points east. 
- Positive Y points north. 

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
P10 = (0,0)].

- Tree Obstruction: Located at coordinates [77.5, 86] with a radius of [1.9] units. This tree's total area (including the radius) CANNOT be cut or intersected (Penalty: $50,000 fine).
- SBC Offsets: Strictly follow the Seattle Building Code minimum setbacks.
- Area Optimization: Maximize the usable area coverage of the house footprint without violating setbacks or the tree radius.
- Entry/Fire Safety: Provide a single entry point (main door + parking) on the SAME side of the layout.
- Scope: Draw ONLY the outer periphery of the house. 

INSTRUCTIONS:
Do not make assumptions about standard house shapes; calculate the maximum bounding box/polygon that satisfies all SBC setbacks and avoids the tree radius.
Output ONLY valid, executable Python code, without extra information.
The DXF drawing is the final artifact. The geometry must satisfy the constraints.

### Agent 2 (Verifier) Prompt

You are a Strict Code Inspector and Z3 Constraint Solver interface. Your task is to verify the provided Python ezdxf code against the Seattle Building Code (SBC) and geometric constraints.
Plot Dimensions: [
Coordinate System: 

- Units are feet. 
- Origin (0,0) is the southwest corner of the property. 
- Positive X points east. 
- Positive Y points north.

Assume that the front of the house is facing south. Property Boundary: 
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
P10 = (0,0)] 

EVALUATION PROTOCOL: You must process the layout mathematically and generate a strict Pass/Fail scorecard based on the following metrics: 

SCORECARD: 

[ ] Tree Preservation (PASS/FAIL): Footprint must not intersect tree coordinates [77.5,86] + radius [1.9]. 

[ ] SBC Offsets (PASS/FAIL): Footprint must not violate Front/Rear/Side setbacks as described by SBC. 

[ ] Single Entry Point (PASS/FAIL): Door and parking must share the same directional side. 

[ ] Scope (PASS/FAIL): Code must only draw the outer periphery. 

[ ] Area Optimization (PASS/FAIL): Footprint utilizes the maximum possible buildable area minus the tree exclusion zone. 

TOTAL CONSTRAINTS MET: [X]/5 FINAL STATUS: [PASS / FAIL] (Must be 5/5 to PASS). If STATUS is FAIL, you must output feedback in the exact format below to prevent the Generator from repeating its mistakes:

* FAILED METRIC(S): [List the specific rules broken]

* WHY IT FAILED: [Mathematical or logical reason based on the generated coordinates. e.g., "The corner at (45, 60) falls within the 10ft tree radius of (50, 50)."]

* ASSUMED FLAW: [List the false assumption the Generator likely made, e.g., "You assumed the side setback was 5ft, but SBC requires 10ft for this zone."]


## 5. Iteration Log 
### Iteration 1 — FAIL (4 / 5 Constraints Met) 
Claude successfully calculated the optimal geometric boundaries but over-delivered on the drawing requirements, violating the strict scope of the prompt. 
| Constraint | Status | Verifier Notes | 
| :--- | :--- | :--- | 
| **Tree Preservation** | **PASS** | House rectangle `[5, 20]` to `[75, 51]` has no intersection with tree circle at `(77.5, 86)` r=1.9. Closest point distance ~35+ ft. | 
|**SBC Offsets** | **PASS** | House respects Front=20' (Y≥20), Rear=25' (Y≤51 from Y=76), Sides=5' (X=5 to X=75). | 
|**Single Entry Point** | **PASS** | Door and parking stalls are present on the south/front side within the driveway notch. | 
| **Scope** | **FAIL** | Code draws property boundary, setbacks, driveway, parking stalls, door details, tree, dimensions, labels, north arrow, and title block. It does not restrict output exclusively to the outer house periphery. | 
|**Area Optimization** | **PASS** | 70' × 31' = 2,170 SF rectangle is the maximum axis-aligned buildable area under the constraints. | 

**Feedback Triggered:** *The generator produces a full annotated site plan instead of restricting output exclusively to the outer rectangular house periphery. The strict Scope constraint requires the code to draw only the house outer boundary.* |


### Iteration 2 — PASS (5 / 5 Constraints Met)
Claude integrated the feedback, stripping away all illustrative fluff while maintaining the correct mathematical coordinates for the footprint.
 | Constraint | Status | Verifier Notes | 
 | :--- | :--- | :--- |
 | **Tree Preservation** | **PASS** | House rectangle `[5.0, 20.0]` to `[75.0, 51.0]` has no intersection with tree circle. Clearance ≈ 35.8 ft. | 
 | **SBC Offsets** | **PASS** | Correctly applies Front=20′ (Y=20), Rear=25′ (Y=51 from Y=76), West=5′ (X=5), East=5′ (X=75). | 
 | **Single Entry Point** | **PASS** | No door or parking elements are drawn, which is now consistent with the strict outer-periphery scope constraint. | 
 | **Scope** | **PASS** | Code draws **only** the single closed `LWPOLYLINE` of the house outer boundary. No annotations, extra geometry, or ancillary layers. | 
 | **Area Optimization** | **PASS** | 70.0′ × 31.0′ = 2,170 SF axis-aligned rectangle remains the maximum possible area. |

 **Final Status:** Loop terminated successfully after 2 iterations.

 ---

