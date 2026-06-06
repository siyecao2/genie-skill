# Common Errors & Troubleshooting (DNV SESAM GeniE V8.8-08)

## 1. Topology Errors

### Missing SimplifyTopology()
**Symptom:** Plates don't join, meshing produces gaps, analysis fails with unconnected nodes.
**Fix:**
```javascript
// Call after major geometry changes and before mesh generation
SimplifyTopology();

// Call after delete operations
Delete(someBeam);
SimplifyTopology();
```

### Unconnected Faces / Internal Edges
**Symptom:** Mesh is disconnected at plate boundaries, Sestra singularities.
**Cause:** Plates created from different guide planes with slight mismatches.
**Fix:**
```javascript
// Check: enable viewing of internal/free edges in GeniE
// Fix: re-snap to a common guide plane
var gp = GuidePlane(Point(0,0,0), Vector3d(0,0,1));
// All plates on this floor must use gp as reference
var p1 = FlatPlate(gp, [p1a, p1b, p1c, p1d]);
var p2 = FlatPlate(gp, [p2a, p2b, p2c, p2d]);
SimplifyTopology();
```

## 2. Missing Defaults

### No Default Section / Material / Thickness
**Symptom:** "No default section defined" or "No default material defined" when creating beams/plates.
**Fix:**
```javascript
// Always set at script start, before any geometry
Section.setDefault(Section("W21x44"));
Material.setDefault(Material("NV_36"));
Thickness.setDefault(0.02);  // 20 mm for plates

// Or assign explicitly to each entity
beam.section = Section("W21x44");
plate.thickness = 0.02;
plate.material = Material("NV_36");
```

## 3. Mesh Errors

### meshDensity Not Assigned or Zero
**Symptom:** "meshDensity must be greater than 0" or no mesh generated.
**Fix:**
```javascript
// Set mesh density on beams and plates
beam.meshDensity = 0.5;  // elements per meter (or size in meters)
plate.meshDensity = 0.3;

// Set globally before meshing
GenieRules.Meshing.elementSize = 0.5;
```

### Poor Mesh Quality
**Symptom:** Analysis warnings about element shape, distorted elements.
**Fix:**
```javascript
// Increase density on curved/slender regions
curvedBeam.meshDensity = 0.2;

// Check mesh before analysis: View > Mesh Visibility
// Uniformize face parameterization
GenieRules.Meshing.useUniformizedFaceParameterization = true;
```

## 4. Analysis Errors

### No LoadResultsActivity
**Symptom:** Analysis runs but no results are available for viewing/post-processing.
**Fix:**
```javascript
var la = LinearAnalysis();
var ma = MeshActivity();
var lra = LoadResultsActivity();  // <-- CRITICAL
lra.name = "Results";
la.addActivity(ma);
la.addActivity(lra);
la.run();
```

### Boundary Condition Issues
**Symptom:** Sestra reports rigid body modes, zero-pivot warnings, or non-convergence.
**Fix:**
```javascript
// Verify supports are correctly assigned
// 6-DOF fixity at jacket leg bottoms:
var sp = SupportPoint(Point(0, 0, -50));
sp.fixity = "fixed";  // all 6 DOFs fixed
// Or partial:
sp.fixity = "pinned";  // translations fixed, rotations free
sp.fixity = "TxTyTz";  // only translations fixed
```

## 5. ModelTransformer Errors

### Lost Connections After Copy
**Symptom:** Copied beams/plates don't connect to original structure.
**Cause:** `connected = false` used without re-connecting.
**Fix:**
```javascript
var mt = ModelTransformer();
// connected=true preserves connections at shared points
mt.connected = true;
var copies = mt.copy(mySet, Vector3d(0, 5, 0));
SimplifyTopology();  // re-join if needed
```

### ObjectNameMap Name Conflicts
**Symptom:** "Name already exists" errors on copy/mirror operations.
**Fix:**
```javascript
var mt = ModelTransformer();
var nameMap = new ObjectNameMap();
nameMap.add("Leg_*", "Leg_Copy_*");  // rename with prefix
mt.objectNameMap = nameMap;
var copies = mt.copy(mySet, Vector3d(10, 0, 0));
```

## 6. Explode / Join Problems

### Calling Explode on a Single Primitive
**Symptom:** `explode()` does nothing or produces unexpected result.
**Cause:** `explode()` only works on combined entities.
**Fix:**
```javascript
// explode is for Sets or combined entities
var mySet = Set("MyGroup");
var parts = mySet.explode();  // returns array of primitives
// Don't call explode() on a single Beam or Plate
```

### Join on Incompatible Faces
**Symptom:** Plates don't join, edges remain free.
**Cause:** Non-coplanar faces, gaps, or different guide planes.
**Fix:**
```javascript
// Ensure plates share the exact same guide plane
// Check gap with distance measurement
// Rebuild plates with precise point coordinates
SimplifyTopology();  // often resolves minor alignment issues
```

## 7. Load Errors

### Load Case Not Linked to Analysis
**Symptom:** Loads defined but not included in analysis results.
**Fix:**
```javascript
var lc1 = LoadCase("Gravity");
lc1.addLoad(VolumeLoad("selfWeight"));
var la = LinearAnalysis();
la.loadCases.add(lc1);  // <-- must add to analysis
```

### Equipment/Footprint Loading
**Symptom:** Equipment loads not transferring, footprint disregarded.
**Fix:**
```javascript
var eq = Equipment(Point(2, 3, 10));
eq.mass = 50000;     // kg
eq.name = "Compressor";
var eqLoad = EquipmentLoad(eq);
lc1.addLoad(eqLoad);   // adds weight force from equipment
```

## 8. Code Check Errors

### Missing Members Definition
**Symptom:** Code check finds no members to verify.
**Fix:**
```javascript
var cm = CapacityManager();
var cr = CapacityRun();
cr.rule = "NORSOK_N004";  // or "ISO_19902", "AISC_LRFD"
var cmember = CapacityMember(beamSet);
cr.addMember(cmember);
cm.addRun(cr);
cm.run();
```

### Wrong Standard Selected
**Symptom:** Code check returns unexpected/utilization values.
**Fix:** Verify `cr.rule` matches project specification:
- NORSOK N-004 → North Sea jacket
- ISO 19902 → International offshore
- AISC 360 → US onshore/offshore
- Eurocode 3 → European structures

## 9. Unit Errors

**Symptom:** Forces 1000x too large/small, stresses unrealistic.
**Fix:**
```javascript
// GeniE uses SI internally: m, kg, N, Pa
// Always be explicit:
var force = 1000e3;   // 1000 kN = 1,000,000 N (not 1000!)
var pressure = 1e6;   // 1 MPa = 1,000,000 Pa
var density = 7850;   // 7850 kg/m3

// When importing from other unit systems:
// 1 kip = 4448.22 N
// 1 ft = 0.3048 m
// 1 psi = 6894.76 Pa
// 1 ksi = 6,894,760 Pa
```
