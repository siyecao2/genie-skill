# Meshing - GeniE SESAM Snippets

## MeshDensity Creation & Assignment

```javascript
// Global mesh density
var mdGlobal = MeshDensity();
mdGlobal.name = "GLOBAL_DENSITY";
mdGlobal.elementSize = 1.0 m;
mdGlobal.setAsGlobal();

// Local mesh density for specific region
var mdLocal = MeshDensity();
mdLocal.name = "FINE_MESH_JOINTS";
mdLocal.elementSize = 0.3 m;

// Assign local density to a specific beam
var bJoint = StraightBeam(Point(5 m, 0 m, 10 m), Point(7 m, 0 m, 10 m));
bJoint.meshDensity = mdLocal;

// Assign to a plate
plateBottom.meshDensity = mdLocal;

// Circular refinement zone
var mdRefine = MeshDensity();
mdRefine.name = "HOLE_REFINEMENT";
mdRefine.elementSize = 0.1 m;
mdRefine.radius = 2 m;  // Refinement radius around feature
mdRefine.center = Point(10 m, 5 m, 20 m);
```

## Element Type Settings

```javascript
// First-order elements (linear interpolation, 4-node quad, 2-node beam)
var meshAct1 = MeshActivity();
meshAct1.elementType = mp1stOrder;  // Faster, less accurate

// Second-order elements (quadratic interpolation, 8-node quad, 3-node beam)
var meshAct2 = MeshActivity();
meshAct2.elementType = mp2ndOrder;  // More accurate, preferred for stress analysis

// Mixed - first order for beams, second order for plates
var meshActMixed = MeshActivity();
meshActMixed.elementType = mp1stOrder;
meshActMixed.plateElementType = mp2ndOrder;  // Plates use 2nd order

// Triangular elements option
var meshActTri = MeshActivity();
meshActTri.elementType = mp2ndOrder;
meshActTri.allowTriangularElements = true;
```

## meshDensityRounded & Face Parameterization

```javascript
// meshDensityRounded - round element size to nearest "nice" value
var mdRounded = MeshDensity();
mdRounded.elementSize = 0.47 m;
mdRounded.meshDensityRounded = true;  // Will round to 0.5 m

// useUniformizedFaceParameterization - consistent element sizing
var meshAct = MeshActivity();
meshAct.useUniformizedFaceParameterization = true;
// Ensures elements on opposite faces have matching sizes for
// better mesh quality in thin-walled structures

// Manual face parameterization control
var mdManual = MeshDensity();
mdManual.elementSize = 0.8 m;
mdManual.useUniformizedFaceParameterization = false;
mdManual.minElementSize = 0.2 m;
mdManual.maxElementSize = 1.5 m;
```

## Per-Element Mesh Density Assignment

```javascript
// Assign different densities to different structural parts
var denseAtConnection = MeshDensity();
denseAtConnection.elementSize = 0.15 m;
denseAtConnection.name = "CONNECTION_ZONE";

var mediumBody = MeshDensity();
mediumBody.elementSize = 0.5 m;
mediumBody.name = "MEDIUM_BODY";

var coarseOuter = MeshDensity();
coarseOuter.elementSize = 1.0 m;
coarseOuter.name = "COARSE_OUTER";

// Apply to beams by proximity to critical zones
var beams = getAllBeams();
for (var i = 0; i < beams.length; i++) {
    var b = beams[i];
    var distToJoint = distanceToNearestJoint(b);

    if (distToJoint < 2 m) {
        b.meshDensity = denseAtConnection;
    } else if (distToJoint < 8 m) {
        b.meshDensity = mediumBody;
    } else {
        b.meshDensity = coarseOuter;
    }
}

// Apply to individual plate sub-regions
var subPlates = mainPlate.explode(IndexedNameMask("SUB_"));
subPlates[0].meshDensity = denseAtConnection;   // Edge zone
subPlates[1].meshDensity = mediumBody;           // Transition
subPlates[2].meshDensity = coarseOuter;          // Interior
```

## Mesh Refinement Strategies

```javascript
// Automatic mesh refinement at loading points
var meshActRefined = MeshActivity();
meshActRefined.autoRefineAtLoads = true;
meshActRefined.autoRefineAtSupports = true;
meshActRefined.refinementLevels = 3;          // 3 levels of refinement
meshActRefined.refinementRatio = 0.5;         // Each level halves element size

// Manual refinement by defining refinement zones
var refineZone1 = RefinementZone();
refineZone1.center = Point(10 m, 5 m, 0 m);
refineZone1.radius = 3 m;
refineZone1.targetSize = 0.2 m;
refineZone1.name = "CRITICAL_JOINT";

var refineZone2 = RefinementZone();
refineZone2.center = Point(15 m, 8 m, 0 m);
refineZone2.radius = 2 m;
refineZone2.targetSize = 0.15 m;
refineZone2.name = "HOLE_ZONE";

meshActRefined.addRefinementZone(refineZone1);
meshActRefined.addRefinementZone(refineZone2);
```

