# GeniE Tutorial Index (V8.8-08)

## Tutorial Location

```
{SESAM_HOME}\Program Files\DNV\GeniE V8.8-08\Help\Tutorials\
```

Each tutorial has:
- A **PDF document** (e.g., `B1_Getting_Started.pdf`)
- A **JavaScript script** (e.g., `B1_Getting_Started.js`)
- Supporting model files (if any)

## Basic Tutorials (B1–B12)

Designed for new users. Each takes 1–3 hours.

| ID | Name | PDF | JS Script | Topic |
|----|------|-----|-----------|-------|
| **B1** | Getting Started | B1_Getting_Started.pdf | B1_Getting_Started.js | GUI overview, creating a simple beam, loads, analysis |
| **B2** | Frame Model | B2_Frame_Model.pdf | B2_Frame_Model.js | Beam structures, sections, materials, linear static analysis |
| **B3** | Plate Model | B3_Plate_Model.pdf | B3_Plate_Model.js | Flat plates, thickness, mesh, stiffeners |
| **B4** | Shell Model | B4_Shell_Model.pdf | B4_Shell_Model.js | Curved shells, general plates, complex geometries |
| **B5** | Equipment & Loads | B5_Equipment_Loads.pdf | B5_Equipment_Loads.js | Equipment mass, point loads, line loads, load cases |
| **B6** | Import & Export | B6_Import_Export.pdf | B6_Import_Export.js | Import FEM, export to Sestra, Wadam, Usfos |
| **B7** | Code Checking | B7_Code_Checking.pdf | B7_Code_Checking.js | CapacityManager, member checks, utilization ratios |
| **B8** | Eigenvalue Analysis | B8_Eigenvalue_Analysis.pdf | B8_Eigenvalue_Analysis.js | Natural frequencies, mode shapes, buckling |
| **B9** | Dynamic Analysis | B9_Dynamic_Analysis.pdf | B9_Dynamic_Analysis.js | Spectral, time-history, seismic |
| **B10** | Pile-Soil Interaction | B10_Pile_Soil.pdf | B10_Pile_Soil.js | Splice, p-y/t-z curves, soil springs |
| **B11** | Submodeling | B11_Submodeling.pdf | B11_Submodeling.js | Extract submodel from global model, boundary conditions |
| **B12** | Scripting Introduction | B12_Scripting.pdf | B12_Scripting.js | JS scripting basics, variables, loops, functions |

## Advanced Tutorials (A1–A16)

For experienced users. Domain-specific workflows.

| ID | Name | PDF | JS Script | Topic |
|----|------|-----|-----------|-------|
| **A1** | Jacket Modeling | A1_Jacket_Modeling.pdf | A1_Jacket_Modeling.js | Full jacket structure: legs, braces, joints |
| **A2** | Topside Modeling | A2_Topside_Modeling.pdf | A2_Topside_Modeling.js | Deck framing, plate girders, equipment |
| **A3** | Wave Load Analysis | A3_Wave_Load_Analysis.pdf | A3_Wave_Load_Analysis.js | Wadam export, wave kinematics, load transfer |
| **A4** | Fatigue Analysis | A4_Fatigue_Analysis.pdf | A4_Fatigue_Analysis.js | Stofat, hot-spot stress, SN curves |
| **A5** | Pushover Analysis | A5_Pushover_Analysis.pdf | A5_Pushover_Analysis.js | Usfos export, nonlinear collapse |
| **A6** | Ship Impact | A6_Ship_Impact.pdf | A6_Ship_Impact.js | Accidental loads, energy absorption |
| **A7** | Tubular Joints | A7_Tubular_Joints.pdf | A7_Tubular_Joints.js | Joint cans, stiffened joints, CapacityJoint |
| **A8** | FPSO Structural | A8_FPSO_Structural.pdf | A8_FPSO_Structural.js | Ship-shaped structures, global hull model |
| **A9** | Wind Turbine | A9_Wind_Turbine.pdf | A9_Wind_Turbine.js | OWT jacket/monopile, RNA modeling |
| **A10** | Complex Sections | A10_Complex_Sections.pdf | A10_Complex_Sections.js | GeneralSection, composite, built-up sections |
| **A11** | Advanced Meshing | A11_Advanced_Meshing.pdf | A11_Advanced_Meshing.js | Mesh refinement, transitions, quality control |
| **A12** | Scripting Advanced | A12_Scripting_Advanced.pdf | A12_Scripting_Advanced.js | Loops, ModelTransformer, parametric models |
| **A13** | Hydrodynamic Model | A13_Hydrodynamic_Model.pdf | A13_Hydrodynamic_Model.js | HydroD panel model, compartments, stability |
| **A14** | LRFD Code Check | A14_LRFD_Code_Check.pdf | A14_LRFD_Code_Check.js | Detailed NORSOK / ISO code checks |
| **A15** | Reinforcement | A15_Reinforcement.pdf | A15_Reinforcement.js | Concrete reinforcement modeling |
| **A16** | Batch Processing | A16_Batch_Processing.pdf | A16_Batch_Processing.js | Framework AVM, batch runs, automation |

## How to Use Tutorials

### Open a Tutorial
1. Start GeniE
2. `Help > Tutorials` → opens the tutorial folder
3. Open the PDF for step-by-step instructions
4. Load the `.js` script: `File > Open > Script` or drag-and-drop into GeniE
5. Run: `Tools > Run Script` or F5

### Use as Starting Point
```javascript
// Copy a tutorial script as template
// 1. Open B1_Getting_Started.js in a text editor
// 2. Study the structure (see best_practices.md for recommended order)
// 3. Copy sections you need:
//    - Material/section definitions
//    - Guide plane setup
//    - Analysis configuration
// 4. Replace geometry with your own structure
// 5. Adjust loads, BCs, and analysis settings
```

## Recommended Learning Path

### Week 1: Foundation
1. **B1** — Getting Started (GUI)
2. **B2** — Frame Model (beams)
3. **B12** — Scripting Introduction (JS basics)

### Week 2: Plates & Loads
4. **B3** — Plate Model
5. **B5** — Equipment & Loads
6. **B6** — Import & Export

### Week 3: Analysis
7. **B7** — Code Checking
8. **B8** — Eigenvalue Analysis
9. **B10** — Pile-Soil Interaction

### Week 4: Offshore Application
10. **A1** — Jacket Modeling
11. **A3** — Wave Load Analysis
12. **A12** — Scripting Advanced (parametric models)

### Beyond: Specialized Topics
- **A4** — Fatigue (Stofat)
- **A5** — Pushover (Usfos)
- **A7** — Tubular Joints (CapacityJoint)
- **A9** — Wind Turbine
- **A14** — LRFD Code Check (NORSOK)
- **A16** — Batch Processing (Framework)

## Tutorial Script Tips

```javascript
// Tutorial scripts often contain commented-out sections
// Uncomment to activate alternative approaches:
//   //var b = Beam(p1, p2);          // alternative: straight
//   var b = Beam(p1, p2, curve);     // curved beam

// Tutorials use "print()" for console output to explain steps
print("Step 3: Assigning sections...");

// Some tutorials include result verification:
print("Expected displacement at top: ~0.05 m");
print("Actual displacement: " + node.getDisplacement());
```
