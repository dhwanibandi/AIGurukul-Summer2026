### City of Redmond Invariant Rules

- **Global Property Scope:**
  - Lot_Width, Lot_Depth, Lot_Area (Width * Depth)

- **Setback Bounds (R-4, R-5, R-6 Baseline):**
  - Front_Setback >= 10 feet
  - Rear_Setback >= 10 feet
  - Side_Setback_Left >= 5 feet
  - Side_Setback_Right >= 5 feet
  - IF Corner_Lot == True THEN Side_Street_Setback >= 10 feet
  - IF Abatting_Alley == True THEN Rear_Setback >= 4 feet

- **Permitted Encroachment Buffers:**
  - IF Element == Chimney OR Element == Bay_Window THEN Setback_Breach <= 2 feet
  - Structural_Clearance_To_Property_Line >= 3 feet

- **Massing and Surface Limits:**
  - Total_Structure_Footprint <= 0.35 * Lot_Area
  - Total_Impervious_Surface (Footprint + Driveway + Paved Patios) <= 0.75 * Lot_Area
  - Max_Building_Height <= 35 feet

- **Driveway Geometry Constraints:**
  - Driveway_Width_At_Property_Line >= 10 feet AND Driveway_Width_At_Property_Line <= 20 feet

- **Tree Keep-Out Model:**
  - Significant_Tree_DBH >= 6.0 inches
  - Circle_Exclusion_Radius = Significant_Tree_DBH * 1.0 foot 
  - Distance(House_Vertices, Tree_Center) >= Circle_Exclusion_Radius