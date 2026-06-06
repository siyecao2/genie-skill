# SESAM Multi-Module Workflow Reference

## 1. SESAM Suite Overview

DNV SESAM is an integrated structural engineering system for offshore/marine structures. The workflow flows from concept modeling through global analysis, detailed analysis, code checking, and results extraction.

```
Sesam Manager (Central Launcher)
    │
    ├── GeniE ────────── Concept modeling, FEM generation
    │
    ├── Sestra ───────── Linear FE solver (static, dynamic, eigenvalue)
    │
    ├── Splice ──────── Pile-soil interaction, superelement generation
    │
    ├── Framework ────── Batch processing (AVM - Analysis & Verification Module)
    │
    ├── Wadam ────────── Wave load analysis (diffraction/radiation)
    ├── Wajac ────────── Wave-in-deck / Morison-based wave loads
    │
    ├── Usfos ────────── Nonlinear pushover / collapse / accidental limit state
    │
    ├── HydroD ───────── Hydrodynamic analysis, stability
    │
    ├── Submod ───────── Submodeling from global results
    │
    ├── Xtract ───────── Results post-processing, viewing, code checking
    │
    └── Stofat ──────── Fatigue analysis
```

## 2. Export Paths from GeniE

### GeniE → Sestra (Linear Static/Dynamic)
```javascript
// Export FEM input file for Sestra solver
var exp = ExportFEM();
exp.fileName = "model.fem";
exp.export();

// Or define analysis in GeniE and run directly
var la = LinearAnalysis();
la.name = "Static_Analysis";
var ma = MeshActivity();
var lra = LoadResultsActivity();
lra.name = "LoadResults";
la.addActivity(ma);
la.addActivity(lra);
la.run();  // meshes, writes .fem, invokes Sestra, imports results
```

**Key concepts:**
- **Superelement analysis**: export as superelement, include for global model use
- **Submodel analysis**: extract displacements from global model, apply as BCs

### GeniE → Wadam (Wave Load Analysis)
```javascript
// Export mass model and panel model for Wadam
var expWadam = ExportWadam();
expWadam.fileName = "wadam_model";
expWadam.exportType = "panelModel";  // or "massModel"
expWadam.export();
```
**Wadam** computes 3D diffraction/radiation wave loads; **Wajac** uses Morison's equation for slender structures.

### GeniE → Usfos (Nonlinear Pushover)
```javascript
var expUsfos = ExportUsfos();
expUsfos.fileName = "usfos_model";
expUsfos.includeHydrodynamicLoads = true;
expUsfos.export();
```
Usfos performs:
- Progressive collapse (pushover) analysis
- Accidental limit state (ALS)
- Ship impact, dropped objects
- Fire/explosion analysis

### GeniE → Splice (Pile-Soil Interaction)
```javascript
// Define pile-soil superelement for export
var psa = PileSoilActivity();
// Configure t-z, q-z, p-y curves for each soil layer
// Export superelement for inclusion in Sestra global model
```

### GeniE → Submod (Submodeling)
```javascript
// Export a refined local model with boundary conditions
// interpolated from a coarser global analysis
var expSub = ExportSubmod();
expSub.globalResultsFile = "global_results.sif";
expSub.boundaryDefinition = myBoundarySet;
expSub.export();
```

### GeniE → HydroD
Panel model export for hydrostatic, stability, and hydrodynamic calculations. Compartment definitions become tanks in HydroD.

### GeniE → Xtract (Post-Processing)
```javascript
// Xtract reads Sestra result files (.Rxx) directly
// Results can also be viewed within GeniE after LoadResultsActivity
```

## 3. Sesam Manager

The central launcher for all SESAM modules:
- **Start** from Windows: `Start > DNV SESAM > Sesam Manager`
- Manages product licenses
- Provides common working directory management
- Can launch batch workflows via Framework / AVM

## 4. Framework (AVM) Batch Processing

Framework automates the multi-module workflow:
```
GeniE JS script → GeniE batch mode → Sestra → Xtract/Post-processing
```

Command-line example:
```
GeniE.exe /batch my_model.js /exit
Sestra.exe my_model.fem
```

## 5. Typical Offshore Jacket Workflow

```
1. GeniE:
   - Build geometry (legs, braces, piles, deck)
   - Assign sections, materials, joints
   - Define loads (gravity, wind, wave, equipment)
   - Define environmental conditions
   - Mesh, define linear analysis

2. Sestra:
   - Solve for: in-place, seismic, fatigue, transportation, lift

3. Xtract:
   - Extract member forces, displacements, reactions
   - Generate utilization ratio reports

4. Wadam / Wajac:
   - Compute wave loads for extreme storm and fatigue sea states
   - Transfer back to GeniE/Sestra for structural analysis

5. Usfos:
   - Pushover analysis for reserve strength ratio (RSR)
   - Accidental limit state verification

6. Framework:
   - Automate design waves, load combinations, and batch runs
```

## 6. File Format Reference

| Extension | Module | Description |
|-----------|--------|-------------|
| `.js` | GeniE | JavaScript model script |
| `.xml` | GeniE | XML model/geometry exchange |
| `.fem` | Sestra | FEM input (nodes, elements, loads, BCs) |
| `.sin` | Sestra | Sestra input command file |
| `.Rxx` | Sestra | Results binary (R01, R02...) |
| `.sif` | Sestra | Superelement results file |
| `.stamod` | Wadam | Wadam structural model |
| `.fwt` | Usfos | Usfos input |
| `.res` | Usfos | Usfos results |
| `.hdx` | HydroD | HydroD model file |
