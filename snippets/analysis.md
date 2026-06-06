# Analysis - GeniE SESAM Snippets

## Analysis Creation & Setup

```javascript
// Create the main analysis object
var ana = Analysis(true);  // true = create new analysis
ana.name = "JACKET_STATIC_ANALYSIS";

// Set SESTRA solver version
ana.useSestra10();

// Set this analysis as active
ana.setActive();

// Basic analysis parameters
ana.linearAnalysis = true;
ana.eigenvalueAnalysis = false;
ana.dynamicAnalysis = false;
```

## Activity Types

```javascript
// MeshActivity - generate finite element mesh
var meshAct = MeshActivity();
meshAct.name = "GENERATE_MESH";

// LinearAnalysis - perform linear static analysis
var linearAct = LinearAnalysis();
linearAct.name = "LINEAR_STATIC";

// LoadResultsActivity - apply loads as analysis results
var loadAct = LoadResultsActivity();
loadAct.name = "LOAD_RESULTS";

// EigenValueAnalysis - natural frequency and buckling
var eigenAct = EigenValueAnalysis();
eigenAct.name = "NATURAL_FREQUENCIES";
eigenAct.numberOfEigenvalues = 20;
eigenAct.eigenvalueType = "NaturalFrequency";  // or "Buckling"

// DynamicAnalysis - time-domain or frequency-domain
var dynAct = DynamicAnalysis();
dynAct.name = "WAVE_RESPONSE";
dynAct.analysisType = "FrequencyDomain";
dynAct.frequencyRange = [0.1, 2.0];  // Hz
dynAct.numberOfFrequencies = 100;
```

## Step Structure & Execution

```javascript
// Build analysis steps using method chaining
ana
    .step("Mesh Generation")
        .addActivity(MeshActivity())
    .step("Linear Static Analysis")
        .addActivity(LinearAnalysis())
    .step("Post-processing")
        .addActivity(LoadResultsActivity())
    .execute();

// Alternative: step-by-step execution
var meshStep = ana.step("Meshing");
meshStep.addActivity(MeshActivity());
meshStep.execute();

var analysisStep = ana.step("Solve");
analysisStep.addActivity(LinearAnalysis());
analysisStep.execute();

var resultsStep = ana.step("Results");
resultsStep.addActivity(LoadResultsActivity());
resultsStep.execute();
```

## Superelement Analysis

```javascript
// Define superelement boundaries
var anaSE = Analysis(true);
anaSE.name = "SUPERELEMENT_ANALYSIS";
anaSE.useSestra10();

// Create superelements from structural parts
var seDeck = SuperElement();
seDeck.name = "SE_DECK";
seDeck.retainedDoF = "all";  // Retain all boundary DOFs

var seJacket = SuperElement();
seJacket.name = "SE_JACKET";
seJacket.retainedDoF = "all";

// Superelement step
anaSE
    .step("Superelement Reduction")
        .addActivity(SuperElementAnalysis())
    .step("Assemble Global")
        .addActivity(LinearAnalysis())
    .execute();
```

## Tension/Compression Analysis

```javascript
// Tension-only member analysis (e.g., X-brace)
var anaTension = Analysis(true);
anaTension.name = "TENSION_ANALYSIS";
anaTension.useSestra10();

// Configure tension/compression options
anaTension.tensionOnlyMembers = true;
anaTension.compressionOnlyMembers = false;
anaTension.maxIterations = 50;
anaTension.convergenceTolerance = 0.01;

// Mark specific members as tension-only
var xBrace1 = StraightBeam(Point(0 m, 0 m, 10 m), Point(5 m, 5 m, 15 m));
xBrace1.beamType = Truss;  // Tension-only behavior
xBrace1.section = BarSection(0.002 m^2);
xBrace1.name = "X_BRACE_01";

anaTension
    .step("Tension Analysis")
        .addActivity(LinearAnalysis())
    .execute();
```

## Pile-Soil Analysis

