# Best Practices for GeniE JS Scripting (V8.8-08)

## 1. Script Organization

Structure your script in a consistent order. This makes debugging and re-use much easier:

```javascript
// ===== HEADER =====
// Project: North Sea Jacket
// Date: 2026-06-06
// Description: 4-leg jacket with topside

// ===== 1. GLOBAL RULES =====
GenieRules.Meshing.elementType = "mp2ndOrder";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 5;   // degrees
GenieRules.Tolerances.distanceTolerance = 0.01; // meters

// ===== 2. MATERIALS =====
var matSteel = Material("NV_36");

// ===== 3. SECTIONS & THICKNESSES =====
Section.setDefault(Section("W14x90"));
Material.setDefault(matSteel);
Thickness.setDefault(0.025);

// ===== 4. PARAMETERS =====
var legSpacing = 20;     // m between legs
var deckElevation = 25;  // m above seabed
var nBays = 4;

// ===== 5. GUIDE PLANES =====
var gpXZ_1 = GuidePlane(Point(0,0,0), Vector3d(0,1,0));
var gpXZ_2 = GuidePlane(Point(0,spacing,0), Vector3d(0,1,0));
// ... define all planes before geometry

// ===== 6. GEOMETRY =====
// 6.1 Foundation / legs
// 6.2 Braces
// 6.3 Deck structure
// 6.4 Appurtenances

// ===== 7. SETS & GROUPS =====
var legSet = Set("Legs");
var braceSet = Set("Braces");
var deckSet = Set("Deck");

// ===== 8. BOUNDARY CONDITIONS =====
var support = SupportPoint(Point(0,0,-50));
support.fixity = "fixed";

// ===== 9. LOADS =====
var lcGravity = LoadCase("Gravity");
lcGravity.addLoad(VolumeLoad("selfWeight"));

// ===== 10. ANALYSIS =====
SimplifyTopology();
var la = LinearAnalysis();
la.name = "InPlace";
// ... define analysis activities

// ===== 11. EXECUTION =====
la.run();
```

## 2. GuidePlane Design Strategy

### Plan Grid Resolution
```javascript
// BAD: many similar planes -> hard to maintain
var gp1 = GuidePlane(Point(0,0,10), Vector3d(0,0,1));
var gp2 = GuidePlane(Point(0,0,10.001), Vector3d(0,0,1));

// GOOD: reuse planes, define precisely
var gpDeck = GuidePlane(Point(0,0,deckElevation), Vector3d(0,0,1));
```

### Snap Mode
```javascript
// When creating plates that must join:
// All plates on the same elevation use the SAME GuidePlane object
var gpBulkhead = GuidePlane(Point(0,5,0), Vector3d(1,0,0));
var plate1 = FlatPlate(gpBulkhead, [p1,p2,p3,p4]);
var plate2 = FlatPlate(gpBulkhead, [p3,p4,p5,p6]); // shares edge p3-p4
```

## 3. ModelTransformer Efficiency

### Batch Operations
```javascript
// GOOD: copy large sets at once
var mt = ModelTransformer();
var allBracing = Set("Bracing");
var nameMap = new ObjectNameMap();
nameMap.add("*", "Leg2_*");
mt.objectNameMap = nameMap;
mt.connected = false;  // faster, then simplify
var leg2Braces = mt.copy(allBracing, Vector3d(0, legSpacing, 0));
SimplifyTopology();

// BAD: copy beam-by-beam
for (...) { mt.copy(singleBeam, offset); }
```

### Reuse ObjectNameMap
```javascript
var mt = ModelTransformer();
var nameMap = new ObjectNameMap();
nameMap.add("Leg1_*", "Leg2_*");
mt.objectNameMap = nameMap;

// Copy to Leg 2
var leg2 = mt.copy(leg1Set, Vector3d(0, spacing, 0));

// Update map for Leg 3
nameMap = new ObjectNameMap();
nameMap.add("Leg1_*", "Leg3_*");
mt.objectNameMap = nameMap;
var leg3 = mt.copy(leg1Set, Vector3d(2*spacing, 0, 0));
```

## 4. SimplifyTopology Strategy

```javascript
// Call at these points:
// 1. After importing geometry
// 2. After ModelTransformer operations (especially connected copies)
// 3. After deleting entities
// 4. Immediately before mesh generation

SimplifyTopology();

// Verification: check free edge count
// View > Display > Free Edges (should be zero at internal connections)
```

## 5. Mesh Strategy

### Coarse → Fine Verification
```javascript
// Phase 1: Coarse mesh for quick verification
GenieRules.Meshing.elementSize = 1.0;
// Run analysis, check:
//   - Support reactions ≈ applied loads
//   - No rigid body modes
//   - Displacements in reasonable range

// Phase 2: Refine for final analysis
GenieRules.Meshing.elementSize = 0.5;
// Re-run analysis for final results
```

### Per-Entity Control
```javascript
// Finer mesh at critical joints
jacketLeg.meshDensity = 0.3;     // dense at joints
deckBeam.meshDensity = 0.6;      // coarse at secondary
handrail.meshDensity = 1.0;      // very coarse at appurtenances
```

## 6. Naming Conventions

```javascript
// Structural hierarchy naming
// Legs: Leg_SW, Leg_SE, Leg_NW, Leg_NE
// Bracing: Brace_Horiz_Elev10_SE, Brace_Diag_Bay2_N
// Decks: Deck_Main, Deck_Cellar, Deck_Mezzanine
// Plates: Plate_DeckMain_Panel3, Plate_Bulkhead_Fr12

// Use Set management early
var legSW = Set("Leg_SW");
var legsAll = Set("Legs_All");
var bracesBay1 = Set("Braces_Bay1");
var deckMain = Set("Deck_Main");
var allStructure = Set("All_Structure");
```

## 7. Set Management

```javascript
// Organize components into logical groups as you create them
// This enables batch operations later

var allLegs = Set("All_Legs");
allLegs.add(legSW);
allLegs.add(legSE);
allLegs.add(legNW);
allLegs.add(legNE);

// Use sets for: section assignment, load application, code checking
var allPrimary = Set("Primary_Structure");
allPrimary.add(allLegs);
allPrimary.add(allBraces);
allPrimary.add(allDeckFraming);

// Batch section assignment
allPrimary.section = Section("W14x90");
```

## 8. Performance Tips

```javascript
// Disable connection tracking during large copies
mt.connected = false;

// Avoid over-meshing: set density only where needed
// Don't set meshDensity=0.1 on a 100m beam!

// Use coarse display during geometry building
// View > Display Level > Wireframe (faster than shaded)

// For repetitive patterns: ModelTransformer > manual creation
// 4 legs × 8 bays = 32 copies vs. 32 manual creations
```

## 9. Error Prevention Checklist

```
□ Set default section AND material at script top
□ Call SimplifyTopology() before meshing
□ Define LoadResultsActivity in each analysis
□ Verify all load cases are added to the analysis
□ Set mesh density on ALL structural entities
□ Check supports: at least one fixed/full-constraint point
□ Verify units: forces in N, lengths in m
□ Check for duplicate names before ModelTransformer operations
□ View free edges before running analysis
□ Run coarse mesh first, verify results, then refine
```
