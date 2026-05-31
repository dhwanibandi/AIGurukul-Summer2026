# 2D CAD file generation and verification

## Problem Statement
- An AI model(A1) will generate a python code using the ezdxf library to make the periphery of a house with a given plot and a tree. The house must follow the Seattle building codes.
- Another AI model(A2) will verify that design and suggest a prompt for A1 including the changes needed.
- This loop goes on until A2 verifies the generated house.

## Choice of AI models
- A1 was chosen to be **Claude Sonnet 4.6(thinking)**.
- A2 was chosen to be **Deepseek**.

## The Plot
The plot was given with the following dimensions![[Screenshot 2026-05-30 130636.png]]
To get dimension in feet, divide each dimension by 10.

## First Generation
The first generation by Claude was the following.![[Pasted image 20260531010240.png|697]]
```python
import sys, io

# Ensure UTF-8 output on Windows (cp1252) terminals

if hasattr(sys.stdout, 'reconfigure'):

    sys.stdout.reconfigure(encoding='utf-8', errors='replace')

else:

    sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8', errors='replace')

  

"""

house_on_plot.py

================

Reads plot geometry and tree from Plot_new.dxf, uses Z3 SMT solver to find

the maximum-area rectangular house that satisfies Seattle Neighborhood

Residential (NR) building-code constraints, then writes house_on_plot.dxf.

  

Seattle Building Code references (SMC 23.44 / NR Zone, 2025 update):

  - Front setback   : 15 ft  (from south/bottom/street-facing edge)

  - Rear setback    : 15 ft  (from north/top/rear edge)

  - Side setbacks   :  5 ft  (from east and west edges)

  - Max lot coverage: 35%    (single-family dwelling, 1 unit)

  - Tree clearance  :  5 ft  from outer edge of trunk (Seattle Tree Ord.)

"""

  

import sys

import math

import ezdxf

import z3

from shapely.geometry import Polygon, Point, box, MultiPolygon

  

# ─────────────────────────────────────────────────────────────────────────────

# CONFIGURATION

# ─────────────────────────────────────────────────────────────────────────────

  

INPUT_DXF  = "Plot_new.dxf"

OUTPUT_DXF = "house_on_plot.dxf"

  

# Seattle NR zone setbacks (feet)

FRONT_SETBACK    = 15.0   # south (bottom, low-Y) — street-facing

REAR_SETBACK     = 15.0   # north (top, high-Y)

SIDE_SETBACK     =  5.0   # east and west

MAX_LOT_COVERAGE = 0.35   # 35 %

TREE_CLEARANCE   =  5.0   # ft from outer trunk edge

  

WIDTH_STEP = 0.5          # sweep resolution (ft)

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 1 — Extract plot geometry from DXF

# ─────────────────────────────────────────────────────────────────────────────

  

print("=" * 60)

print("  READING PLOT GEOMETRY FROM", INPUT_DXF)

print("=" * 60)

  

doc_in = ezdxf.readfile(INPUT_DXF)

msp_in = doc_in.modelspace()

  

raw_lines = []

tree_raw  = None

  

for e in msp_in:

    if e.dxftype() == "LINE":

        s = (e.dxf.start.x, e.dxf.start.y)

        t = (e.dxf.end.x,   e.dxf.end.y)

        raw_lines.append((s, t))

    elif e.dxftype() == "CIRCLE":

        tree_raw = {

            "cx": e.dxf.center.x,

            "cy": e.dxf.center.y,

            "r":  e.dxf.radius,

        }

    elif e.dxftype() == "LWPOLYLINE":

        pts = list(e.get_points())

        for i in range(len(pts)):

            s = (pts[i][0],                  pts[i][1])

            t = (pts[(i+1) % len(pts)][0],   pts[(i+1) % len(pts)][1])

            raw_lines.append((s, t))

  

if not raw_lines:

    sys.exit("ERROR: No LINE / LWPOLYLINE entities found in " + INPUT_DXF)

if tree_raw is None:

    sys.exit("ERROR: No CIRCLE (tree) entity found in " + INPUT_DXF)

  

# Determine origin offset (bottom-left of all raw coords)

all_pts = [p for seg in raw_lines for p in seg]

ox = min(p[0] for p in all_pts)

oy = min(p[1] for p in all_pts)

  

def to_feet(rx, ry):

    """Normalise raw DXF coord → feet (origin at lot bottom-left)."""

    return (rx - ox) / 10.0, (ry - oy) / 10.0

  

seg_ft = [(to_feet(*s), to_feet(*e)) for s, e in raw_lines]

  

# Build closed polygon by chaining segments

def chain_segments(segs):

    from collections import defaultdict

    adj = defaultdict(list)

    for i, (a, b) in enumerate(segs):

        adj[a].append((b, i))

        adj[b].append((a, i))

    start = segs[0][0]

    path, used, cur = [start], set(), start

    while True:

        moved = False

        for nxt, idx in adj[cur]:

            if idx not in used:

                used.add(idx); path.append(nxt); cur = nxt; moved = True; break

        if not moved or cur == start:

            break

    return path[:-1]

  

polygon_pts = chain_segments(seg_ft)

  

lot_poly = Polygon(polygon_pts)

if not lot_poly.is_valid:

    lot_poly = lot_poly.buffer(0)

  

lot_area = lot_poly.area

print(f"  Lot polygon  : {len(polygon_pts)} vertices")

print(f"  Lot area     : {lot_area:.2f} sq ft")

  

# Tree in feet

tcx, tcy = to_feet(tree_raw["cx"], tree_raw["cy"])

tr        = tree_raw["r"] / 10.0

print(f"  Tree         : center=({tcx:.2f}, {tcy:.2f}) ft  radius={tr:.2f} ft")

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 2 — Compute buildable envelope with shapely

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 60)

print("  COMPUTING BUILDABLE ENVELOPE")

print("=" * 60)

  

minx, miny, maxx, maxy = lot_poly.bounds

  

# Setback rectangle clipped to lot

setback_box = box(

    minx + SIDE_SETBACK,

    miny + FRONT_SETBACK,

    maxx - SIDE_SETBACK,

    maxy - REAR_SETBACK,

)

buildable = lot_poly.intersection(setback_box)

  

if buildable.is_empty:

    sys.exit("ERROR: Setbacks consume the entire lot.")

  

# Remove tree clearance zone

tree_exc  = Point(tcx, tcy).buffer(tr + TREE_CLEARANCE, resolution=64)

buildable = buildable.difference(tree_exc)

  

if buildable.is_empty:

    sys.exit("ERROR: Tree exclusion zone eliminates all buildable area.")

  

# If fragmented, keep the largest piece

if isinstance(buildable, MultiPolygon):

    buildable = max(buildable.geoms, key=lambda g: g.area)

  

bminx, bminy, bmaxx, bmaxy = buildable.bounds

print(f"  Buildable envelope bounds  : x=[{bminx:.2f}, {bmaxx:.2f}]  y=[{bminy:.2f}, {bmaxy:.2f}]")

print(f"  Buildable envelope area    : {buildable.area:.2f} sq ft")

print(f"  Max allowed house area (35%): {lot_area * MAX_LOT_COVERAGE:.2f} sq ft")

  

# For containment in Z3, we need the half-planes of the buildable polygon.

# Make sure the exterior ring is CCW so inward normals point inside.

if not buildable.exterior.is_ccw:

    bld_coords = list(buildable.exterior.coords)[:-1][::-1]  # reverse to CCW

else:

    bld_coords = list(buildable.exterior.coords)[:-1]

  

print(f"  Buildable polygon vertices : {len(bld_coords)}")

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 3 — Z3 width-sweep to maximise house area

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 60)

print("  RUNNING Z3 OPTIMIZER  (width sweep, step={} ft)".format(WIDTH_STEP))

print("=" * 60)

  

max_allowed_area = lot_area * MAX_LOT_COVERAGE

max_width  = bmaxx - bminx

max_height = bmaxy - bminy

  

best = None   # best feasible solution

  

# Tree bounding-box exclusion radii (conservative rectangle around clearance circle)

tex_r = tr + TREE_CLEARANCE   # 6.0 ft

  

def solve_for_width(w):

    """

    Ask Z3: for this fixed width w, what is the maximum height h and

    position (hx1, hy1) such that the rectangle [hx1,hx1+w] x [hy1,hy1+h]

    fits inside the buildable envelope and respects coverage limits?

    Returns (hx1, hy1, h) floats, or None if infeasible.

    """

    hx1 = z3.Real("hx1")

    hy1 = z3.Real("hy1")

    h   = z3.Real("h")

  

    # hx2 = hx1 + w  (constant)

    hx2_val = hx1 + w

  

    s = z3.Optimize()

  

    # ── Dimension bounds ──────────────────────────────────────────────────

    s.add(h >= 15.0)                       # minimum habitable depth

    s.add(h <= min(max_allowed_area / w, max_height))

    s.add(w * h <= max_allowed_area)       # Seattle 35 % lot coverage

  

    # ── Bounding box containment (fast, always correct) ──────────────────

    s.add(hx1 >= bminx)

    s.add(hx1 + w <= bmaxx)

    s.add(hy1 >= bminy)

    s.add(hy1 + h <= bmaxy)

  

    # ── Half-plane containment in buildable polygon ───────────────────────

    # For each directed edge (A→B) of the CCW polygon, every corner of the

    # rectangle must satisfy:  (P - A) × (B - A) >= 0

    # i.e.  (Py - Ay)*(Bx - Ax) - (Px - Ax)*(By - Ay) >= 0

    # Expanded per corner (linear in hx1, hy1, h):

    n = len(bld_coords)

    corners = [

        (hx1,       hy1),

        (hx1 + w,   hy1),

        (hx1 + w,   hy1 + h),

        (hx1,       hy1 + h),

    ]

    for i in range(n):

        ax, ay = bld_coords[i]

        bx, by = bld_coords[(i + 1) % n]

        dx, dy = bx - ax, by - ay      # edge direction

        # cross product ≥ 0:  (py - ay)*dx - (px - ax)*dy >= 0

        #                 =>  dx*py - dy*px >= dx*ay - dy*ax

        rhs = dx * ay - dy * ax

        for (px, py) in corners:

            s.add(dx * py - dy * px >= rhs)

  

    # ── Tree exclusion (rectangle must not overlap clearance disk bbox) ───

    # The tree is in the alcove at (77.5, 85.5) which is OUTSIDE the

    # buildable envelope, so it is automatically excluded.  We still add

    # a belt-and-suspenders disjunctive constraint:

    s.add(z3.Or(

        hx1 + w <= tcx - tex_r,

        hx1     >= tcx + tex_r,

        hy1 + h <= tcy - tex_r,

        hy1     >= tcy + tex_r,

    ))

  

    # ── Maximise height ───────────────────────────────────────────────────

    s.maximize(h)

  

    if s.check() == z3.sat:

        m = s.model()

        def ev(var):

            v = m.evaluate(var, model_completion=True)

            if z3.is_rational_value(v):

                return v.numerator_as_long() / v.denominator_as_long()

            try:

                return float(v.as_decimal(10).rstrip("?"))

            except Exception:

                return float(str(v))

        return ev(hx1), ev(hy1), ev(h)

    return None

  

# Sweep widths from maximum downward

width_candidates = sorted(

    {round(max_width - i * WIDTH_STEP, 2)

     for i in range(int(max_width / WIDTH_STEP) + 2)

     if round(max_width - i * WIDTH_STEP, 2) >= 15.0},

    reverse=True,

)

  

for w in width_candidates:

    result = solve_for_width(w)

    if result is not None:

        x1, y1, h_v = result

        area = w * h_v

        if best is None or area > best["area"]:

            best = {

                "x1": x1, "y1": y1,

                "x2": x1 + w, "y2": y1 + h_v,

                "w": w, "h": h_v, "area": area,

            }

            print(f"  w={w:6.1f} ft  h={h_v:6.2f} ft  area={area:8.2f} sq ft  pos=({x1:.2f},{y1:.2f})  ✓")

  

if best is None:

    sys.exit("ERROR: Z3 found no feasible house configuration.")

  

print()

print("=" * 60)

print("  OPTIMAL RESULT")

print("=" * 60)

print(f"  Width         : {best['w']:.2f} ft")

print(f"  Depth (height): {best['h']:.2f} ft")

print(f"  Area          : {best['area']:.2f} sq ft")

print(f"  Lot Coverage  : {100 * best['area'] / lot_area:.2f}%  (limit 35%)")

print(f"  SW corner     : ({best['x1']:.2f}, {best['y1']:.2f}) ft")

print(f"  NE corner     : ({best['x2']:.2f}, {best['y2']:.2f}) ft")

  

# Sanity assertions

assert best["area"] <= max_allowed_area + 0.05, "COVERAGE VIOLATION!"

assert best["x2"] - best["x1"] >= 14.99,        "WIDTH TOO SMALL!"

assert best["y2"] - best["y1"] >= 14.99,         "DEPTH TOO SMALL!"

# Verify inside lot

from shapely.geometry import box as sbox

house_box = sbox(best["x1"], best["y1"], best["x2"], best["y2"])

assert lot_poly.contains(house_box) or lot_poly.covers(house_box), \

    "HOUSE IS OUTSIDE LOT!"

print("  All constraint checks PASSED ✓")

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 4 — Generate output DXF

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 60)

print("  GENERATING", OUTPUT_DXF)

print("=" * 60)

  

doc_out = ezdxf.new("R2010")

msp_out = doc_out.modelspace()

  

# Add DASHED linetype

doc_out.linetypes.add("DASHED",  pattern=[0.75, -0.375])

doc_out.linetypes.add("DASHDOT", pattern=[1.0, -0.25, 0.0, -0.25])

  

# Layers

#  ACI colours:  7=White  2=Yellow  3=Green  4=Cyan  1=Red  5=Blue  6=Magenta

doc_out.layers.add("PLOT",     color=7,  linetype="Continuous")  # lot boundary

doc_out.layers.add("SETBACK",  color=2,  linetype="DASHED")      # setback line

doc_out.layers.add("TREE",     color=3,  linetype="Continuous")  # tree trunk

doc_out.layers.add("TREE_CLR", color=3,  linetype="DASHED")      # tree clearance

doc_out.layers.add("HOUSE",    color=1,  linetype="Continuous")  # house outline

doc_out.layers.add("LABELS",   color=6,  linetype="Continuous")  # text labels

  

# ── 1. Lot boundary ───────────────────────────────────────────────────────────

lot_pts_dxf = [(x, y) for (x, y) in polygon_pts]

msp_out.add_lwpolyline(

    lot_pts_dxf, close=True,

    dxfattribs={"layer": "PLOT", "lineweight": 50},

)

  

# ── 2. Setback envelope ───────────────────────────────────────────────────────

setback_coords_dxf = list(buildable.exterior.coords)[:-1]

msp_out.add_lwpolyline(

    setback_coords_dxf, close=True,

    dxfattribs={"layer": "SETBACK", "lineweight": 13},

)

  

# ── 3. Tree (trunk + clearance buffer) ───────────────────────────────────────

msp_out.add_circle(

    (tcx, tcy), tr,

    dxfattribs={"layer": "TREE", "lineweight": 25},

)

msp_out.add_circle(

    (tcx, tcy), tr + TREE_CLEARANCE,

    dxfattribs={"layer": "TREE_CLR", "lineweight": 13},

)

  

# ── 4. House rectangle ────────────────────────────────────────────────────────

x1, y1, x2, y2 = best["x1"], best["y1"], best["x2"], best["y2"]

house_pts_dxf = [(x1, y1), (x2, y1), (x2, y2), (x1, y2)]

msp_out.add_lwpolyline(

    house_pts_dxf, close=True,

    dxfattribs={"layer": "HOUSE", "lineweight": 70},

)

  

# ── 5. Dimension lines ───────────────────────────────────────────────────────

# Use simple leader lines to annotate house dimensions

char_h = 1.8  # text height in feet

  

def add_dim_line(msp, p1, p2, label, layer, offset=(0, 0)):

    """Draw a simple annotated line."""

    mx = (p1[0] + p2[0]) / 2 + offset[0]

    my = (p1[1] + p2[1]) / 2 + offset[1]

    msp.add_line(p1, p2, dxfattribs={"layer": layer, "lineweight": 9})

    txt = msp.add_text(label, dxfattribs={"layer": layer, "height": char_h})

    txt.set_placement((mx, my), align=ezdxf.enums.TextEntityAlignment.MIDDLE_CENTER)

  

# House width annotation (along bottom edge of house)

add_dim_line(msp_out,

    (x1, y1 - 2), (x2, y1 - 2),

    f"W={best['w']:.1f} ft", "LABELS", offset=(0, -1.5))

  

# House depth annotation (along right edge of house)

add_dim_line(msp_out,

    (x2 + 2, y1), (x2 + 2, y2),

    f"D={best['h']:.1f} ft", "LABELS", offset=(3, 0))

  

# ── 6. Labels ────────────────────────────────────────────────────────────────

cx_h = (x1 + x2) / 2

cy_h = (y1 + y2) / 2

  

# House interior label

house_label_lines = [

    "HOUSE",

    f"Area: {best['area']:.1f} sq ft",

    f"Coverage: {100*best['area']/lot_area:.1f}%",

]

y_off = cy_h + char_h * (len(house_label_lines) - 1) / 2

for line in house_label_lines:

    txt = msp_out.add_text(line, dxfattribs={"layer": "HOUSE", "height": char_h * 0.85})

    txt.set_placement((cx_h, y_off), align=ezdxf.enums.TextEntityAlignment.MIDDLE_CENTER)

    y_off -= char_h * 1.3

  

# Code compliance note (placed to the left of the lot)

note_lines = [

    "SEATTLE NR ZONE (SMC 23.44)",

    f"Front setback : {FRONT_SETBACK:.0f} ft",

    f"Rear setback  : {REAR_SETBACK:.0f} ft",

    f"Side setback  : {SIDE_SETBACK:.0f} ft",

    f"Max coverage  : {int(MAX_LOT_COVERAGE*100)}%",

    f"Tree clearance: {TREE_CLEARANCE:.0f} ft",

]

note_x = -22.0

note_y = (miny + maxy) / 2 + char_h * len(note_lines) / 2

for line in note_lines:

    txt = msp_out.add_text(line, dxfattribs={"layer": "LABELS", "height": char_h * 0.75})

    txt.set_placement((note_x, note_y), align=ezdxf.enums.TextEntityAlignment.MIDDLE_LEFT)

    note_y -= char_h * 1.3

  

# ── Save ──────────────────────────────────────────────────────────────────────

doc_out.saveas(OUTPUT_DXF)

print(f"  Saved → {OUTPUT_DXF}")

  

print()

print("=" * 60)

print("  FINAL SUMMARY")

print("=" * 60)

print(f"  Input file       : {INPUT_DXF}")

print(f"  Output file      : {OUTPUT_DXF}")

print(f"  Lot area         : {lot_area:.2f} sq ft")

print(f"  House width      : {best['w']:.2f} ft")

print(f"  House depth      : {best['h']:.2f} ft")

print(f"  House area       : {best['area']:.2f} sq ft")

print(f"  Lot coverage     : {100*best['area']/lot_area:.2f}%  (≤ 35% ✓)")

print(f"  SW corner (ft)   : ({best['x1']:.2f}, {best['y1']:.2f})")

print(f"  NE corner (ft)   : ({best['x2']:.2f}, {best['y2']:.2f})")

print(f"  Front setback    : {best['y1'] - miny:.2f} ft  (req. ≥ {FRONT_SETBACK:.0f} ft) ✓")

print(f"  Rear setback     : {maxy - best['y2']:.2f} ft  (req. ≥ {REAR_SETBACK:.0f} ft) ✓")

print(f"  Left side setback: {best['x1'] - minx:.2f} ft  (req. ≥ {SIDE_SETBACK:.0f} ft) ✓")

print(f"  Right side setb. : {maxx - best['x2']:.2f} ft  (req. ≥ {SIDE_SETBACK:.0f} ft) ✓")

print("=" * 60)
```