```javascript
// Pile-soil interaction analysis
var anaPile = Analysis(true);
anaPile.name = "PILE_SOIL_ANALYSIS";
anaPile.useSestra10();

// Define soil springs along pile
var pileSoilAct = PileSoilActivity();
pileSoilAct.name = "PILE_SOIL_INTERACTION";
pileSoilAct.soilModel = "p-y";    // Lateral soil springs
pileSoilAct.tzModel = "t-z";      // Axial soil springs
pileSoilAct.qzModel = "q-z";      // End bearing springs

// Soil layer definition
pileSoilAct.addSoilLayer(0 m, -5 m, "soft_clay");
pileSoilAct.addSoilLayer(-5 m, -20 m, "medium_clay");
pileSoilAct.addSoilLayer(-20 m, -40 m, "dense_sand");
pileSoilAct.addSoilLayer(-40 m, -60 m, "very_dense_sand");

anaPile
    .step("Pile-Soil Setup")
        .addActivity(pileSoilAct)
    .step("Static Analysis")
        .addActivity(LinearAnalysis())
    .execute();
```

## Equipment Masses & LoadsAsMasses

```javascript
// Setup equipment for mass participation in dynamic analysis
var anaDyn = Analysis(true);
anaDyn.name = "DYNAMIC_MASS_ANALYSIS";
anaDyn.useSestra10();

// Convert loads to masses for dynamic analysis
anaDyn.loadsAsMasses = true;

// Equipment masses
var eqMasses = EquipmentMasses();
eqMasses.includeEquipmentMass = true;
eqMasses.includeAddedMass = true;
eqMasses.addedMassFactor = 1.0;  // Full added mass inclusion

// Add equipment to analysis
var eqGenset = PrismEquipment();
eqGenset.name = "GENERATOR_SET";
eqGenset.mass = 35000 kg;
eqGenset.cog = Point(0 m, 0 m, 1.2 m);
eqGenset.placeAtPoint(Point(15 m, 8 m, 30 m));

anaDyn
    .step("Setup Equipment Masses")
        .addActivity(eqMasses)
    .step("Eigenvalue Analysis")
        .addActivity(eigenAct)
    .execute();
```

## Full Analysis Workflow

```javascript
// Complete analysis setup for a jacket structure
var anaFull = Analysis(true);
anaFull.name = "FULL_JACKET_ANALYSIS";
anaFull.useSestra10();
anaFull.setActive();

// Step 1: Generate mesh
var meshActivity = MeshActivity();
meshActivity.elementType = mp2ndOrder;
meshActivity.meshSize = 1.0 m;

// Step 2: Linear static analysis with multiple load combinations
var linAnalysis = LinearAnalysis();
linAnalysis.addLoadCase(lcDead);
linAnalysis.addLoadCase(lcLive);
linAnalysis.addLoadCase(lcWind);
linAnalysis.addLoadCombination(combULS);
linAnalysis.addLoadCombination(combSLS);

// Step 3: Eigenvalue analysis for natural frequencies
var eigenActivity = EigenValueAnalysis();
eigenActivity.numberOfEigenvalues = 30;
eigenActivity.eigenvalueType = "NaturalFrequency";

// Step 4: Load results
var resultsActivity = LoadResultsActivity();

// Execute all steps
anaFull
    .step("Mesh Generation")
        .addActivity(meshActivity)
    .step("Static Analysis")
        .addActivity(linAnalysis)
    .step("Eigenvalue Analysis")
        .addActivity(eigenActivity)
    .step("Load Results")
        .addActivity(resultsActivity)
    .execute();

print("Analysis completed: " + anaFull.name);
print("   Steps executed: " + anaFull.numberOfSteps());
```

## Execute Strategies

```javascript
// Strategy 1: Full execution (all steps at once)
var ana1 = Analysis(true);
ana1.name = "FULL_EXECUTE";
ana1
    .step("Step1").addActivity(MeshActivity())
    .step("Step2").addActivity(LinearAnalysis())
    .execute();  // Runs all steps sequentially

// Strategy 2: Per-step execution with intermediate checks
var ana2 = Analysis(true);
ana2.name = "STEP_BY_STEP";

var s1 = ana2.step("Mesh");
s1.addActivity(MeshActivity());
s1.execute();
print("Meshing completed successfully");

// Inspect mesh quality before proceeding
var s2 = ana2.step("Solve");
s2.addActivity(LinearAnalysis());
s2.execute();
print("Linear analysis completed");

var s3 = ana2.step("Results");
s3.addActivity(LoadResultsActivity());
s3.execute();
print("Results loaded");
```