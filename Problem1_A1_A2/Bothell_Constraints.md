Markdown
### City of Bothell Invariant Rules

- **Global Property Scope:**
  - Lot_Width, Lot_Depth, Lot_Area (Width * Depth)

- **Setback Bounds (R-L Single Family):**
  - Front_Setback >= 20 feet
  - Rear_Setback >= 15 feet
  - Side_Setback_Left >= 5 feet
  - Side_Setback_Right >= 5 feet

- **Zoning Adjustment Flags:**
  - IF Zone == RM1 AND Element == Residential_Wall THEN Front_Setback >= 10 feet
  - IF Zone == RM1 AND Element == Garage_Door_Entrance THEN Front_Setback >= 20 feet

- **Permitted Encroachment Buffers:**
  - IF Element == Chimney OR Element == Bay_Window THEN Setback_Breach <= 2 feet
  - Structural_Clearance_To_Property_Line >= 3 feet

- **Massing and Surface Limits:**
  - Total_Structure_Footprint <= 0.35 * Lot_Area
  - Total_Impervious_Surface (Footprint + Driveway + Paved Patios) <= 0.70 * Lot_Area
  - Max_Building_Height <= 35 feet

- **Driveway Geometry Constraints:**
  - Driveway_Width_At_Property_Line >= 10 feet AND Driveway_Width_At_Property_Line <= 20 feet

- **Tree Keep-Out Model:**
  - Circle_Exclusion_Radius = Tree_Dripline_Radius + 1.0 foot
  - Distance(House_Vertices, Tree_Center) >= Circle_Exclusion_Radius