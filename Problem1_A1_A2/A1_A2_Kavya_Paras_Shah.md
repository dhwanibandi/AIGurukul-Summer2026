# Autonomous Multi-Agent System for House Footprint Optimization

## 1. Project Objective

This project focuses on developing an autonomous multi-agent workflow capable of generating the largest legally permissible house footprint for an irregular land parcel. The system is responsible for producing the coordinates of the building outline while ensuring full compliance with all geometric, environmental, and zoning regulations. Through an iterative collaboration process, the agents continuously refine the design until every constraint is satisfied.

## 2. System Architecture

The solution follows a collaborative **Design–Validation framework** consisting of two specialized AI agents connected through an automated Python orchestration layer.

| Agent | Function | Model | Primary Duties |
|---------|----------|---------|----------------|
| Design Agent | Footprint Creation | Claude Sonnet (Low Effort) | Interprets plot geometry and regulatory requirements, determines the optimal buildable region, and generates coordinate points defining the house boundary. |
| Validation Agent | Regulatory Review | Grok | Independently verifies the proposed footprint, performs geometric checks, detects violations, and returns corrective feedback. |

### Iterative Execution Process

**Step 1: Setup**

The system loads all fixed inputs, including property boundaries, setback regulations, protected environmental features, and operational instructions for both agents.

**Step 2: Design Generation**

The Design Agent proposes a closed polygon representing the external perimeter of the house.

**Step 3: Compliance Evaluation**

The Validation Agent assesses the proposed geometry against every applicable restriction and computes any violations.

**Step 4: Feedback Cycle**

- **Approved Result:** If no violations are detected, the workflow concludes and the footprint is accepted.
- **Rejected Result:** If any rule is breached, the Validation Agent returns detailed numerical feedback identifying the specific issue and its magnitude.

**Step 5: Revision**

The Design Agent incorporates the corrective observations and generates a revised footprint. The cycle repeats until a fully compliant solution is obtained.

## 3. Design Requirements and Constraints

### Protected Tree Buffer

A designated tree located at coordinates **(77.5, 86)** with a protection radius of **1.9 feet** must remain completely undisturbed. The house outline may not overlap or encroach upon this protected zone.

### Setback Compliance

The footprint must adhere to all prescribed zoning offsets:

- Front setback: **20 ft**
- Rear setback: **25 ft** (referenced from Y = 76)
- East side setback: **5 ft**
- West side setback: **5 ft**

### Access Consideration

The design must preserve a feasible entrance or driveway connection along the southern frontage of the property.

### Geometry Scope

The final DXF output must contain only a single closed polyline representing the exterior boundary of the house. Additional architectural features, landscaping elements, or property annotations are excluded from the generated drawing.

### Footprint Maximization

Subject to all constraints, the system aims to maximize the usable building area and produce the largest permissible house outline within the available construction envelope.

## 4. User Prompts

### Agent 1 Prompt
Generates Python code (using a DXF library) to create a 2D engineering drawing (.dxf/.dwg file) of a house plot layout. 
Component Tool/Technology: 2D Drawing File Format .dxf (DXF Python library — ezdxf) Code Generator LLM (YOU) Constraint Solver Z3 Solver (Microsoft Research) Constraint
 Source Seattle Building Code (SBC) — https://www.seattle.gov/sdci/codes/codes-we-enforce-(a-z)/building-code 
 Concrete Problem Specification (Plot Layout) - uploaded in the picture the circle in the picture is the tree
 Key Constraints: The tree cannot be cut (Seattle regulation: ~$50,000 fine + 6 months permit). 
 Offsets must follow Seattle Building Code (SBC). Maximize area coverage (land utilization). 
 The layout must include only the outer periphery of the house. 
 A single entry point (main door + parking) must be on the same side — per SBC fire safety norms.
 Generate Python (DXF) code to draw the outer periphery of a house on this plot, following all constraints
 tree is exactly in the middle of the 5*4 area 
 the plot in north is 75+5(HEIGHT OF 4) south is 35+40(DEPTH OF 8)+5 east is 80+8 (5 Towards left) west is 4(75 to right)+76+8 (35 to right) 
 ENTRY IS FROM SOUTH SIDE also give a diagram of the plot 
 the coordinates exactly will be in a line u can join 0,0 35,0 35,-8 75,-8 75,0 80,0 80,80 75,80 75,76 0,76 0,0