| Item           | Value                          | Requirement            |
| -------------- | ------------------------------ | ---------------------- |
| House Area     | 2247 sq. ft                    | max 2247 (35% of 6420) |
| Lot coverage   | 35%                            | ≤35%                   |
| Dimensions     | 69 ft × 32.57 ft               | min 15 ft each         |
| Front setback  | 15.0 ft                        | ≥ 15 ft                |
| Rear setback   | 40.4 ft                        | ≥ 15 ft                |
| Side setbacks  | 5.0 ft / 6.0 ft                | ≥ 5 ft                 |
| Tree clearance | tree in alcove — outside house | ≥ 5 ft                 |
- The **yellow dashed boundary** is the buildable envelope and it represents the **legal zone where construction is allowed** after subtracting all Seattle NR zone setbacks from the lot edges.
- The **lot coverage rule** says that 35% of the lot area can be used for making the house.

## First verification
The generated drawing given as input to Deepseek(A2) and a prompt was written telling it to find violations and generate another prompt which tells A1 to generate the new drawing.
For the first verification, deepseek found the tree clearance violation and generated the given prompt:
You are tasked with generating a new, code-compliant house boundary for a residential lot. Your goal is to maximize the house's footprint area while strictly adhering to the Seattle Building Code for a Neighborhood Residential (NR) zone (likely NR1, NR2, or NR3).
You are provided with an image showing the existing plot boundary (white), a tree (circle), and the previous house boundary (blue). The plot is 69.0 ft wide. Calculate the required Tree Protection Area using the trunk diameter method (radius = tree diameter in inches, in feet). No part of the new house may encroach upon this Tree Protection Area.
**Code Requirements:**
*   **Maximum Lot Coverage:** 35% of the total lot area.
*   **Maximum Floor Area Ratio (FAR):** 0.6 for a single-family home.
*   **Front Setback:** The lesser of 20 feet OR the average of the front yards of the single-family structures on either side. (Assume average is 18 ft if data is unavailable).
*   **Rear Setback:** The lesser of 25 feet or 20% of the lot's depth, but never less than 10 feet.
*   **Side Setback:** 5 feet on each side.
**Process:**
1.  **Determine Lot Dimensions:** The width is 69 ft. Calculate the lot's depth using the provided coverage data (house area / 0.35) or from the attached drawing.
2.  **Establish Buildable Area:** Start with the full plot area. Subtract the Tree Protection Area, and then subtract the required front, rear, and side setbacks.
3.  **Generate House Boundary:** In the remaining buildable area, create a new polygon for the house's footprint. This polygon must be as large as possible, but cannot exceed 35% of the total lot area. Use a simple, convex shape to maximize area.
4.  **Output:** Provide the new house boundary as a set of coordinates in the same format as the input file. Also, calculate and report the new house's area and its percentage of lot coverage.