## autoSimplifyTopology Settings

```javascript
// autoSimplifyTopology - automatic topology cleanup before meshing
var meshActClean = MeshActivity();
meshActClean.autoSimplifyTopology = true;
meshActClean.topologyTolerance = 0.01 m;      // Merge tolerance
meshActClean.removeSmallEdges = true;
meshActClean.minEdgeLength = 0.05 m;           // Edges shorter than this removed
meshActClean.removeSmallFaces = true;
meshActClean.minFaceArea = 0.001 m^2;          // Faces smaller than this removed

// Disable auto simplification for precise geometry
var meshActPrecise = MeshActivity();
meshActPrecise.autoSimplifyTopology = false;

// Manual topology simplification
simplifyTopology();  // Call the global simplify function
```

## Element Repair & Quality Improvements

```javascript
// Mesh quality settings
var meshActQuality = MeshActivity();
meshActQuality.elementType = mp2ndOrder;

// Quality criteria
meshActQuality.maxAspectRatio = 5.0;           // Max element aspect ratio
meshActQuality.minInteriorAngle = 20 deg;       // Min angle in quad/tri
meshActQuality.maxInteriorAngle = 160 deg;      // Max angle in quad/tri
meshActQuality.maxJacobianDistortion = 0.7;     // Min Jacobian quality (>0.7 good)
meshActQuality.maxWarpAngle = 10 deg;           // Max element warpage

// Auto-repair poor quality elements
meshActQuality.autoRepair = true;
meshActQuality.repairIterations = 5;
meshActQuality.repairMethod = "nodeSmoothing";  // or "edgeSwap", "combined"

// Manual quality check
var qualityReport = checkMeshQuality();
for (var i = 0; i < qualityReport.count(); i++) {
    var elem = qualityReport.element(i);
    if (elem.aspectRatio > 4.0) {
        print("Warning: Element " + elem.id + " aspect ratio = " + elem.aspectRatio);
    }
}
```

## Mesh Import & Export

```javascript
// ImportMeshFem - import FEM mesh from external source
var importedMesh = ImportMeshFem("imported_mesh.fem");
importedMesh.name = "IMPORTED_FEM";
// Supported formats: SESAM .FEM, Nastran .dat/.bdf, Abaqus .inp

// Import with options
var importOpts = ImportMeshOptions();
importOpts.mergeTolerance = 0.001 m;
importOpts.preserveNodeIds = true;
importOpts.importMaterials = true;
importOpts.importSections = true;
var importedMeshAdv = ImportMeshFem("structure_mesh.bdf", importOpts);

// ExportMeshFem - export mesh for external solvers
var exportOpts = ExportMeshOptions();
exportOpts.format = "SESAM";       // "SESAM", "Nastran", "Abaqus"
exportOpts.exportNodes = true;
exportOpts.exportElements = true;
exportOpts.exportMaterials = true;
exportOpts.exportSections = true;
exportOpts.exportLoads = true;
exportOpts.exportBoundaryConditions = true;

ExportMeshFem("analysis_mesh.fem", exportOpts);
print("Mesh exported to analysis_mesh.fem");

// Quick mesh export
ExportMeshFem("simple_export.fem");
```

## Complete Meshing Workflow

```javascript
// Setup comprehensive meshing
function setupMeshWorkflow() {
    // 1. Define global mesh density
    var mdGlobal = MeshDensity();
    mdGlobal.elementSize = 0.8 m;
    mdGlobal.setAsGlobal();

    // 2. Local refinement at joints
    var mdJoints = MeshDensity();
    mdJoints.elementSize = 0.25 m;

    // 3. Create mesh activity
    var meshAct = MeshActivity();
    meshAct.name = "FULL_MODEL_MESH";
    meshAct.elementType = mp2ndOrder;
    meshAct.autoSimplifyTopology = true;
    meshAct.useUniformizedFaceParameterization = true;

    // 4. Quality settings
    meshAct.maxAspectRatio = 4.0;
    meshAct.autoRepair = true;

    // 5. Assign local densities
    var allBeams = getAllBeams();
    for (var i = 0; i < allBeams.length; i++) {
        if (isNearJoint(allBeams[i])) {
            allBeams[i].meshDensity = mdJoints;
        }
    }

    return meshAct;
}

// Execute full meshing in analysis
var ana = Analysis(true);
ana.useSestra10();
ana
    .step("Meshing")
        .addActivity(setupMeshWorkflow())
    .step("Analysis")
        .addActivity(LinearAnalysis())
    .execute();
```