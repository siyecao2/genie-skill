# GenieRules Reference (DNV SESAM GeniE V8.8-08)

The `GenieRules` global object controls global modeling, meshing, and export behavior. Set rules before creating geometry.

## 1. GenieRules.Compatibility

```javascript
// Version check
print(GenieRules.Compatibility.version);
// Output: "8.8-08"

// Compatibility mode for older model files
GenieRules.Compatibility.compatibilityMode = "V8.0";
// Available modes: "V8.8", "V8.0", "V7.0"
// Use when importing legacy models with different default behaviors
```

## 2. GenieRules.Tolerances

Controls how GeniE determines if points/edges are coincident:

```javascript
// Enable tolerant modelling (recommended for large models)
GenieRules.Tolerances.useTolerantModelling = true;

// Angle tolerance: edges with dihedral angle below this are merged
GenieRules.Tolerances.angleTolerance = 5;  // degrees (default)

// Distance tolerance: points closer than this are considered coincident
GenieRules.Tolerances.distanceTolerance = 0.001;  // meters (default 1 mm)

// Use case: if importing from CAD where points may have sub-mm gaps
GenieRules.Tolerances.distanceTolerance = 0.005;  // 5 mm tolerance
```

**Warning:** Setting `distanceTolerance` too large may collapse small features. For detailed local models, use 0.001 m (1 mm).

## 3. GenieRules.Meshing

```javascript
// === Global element size (overrides entity-level if entity not set) ===
GenieRules.Meshing.elementSize = 0.5;  // meters

// === Element order ===
GenieRules.Meshing.elementType = "mp2ndOrder";  // quadratic (default, more accurate)
GenieRules.Meshing.elementType = "mp1stOrder";  // linear (faster, less accurate)
// 2nd order: 8-node quads, 3-node beams (cubic formulation)
// 1st order: 4-node quads, 2-node beams

// === Auto simplify before mesh ===
GenieRules.Meshing.autoSimplifyTopology = true;  // recommended
// Automatically calls SimplifyTopology() before mesh generation
// Set false if you want explicit control

// === Eliminate internal edges ===
GenieRules.Meshing.eliminateInternalEdges = true;
// Merges co-planar faces that share edges, improving mesh quality
// Set false if you need to keep internal edges for:
//   - separate plate thicknesses
//   - load application boundaries
//   - structural detailing

// === Mesh density rounding ===
GenieRules.Meshing.meshDensityRounded = true;
// Rounds mesh density to nearest whole number of elements per edge
// Set false for exact sizing control in local models

// === Uniform face parameterization ===
GenieRules.Meshing.useUniformizedFaceParameterization = true;
// Improves mesh regularity on curved surfaces
// Small performance cost, better results
```

### Recommended Meshing Configurations

**Global jacket model (coarse):**
```javascript
GenieRules.Meshing.elementSize = 1.0;
GenieRules.Meshing.elementType = "mp2ndOrder";
GenieRules.Meshing.autoSimplifyTopology = true;
GenieRules.Meshing.eliminateInternalEdges = true;
```

**Local joint model (fine):**
```javascript
GenieRules.Meshing.elementSize = 0.05;
GenieRules.Meshing.elementType = "mp2ndOrder";
GenieRules.Meshing.meshDensityRounded = false;
GenieRules.Meshing.eliminateInternalEdges = false;
```

## 4. GenieRules.BeamCreation

```javascript
// === Default curve offset ===
// Controls how beams curved along a GuideCurve are placed relative to the curve
GenieRules.BeamCreation.DefaultCurveOffset = "centre";  // beam center on curve
// Options: "centre", "top", "bottom", "left", "right"

// === Default curve orientation ===
GenieRules.BeamCreation.DefaultCurveOrientation = "vertical";
// Controls strong-axis orientation for curved beams
// "vertical": web remains vertical
// "normal": web follows curve normal

// === Beam creation rule ===
GenieRules.BeamCreation.BeamCreationRule = "standard";
// Controls how beam ends are trimmed at intersections
// "standard":  trim to connected faces
// "extended": extend to intersection plane
// "none":     no trim (use for explicit modeling)
```

## 5. GenieRules.Transformation

```javascript
// === Default connected copy behavior ===
GenieRules.Transformation.DefaultConnectedCopy = true;
// When true: ModelTransformer.copy() connects copies to originals
// When false: copies are independent (faster for large batch copies)
// Override per-operation:
//   mt.connected = false;  // overrides the rule
```

## 6. GenieRules.Export

```javascript
// === FEM export format ===
GenieRules.Export.femFormat = "Sestra_V7";
// Options: "Sestra_V7", "Sestra_V8"

// === Include suppressed entities ===
GenieRules.Export.includeSuppressed = false;
// Suppressed entities (visibility off) are excluded from export

// === Export coordinate precision ===
GenieRules.Export.coordinatePrecision = 6; // decimal places in .fem node coordinates

// === Export superelement options ===
GenieRules.Export.superelementType = "physical";
// "physical": retain physical DOFs only
// "mathematical": all DOFs (for Guyan reduction)
```

## 7. GenieRules.Display

```javascript
// Control visual feedback
GenieRules.Display.renderingMode = "shaded";    // "wireframe", "shaded", "hiddenLine"
GenieRules.Display.showSectionProfiles = true;   // render beam profiles
GenieRules.Display.showThickness = false;        // render plate thickness
GenieRules.Display.showFreeEdges = true;         // highlight unconnected edges
```

## 8. Complete Recommended Setup

```javascript
// === Standard setup for offshore structural models ===
// Place at the very start of every script

// Version
// GenieRules.Compatibility.compatibilityMode = "V8.8";

// Tolerances
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 5;
GenieRules.Tolerances.distanceTolerance = 0.001;

// Meshing
GenieRules.Meshing.autoSimplifyTopology = true;
GenieRules.Meshing.eliminateInternalEdges = true;
GenieRules.Meshing.elementType = "mp2ndOrder";
GenieRules.Meshing.elementSize = 0.5;
GenieRules.Meshing.useUniformizedFaceParameterization = true;

// Transformation
GenieRules.Transformation.DefaultConnectedCopy = false;

// Export
GenieRules.Export.femFormat = "Sestra_V8";
```