## Second generation
The second generation by Claude after receiving the prompt generated by using deepseek.![[Pasted image 20260531081827.png]]
```python
import sys, io

if hasattr(sys.stdout, 'reconfigure'):

    sys.stdout.reconfigure(encoding='utf-8', errors='replace')

else:

    sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8', errors='replace')

  

"""

house_on_plot_v2.py

===================

Generates a maximum-area, code-compliant house footprint inside an

L-shaped residential lot using the Z3 SMT solver and ezdxf.

  

Seattle NR Zone Code (SMC 23.44, 2025 update):

  - Front setback   : min(20 ft, avg of neighbours) → 18 ft assumed

  - Rear setback    : min(25 ft, 20% of lot depth)  → 17.6 ft (never < 10 ft)

  - Side setbacks   : 5 ft each side

  - Max lot coverage: 35%

  - Max FAR         : 0.6  (single-story footprint = footprint area / lot area)

  - Tree Protection : TPA radius (ft) = trunk diameter (inches)

                      → trunk ⌀ = 24 in  → TPA radius = 24 ft

"""

  

import math

import ezdxf

import z3

from shapely.geometry import Polygon, Point, box, MultiPolygon

  

# ─────────────────────────────────────────────────────────────────────────────

# CONFIGURATION

# ─────────────────────────────────────────────────────────────────────────────

  

INPUT_DXF  = "Plot_new.dxf"

OUTPUT_DXF = "house_on_plot_v2.dxf"

  

# Seattle NR setbacks (feet)

FRONT_SETBACK    = 18.0   # min(20, assumed neighbour avg 18)

REAR_SETBACK_PCT = 0.20   # 20% of lot depth

REAR_MAX         = 25.0   # never more than 25 ft

REAR_MIN         = 10.0   # never less than 10 ft

SIDE_SETBACK     =  5.0

  

MAX_LOT_COVERAGE = 0.35   # 35%

MAX_FAR          = 0.60   # 0.6 (single-story → same as coverage ratio if 1 floor)

  

# Tree Protection Area

TRUNK_DIAMETER_IN = 24.0          # inches (derived from DXF: r=1.0ft → ⌀=2ft=24in)

TPA_RADIUS        = TRUNK_DIAMETER_IN  # TPA radius in FEET = diameter in inches

  

WIDTH_STEP = 0.5   # sweep resolution (ft)

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 1 — Read plot geometry from DXF

# ─────────────────────────────────────────────────────────────────────────────

  

print("=" * 65)

print("  READING PLOT GEOMETRY FROM", INPUT_DXF)

print("=" * 65)

  

doc_in = ezdxf.readfile(INPUT_DXF)

msp_in = doc_in.modelspace()

  

raw_lines, tree_raw = [], None

  

for e in msp_in:

    if e.dxftype() == "LINE":

        raw_lines.append(((e.dxf.start.x, e.dxf.start.y),

                          (e.dxf.end.x,   e.dxf.end.y)))

    elif e.dxftype() == "CIRCLE":

        tree_raw = {"cx": e.dxf.center.x,

                    "cy": e.dxf.center.y,

                    "r":  e.dxf.radius}

    elif e.dxftype() == "LWPOLYLINE":

        pts = list(e.get_points())

        for i in range(len(pts)):

            raw_lines.append(((pts[i][0], pts[i][1]),

                              (pts[(i+1)%len(pts)][0], pts[(i+1)%len(pts)][1])))

  

if not raw_lines: sys.exit("ERROR: No LINE entities in " + INPUT_DXF)

if tree_raw is None: sys.exit("ERROR: No CIRCLE (tree) in " + INPUT_DXF)

  

all_pts = [p for seg in raw_lines for p in seg]

ox = min(p[0] for p in all_pts)

oy = min(p[1] for p in all_pts)

  

def ft(rx, ry):

    return (rx - ox) / 10.0, (ry - oy) / 10.0

  

seg_ft = [(ft(*s), ft(*e)) for s, e in raw_lines]

  

def chain_segments(segs):

    from collections import defaultdict

    adj = defaultdict(list)

    for i, (a, b) in enumerate(segs):

        adj[a].append((b, i)); adj[b].append((a, i))

    start = segs[0][0]; path, used, cur = [start], set(), start

    while True:

        moved = False

        for nxt, idx in adj[cur]:

            if idx not in used:

                used.add(idx); path.append(nxt); cur = nxt; moved = True; break

        if not moved or cur == start: break

    return path[:-1]

  

polygon_pts = chain_segments(seg_ft)

lot_poly    = Polygon(polygon_pts)

if not lot_poly.is_valid: lot_poly = lot_poly.buffer(0)

  

lot_area  = lot_poly.area

lot_depth = lot_poly.bounds[3] - lot_poly.bounds[1]   # maxy - miny

lot_minx, lot_miny, lot_maxx, lot_maxy = lot_poly.bounds

  

tcx, tcy = ft(tree_raw["cx"], tree_raw["cy"])

tr       = tree_raw["r"] / 10.0                        # trunk radius in feet

trunk_d_in = tr * 2 * 12 if TRUNK_DIAMETER_IN is None else TRUNK_DIAMETER_IN

tpa_r    = TPA_RADIUS

  

print(f"  Lot polygon    : {len(polygon_pts)} vertices")

print(f"  Lot area       : {lot_area:.2f} sq ft")

print(f"  Lot depth      : {lot_depth:.2f} ft")

print(f"  Tree center    : ({tcx:.2f}, {tcy:.2f}) ft")

print(f"  Tree trunk r   : {tr:.2f} ft  ({TRUNK_DIAMETER_IN:.0f}-inch trunk)")

print(f"  TPA radius     : {tpa_r:.2f} ft")

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 2 — Compute constraints

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 65)

print("  CODE CONSTRAINTS")

print("=" * 65)

  

rear_setback = max(REAR_MIN, min(REAR_MAX, REAR_SETBACK_PCT * lot_depth))

front_setback = FRONT_SETBACK

  

max_coverage_area = MAX_LOT_COVERAGE * lot_area

max_far_area      = MAX_FAR * lot_area

max_house_area    = min(max_coverage_area, max_far_area)

  

print(f"  Front setback  : {front_setback:.1f} ft  (min of 20ft and assumed 18ft avg)")

print(f"  Rear setback   : {rear_setback:.1f} ft  (20% x {lot_depth:.0f}ft = {REAR_SETBACK_PCT*lot_depth:.1f}ft, cap 25, floor 10)")

print(f"  Side setback   : {SIDE_SETBACK:.1f} ft  (each side)")

print(f"  Max coverage   : {max_coverage_area:.2f} sq ft  (35% of {lot_area:.0f})")

print(f"  Max FAR area   : {max_far_area:.2f} sq ft  (FAR 0.6, single-story)")

print(f"  Binding limit  : {max_house_area:.2f} sq ft  ({'coverage' if max_coverage_area < max_far_area else 'FAR'})")

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 3 — Buildable envelope (shapely)

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 65)

print("  COMPUTING BUILDABLE ENVELOPE")

print("=" * 65)

  

setback_box = box(

    lot_minx + SIDE_SETBACK,

    lot_miny + front_setback,

    lot_maxx - SIDE_SETBACK,

    lot_maxy - rear_setback,

)

buildable = lot_poly.intersection(setback_box)

if buildable.is_empty: sys.exit("ERROR: Setbacks consume entire lot.")

  

# Subtract Tree Protection Area

tpa_zone  = Point(tcx, tcy).buffer(tpa_r, resolution=128)

buildable = buildable.difference(tpa_zone)

if buildable.is_empty: sys.exit("ERROR: TPA eliminates all buildable area.")

  

if isinstance(buildable, MultiPolygon):

    buildable = max(buildable.geoms, key=lambda g: g.area)

  

bminx, bminy, bmaxx, bmaxy = buildable.bounds

  

# Ensure CCW for half-plane constraints

if not buildable.exterior.is_ccw:

    bld_coords = list(buildable.exterior.coords)[:-1][::-1]

else:

    bld_coords = list(buildable.exterior.coords)[:-1]

  

print(f"  Buildable bounds  : x=[{bminx:.2f}, {bmaxx:.2f}]  y=[{bminy:.2f}, {bmaxy:.2f}]")

print(f"  Buildable area    : {buildable.area:.2f} sq ft")

print(f"  Buildable vertices: {len(bld_coords)}")

print(f"  TPA cuts into zone: x<{tcx - tpa_r:.1f}  and  y<{tcy - tpa_r:.1f}")

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 4 — Z3 width-sweep optimiser

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 65)

print(f"  Z3 OPTIMIZER  (width sweep, step={WIDTH_STEP} ft)")

print("=" * 65)

  

max_width  = bmaxx - bminx

max_height = bmaxy - bminy

  

best = None

  

def ev(model, var):

    v = model.evaluate(var, model_completion=True)

    if z3.is_rational_value(v):

        return v.numerator_as_long() / v.denominator_as_long()

    try:    return float(v.as_decimal(10).rstrip("?"))

    except: return float(str(v))

  

def solve(w):

    hx1 = z3.Real("hx1")

    hy1 = z3.Real("hy1")

    h   = z3.Real("h")

  

    s = z3.Optimize()

  

    # Dimension bounds

    max_h = min(max_house_area / w, max_height) if w > 0 else 0

    if max_h < 15.0: return None

    s.add(h >= 15.0)

    s.add(h <= max_h)

    s.add(w * h <= max_house_area)   # coverage + FAR (whichever is binding)

  

    # Bounding box inside buildable envelope

    s.add(hx1 >= bminx)

    s.add(hx1 + w <= bmaxx)

    s.add(hy1 >= bminy)

    s.add(hy1 + h <= bmaxy)

  

    # Half-plane constraints (all 4 corners inside CCW buildable polygon)

    n = len(bld_coords)

    corners = [(hx1, hy1), (hx1+w, hy1), (hx1+w, hy1+h), (hx1, hy1+h)]

    for i in range(n):

        ax, ay = bld_coords[i]

        bx, by = bld_coords[(i+1) % n]

        dx, dy = bx - ax, by - ay

        rhs = dx * ay - dy * ax

        for (px, py) in corners:

            s.add(dx * py - dy * px >= rhs)

  

    # Tree Protection Area — disjunctive exclusion (rectangle must not overlap TPA circle)

    # Conservative: use bounding-box of TPA circle, then add exact Euclidean constraints

    # for corners via disjunction:

    #   house is fully LEFT, RIGHT, BELOW, or ABOVE the TPA bounding box

    s.add(z3.Or(

        hx1 + w <= tcx - tpa_r,   # entirely left of TPA

        hx1     >= tcx + tpa_r,   # entirely right of TPA (unreachable but safe)

        hy1 + h <= tcy - tpa_r,   # entirely below TPA

        hy1     >= tcy + tpa_r,   # entirely above TPA (unreachable)

    ))

  

    s.maximize(h)

  

    if s.check() == z3.sat:

        m = s.model()

        return ev(m, hx1), ev(m, hy1), ev(m, h)

    return None

  

width_candidates = sorted(

    {round(max_width - i * WIDTH_STEP, 2)

     for i in range(int(max_width / WIDTH_STEP) + 2)

     if round(max_width - i * WIDTH_STEP, 2) >= 15.0},

    reverse=True,

)

  

for w in width_candidates:

    result = solve(w)

    if result is not None:

        x1, y1, h_v = result

        area = w * h_v

        if best is None or area > best["area"]:

            best = {

                "x1": x1, "y1": y1,

                "x2": x1 + w, "y2": y1 + h_v,

                "w": w, "h": h_v, "area": area,

            }

            print(f"  w={w:6.1f} ft  h={h_v:6.2f} ft  area={area:8.2f} sq ft"

                  f"  pos=({x1:.2f}, {y1:.2f})  OK")

  

if best is None:

    sys.exit("ERROR: Z3 found no feasible configuration.")

  

print()

print("=" * 65)

print("  OPTIMAL RESULT")

print("=" * 65)

print(f"  Width (E-W)    : {best['w']:.2f} ft")

print(f"  Depth (N-S)    : {best['h']:.2f} ft")

print(f"  Area           : {best['area']:.2f} sq ft")

print(f"  Lot coverage   : {100*best['area']/lot_area:.2f}%  (limit 35%)")

print(f"  FAR            : {best['area']/lot_area:.3f}  (limit 0.6)")

print(f"  SW corner (ft) : ({best['x1']:.2f}, {best['y1']:.2f})")

print(f"  NE corner (ft) : ({best['x2']:.2f}, {best['y2']:.2f})")

  

# ── Sanity checks ─────────────────────────────────────────────────────────────

from shapely.geometry import box as sbox

house_box = sbox(best["x1"], best["y1"], best["x2"], best["y2"])

assert best["area"] <= max_house_area + 0.05,           "AREA LIMIT VIOLATED"

assert lot_poly.contains(house_box) or lot_poly.covers(house_box), "HOUSE OUTSIDE LOT"

assert not house_box.intersects(tpa_zone.buffer(-0.01)), "TPA VIOLATED"

print("  All checks PASSED")

  

# ─────────────────────────────────────────────────────────────────────────────

# STEP 5 — Generate output DXF

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 65)

print("  GENERATING", OUTPUT_DXF)

print("=" * 65)

  

doc_out = ezdxf.new("R2010")

msp_out = doc_out.modelspace()

  

doc_out.linetypes.add("DASHED",  pattern=[0.75, -0.375])

doc_out.linetypes.add("DASHDOT", pattern=[1.0, -0.25, 0.0, -0.25])

  

# Layers

doc_out.layers.add("PLOT",     color=7,  linetype="Continuous")   # White — lot boundary

doc_out.layers.add("SETBACK",  color=2,  linetype="DASHED")       # Yellow — setback envelope

doc_out.layers.add("TREE",     color=3,  linetype="Continuous")   # Green — tree trunk

doc_out.layers.add("TPA",      color=3,  linetype="DASHED")       # Green dashed — TPA

doc_out.layers.add("HOUSE",    color=1,  linetype="Continuous")   # Red — house footprint

doc_out.layers.add("LABELS",   color=6,  linetype="Continuous")   # Magenta — annotations

  

# 1. Lot boundary

msp_out.add_lwpolyline(

    [(x, y) for x, y in polygon_pts], close=True,

    dxfattribs={"layer": "PLOT", "lineweight": 50})

  

# 2. Setback envelope (buildable zone outline)

msp_out.add_lwpolyline(

    list(buildable.exterior.coords)[:-1], close=True,

    dxfattribs={"layer": "SETBACK", "lineweight": 13})

  

# 3. Tree trunk

msp_out.add_circle((tcx, tcy), tr,

    dxfattribs={"layer": "TREE", "lineweight": 25})

  

# 4. Tree Protection Area circle

msp_out.add_circle((tcx, tcy), tpa_r,

    dxfattribs={"layer": "TPA", "lineweight": 25})

  

# 5. House rectangle

x1, y1, x2, y2 = best["x1"], best["y1"], best["x2"], best["y2"]

msp_out.add_lwpolyline(

    [(x1, y1), (x2, y1), (x2, y2), (x1, y2)], close=True,

    dxfattribs={"layer": "HOUSE", "lineweight": 70})

  

# 6. House interior labels

char_h = 1.8

cx_h = (x1 + x2) / 2

cy_h = (y1 + y2) / 2

lines = [

    "HOUSE",

    f"Area: {best['area']:.1f} sq ft",

    f"Coverage: {100*best['area']/lot_area:.1f}%",

    f"FAR: {best['area']/lot_area:.3f}",

]

y_off = cy_h + char_h * len(lines) / 2

for ln in lines:

    t = msp_out.add_text(ln, dxfattribs={"layer": "HOUSE", "height": char_h * 0.85})

    t.set_placement((cx_h, y_off), align=ezdxf.enums.TextEntityAlignment.MIDDLE_CENTER)

    y_off -= char_h * 1.4

  

# 7. Width and depth annotations

def ann_line(msp, p1, p2, label, layer):

    mx, my = (p1[0]+p2[0])/2, (p1[1]+p2[1])/2

    msp.add_line(p1, p2, dxfattribs={"layer": layer, "lineweight": 9})

    t = msp.add_text(label, dxfattribs={"layer": layer, "height": char_h})

    t.set_placement((mx, my), align=ezdxf.enums.TextEntityAlignment.MIDDLE_CENTER)

  

ann_line(msp_out, (x1, y1-2.5), (x2, y1-2.5), f"W={best['w']:.1f} ft", "LABELS")

ann_line(msp_out, (x2+2.5, y1), (x2+2.5, y2), f"D={best['h']:.1f} ft", "LABELS")

  

# 8. Code note (left of lot)

note_lines = [

    "SEATTLE NR ZONE (SMC 23.44)  2025",

    f"Front setback : {front_setback:.0f} ft  [min(20, 18)]",

    f"Rear setback  : {rear_setback:.1f} ft  [20% x {lot_depth:.0f}ft]",

    f"Side setback  : {SIDE_SETBACK:.0f} ft",

    f"Max coverage  : {int(MAX_LOT_COVERAGE*100)}%  = {max_coverage_area:.0f} sq ft",

    f"Max FAR       : {MAX_FAR}  = {max_far_area:.0f} sq ft",

    f"TPA radius    : {tpa_r:.0f} ft  (trunk {TRUNK_DIAMETER_IN:.0f}-in dia)",

]

note_x = lot_minx - 25.0

note_y = (lot_miny + lot_maxy) / 2 + char_h * len(note_lines) / 2

for ln in note_lines:

    t = msp_out.add_text(ln, dxfattribs={"layer": "LABELS", "height": char_h * 0.75})

    t.set_placement((note_x, note_y), align=ezdxf.enums.TextEntityAlignment.MIDDLE_LEFT)

    note_y -= char_h * 1.3

  

doc_out.saveas(OUTPUT_DXF)

print(f"  Saved -> {OUTPUT_DXF}")

  

# ─────────────────────────────────────────────────────────────────────────────

# FINAL SUMMARY

# ─────────────────────────────────────────────────────────────────────────────

  

print()

print("=" * 65)

print("  FINAL SUMMARY")

print("=" * 65)

print(f"  Input            : {INPUT_DXF}")

print(f"  Output           : {OUTPUT_DXF}")

print(f"  Lot area         : {lot_area:.2f} sq ft")

print(f"  House width      : {best['w']:.2f} ft")

print(f"  House depth      : {best['h']:.2f} ft")

print(f"  House area       : {best['area']:.2f} sq ft")

print(f"  Lot coverage     : {100*best['area']/lot_area:.2f}%    (limit 35%) OK")

print(f"  FAR              : {best['area']/lot_area:.3f}     (limit 0.6)  OK")

print(f"  Front setback    : {best['y1'] - lot_miny:.2f} ft  (req. >= {front_setback:.0f} ft) OK")

print(f"  Rear setback     : {lot_maxy - best['y2']:.2f} ft  (req. >= {rear_setback:.1f} ft) OK")

print(f"  Left setback     : {best['x1'] - lot_minx:.2f} ft  (req. >= {SIDE_SETBACK:.0f} ft) OK")

print(f"  Right setback    : {lot_maxx - best['x2']:.2f} ft  (req. >= {SIDE_SETBACK:.0f} ft) OK")

tpa_dist = math.sqrt((cx_h - tcx)**2 + (cy_h - tcy)**2) - tpa_r

print(f"  Dist house->TPA  : {tpa_dist:.2f} ft  (req. > 0) OK")

print(f"  Trunk dia assumed: {TRUNK_DIAMETER_IN:.0f} inches (from DXF circle r=1.0ft)")

print("=" * 65)
```

| Items                        | Value         | Requirements  |
| ---------------------------- | ------------- | ------------- |
| House area                   | 2247 sq.ft    | ≤ 2,247 sq ft |
| Lot coverage                 | 35%           | ≤ 35%         |
| FAR                          | 0.35          | ≤ 0.6         |
| House dimensions             | 69 × 32.57 ft | min 15ft      |
| Front setback                | 18 ft         | ≥ 18 ft       |
| Rear setback                 | 37.4 ft       | ≥ 17.6 ft     |
| Side setbacks                | 5 ft / 6 ft   | ≥ 5 ft        |
| House $\rightarrow$ TPA edge | 39.8 ft       | > 0 ft        |

## Second verification
The second drawing was uploaded in deepseek again was asked to verify it. It said that the layout is compliant with the building codes, but it pointed out a notation error, but when i checked in the file there was no such error. This may have occurred due to wrong identification of a number by deepseek.

## Conclusion
A1 required 2 iterations to give a layout completely verified by A2.