### Agent 2 Prompt
Verify the generated code against a set of constraints (both logical and semantic), using a solver (Z3), and provides feedback to A1.
ComponentTool/TechnologyConstraint SolverZ3 Solver (Microsoft Research)Constraint SourceSeattle Building Code (SBC) — well-documented, publicly available  https://www.seattle.gov/sdci/codes/codes-we-enforce-(a-z)/building-code
Concrete Problem Specification (Plot Layout)
uploaded the tree is exactly in the centre of the 4*5 area
Key Constraints:
The tree cannot be cut (Seattle regulation: ~$50,000 fine + 6 months permit).
Offsets must follow Seattle Building Code (SBC).
Maximize area coverage (land utilization).
The layout must include only the outer periphery of the house.
A single entry point (main door + parking) must be on the same side — per SBC fire safety norms.
Verify the generated code against SBC constraints and geometric rules.
If constraints are violated, output specific feedback: "Out of 3 constraints, you missed X — fix and regenerate."
Reference Resources
Seattle Building Code (SBC): Publicly documented, well-structured. Anupam provided a link in chat. (Prasad Gunjikar to compile all links in a shared Google Doc.)
DXF Python library: ezdxf (or similar package).
Z3 Solver: Available via pip — pip install z3-solver.
verify the code
only give me the checklist for constraints passed nothing else
the coordinates exactly will be in a line u can join 0,0 35,0 35,-8 75,-8 75,0 80,0 80,80 75,80 75,76 0,76 0,0
tree at 77.5,78

## 5.Iterations

### Iteration 1 (6/7 contraints passed)
- Claude calculated it without maximizing the area

- Output of A2
- [x] Plot coordinates exactly match the required linear boundary.
- [x] Tree located exactly at (77.5, 78).
- [x] Tree cannot be cut (SBC protection zone implemented).
- [x] Offsets follow Seattle Building Code (SBC).
- [x] Layout includes only the outer periphery of the house (no internal rooms).
- [x] Single entry point (main door + parking) located on the same side (South).
- Out of 7 constraints, you missed 1 (Maximize area coverage: the script hardcodes a simplified rectangular buildable envelope starting at y=0, ignoring the available plot space extending down to y=-8 between x=35 and x=75) — fix and regenerate

###Iteration 2 (6/7 paseed)
- Claude includes coordinates out of the Scope
- output of A2
- at first it said all constraints passed but as i ask it to redo and make sure it happens that one constraint fails
- *Initial*
- [x] Plot coordinates exactly match the required linear boundary.
- [x] Tree located exactly at (77.5, 78).
- [x] Tree cannot be cut (SBC protection zone implemented).
- [x] Offsets follow Seattle Building Code (SBC).
- [x] Maximize area coverage (buildable zone correctly utilizes the L-shape and southern notch area).
- [x] Layout includes only the outer periphery of the house (no internal rooms).
- [x] Single entry point (main door + parking) located on the same side (South).
- STATUS: PASS. All 7 constraints have been successfully met.
- *finally*
- Out of 7 constraints, you missed 1 (The buildable zone geometry extends outside the plot boundaries at the SE corner; the vertex at (78.5, -2) is out of bounds because the plot only exists for y ≥ 0 in the x=75 to 80 range) — fix and regenerate.

###Iteration 3
- claude now fails to follow SBC
- output of A2
- again at first it says all constraints passed and later asking it to do it again it gives out the one constraint failed
- *Initial*
- says all constraints passed
- *finally*
- Out of 7 constraints, you missed 1 (Offsets follow Seattle Building Code: The buildable polygon is placed exactly on the property line at y=0 for the x=0–35 and x=75–80 segments because BY_NF is set to 0.0. These are exterior south-facing lot boundaries, not interior edges, and must include a legal setback > 0m) — fix and regenerate.

### further Iterations
- after this it just goes in a loop where after iteration 3 problem in iteration 2 comes up and if we solve that the problem of iteration 3 comes up.
- A2 for a lot of times says that the constraints have passed and on rechecking it comes up with the constraints that fail.

## 6.Models
- Agent 1: Claude(Sonnet 4.6 Low)
- Agent 2: Gemini (Pro)

## 7.Observations
- Initially when I was uploading a clean diagram for the LLM to read it could not understand the plot properly. After I gave in the coordinates and it started working fine.
- The context window maybe getting full due to which the llm may be confused lleading to the loop.
- A2 a lot of times fails to detect the failed constraint and had to write a prompt : 'Recheck' for it to give out the failed constraint.
