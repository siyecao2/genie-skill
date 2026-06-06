# Panel Code Check — DNVGL CSR BC & OT

GeniE supports panel-level code checking according to the harmonised Common Structural Rules for Bulk Carriers and Oil Tankers (CSR BC & OT). The workflow covers permissible UF screening, yield screening, and buckling evaluation of stiffened and unstiffened panels.

Reference: `Help\UserDocumentation\PanelCodeCheck\`

## Workflow Overview

```
┌────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│ Create Capacity    │    │ Define Panels        │    │ Create CSR BC & OT   │
│ Manager for Panels │───▶│ (DNVGLOrCSRBCAndOT   │───▶│ Code Check Run       │
│ (CapacityManager)  │    │  DefiningPanels)      │    │ (CSR_BC_OT_Run)      │
└────────────────────┘    └──────────────────────┘    └──────────┬───────────┘
                                                                 │
                                            ┌────────────────────▼───────────┐
                                            │ Configure Run Parameters        │
                                            │ ┌─────────────────────────────┐ │
                                            │ │ General Parameters           │ │
                                            │ │ Panel Parameters             │ │
                                            │ │ Permissible UF Parameters    │ │
                                            │ │ Yield Parameters             │ │
                                            │ └─────────────────────────────┘ │
                                            └────────────────────┬───────────┘
                                                                 │
                              ┌──────────────────────────────────▼─────────────┐
                              │ Screening & Checking                            │
                              │ ┌──────────────────┐  ┌──────────────────────┐  │
                              │ │ Permissible UF    │  │ Yield Screening      │  │
                              │ │ • Coarse Input    │  │ • Coarse Input       │  │
                              │ │ • Fine Mesh       │  │ • Fine Mesh          │  │
                              │ │ • Manual Input    │  │ • Manual Input       │  │
                              │ └──────────────────┘  └──────────────────────┘  │
                              └────────────────────┬────────────────────────────┘
                                                   │
                                          ┌────────▼────────┐
                                          │ RunAll / Compute │
                                          └─────────────────┘
```

## Key JS Classes

| Class | Purpose |
|-------|---------|
| `CapacityManager("name")` | Top-level container for code check definitions |
| `CapacityPanel` | Represents a code-checkable panel derived from plates |
| `CSR_BC_OT_Run()` | Harmonised CSR panel code check run |
| `LinearSlicerCSRBulk()` | Slice model into stiffened panels (Bulk carrier rules) |
| `LinearSlicerCSRTank()` | Slice model into panels (Tanker rules) |

## Creating a Capacity Manager for Panels

```javascript
// Create a CapacityManager dedicated to panel code checking
PanelCc = CapacityManager("Panel_CC");

// Define panels from the structural model
// Panels are automatically detected via linear slicing or manual definition
```

## Defining Panels

Panels are defined via `Help\UserDocumentation\PanelCodeCheck\DNVGLOrCSRBCAndOTDefiningPanels\`. The process:
1. Select a group of plates in the model
2. Use the **Create Panels** tool — GeniE auto-detects stiffener boundaries
3. Panels are grouped using **named sets** for batch processing
4. Individual panel properties (thickness, stiffener geometry, boundary conditions) can be modified

## Creating a CSR Panel Code Check Run

```javascript
PanelCc = CapacityManager("Panel_CC");
PanelCc.AddRun(CSR_BC_OT_Run());

// General Parameters
PanelCc.run(1).generalOptions.yieldCheck = true;
PanelCc.run(1).generalOptions.bucklingCheck = true;

// Panel Parameters
PanelCc.run(1).panelOptions.panelThickness = ptAverageIdealisedPanel;
PanelCc.run(1).panelOptions.rotationBoundaryTop = rbClamped;

// Permissible UF Parameters
PanelCc.run(1).permissibleUF.coarseInputFromFE = true;
PanelCc.run(1).permissibleUF.manualInputValue = 0.8;

// Yield Parameters
PanelCc.run(1).yield.coarseInputFromFE = true;
```

## Permissible UF Screening

Three modes (from `Help\UserDocumentation\PanelCodeCheck\DNVGLOrCSRBCAndOTPermissibleUF\`):

### 1. Coarse Input View
```javascript
PanelCc.run(1).permissibleUF.coarseInputFromFE = true;
// Reads von Mises stress from coarse mesh FEM results
// Computes permissible utilisation factor per panel
```

### 2. Fine Mesh View
```javascript
PanelCc.run(1).permissibleUF.fineMeshFromFE = true;
// Uses fine mesh results for higher accuracy
// Typically applied to critical panels identified in coarse screening
```

### 3. Manual Input View
```javascript
PanelCc.run(1).permissibleUF.manualInput = true;
PanelCc.run(1).permissibleUF.manualInputValue = 0.75;
// User directly specifies permitted UF for each panel
// Useful when code values are overridden by project specifications
```

## Yield Screening

Three modes (from `Help\UserDocumentation\PanelCodeCheck\DNVGLOrCSRBCAndOTYieldScreening\`):

### 1. Coarse Input
```javascript
PanelCc.run(1).yield.coarseInputFromFE = true;
// Evaluates yield from coarse mesh stress results
```

### 2. Fine Mesh
```javascript
PanelCc.run(1).yield.fineMeshFromFE = true;
// Fine mesh yield evaluation for critical panels
```

### 3. Manual Input
```javascript
PanelCc.run(1).yield.manualInput = true;
PanelCc.run(1).yield.manualInputValue = 355 MPa;
// Direct yield stress input (e.g., for S355 steel)
```

## Single Panel Tool

The single panel tool (`DNVGLOrCSRBCAndOTSinglePanelTool`) allows quick checking of an individual panel without setting up a full CapacityManager:

```javascript
// Select a panel → Right-click → Single Panel Check
// Useful for quick design iterations
```

## Using Named Sets for Panel Grouping

```javascript
// Create named sets to organize panels
BottomPanels = Set("BottomPanels");
SidePanels = Set("SidePanels");
DeckPanels = Set("DeckPanels");

// Populate sets with panels
BottomPanels.add(Panel1, Panel2, Panel3);
SidePanels.add(Panel4, Panel5);
DeckPanels.add(Panel6, Panel7, Panel8);

// Assign different run configurations per set
PanelCc.run(1).setPanels(BottomPanels);
PanelCc.run(2).setPanels(SidePanels);
```

## Panel Functions

Manipulate panel geometry via scripting:

| Function | Description |
|----------|-------------|
| `DividePanelAlongDirection(panel, direction, offset)` | Split panel along an axis at given offset |
| `DividePanelByPlane(panel, plane)` | Split panel by an intersecting guide plane |
| `ExplodePanel(panel)` | Decompose panel into individual stiffener span panels |
| `JoinTwoPanels(panel1, panel2)` | Merge two adjacent panels into one |

```javascript
// Divide panel at mid-span
GuidePlane1 = GuidePlane();
GuidePlane1.setPlane(Vector3d(0, 0, 1), Point(0, 0, 3 m));
NewPanels = DividePanelByPlane(Panel1, GuidePlane1);

// Join two panels
JoinedPanel = JoinTwoPanels(PanelA, PanelB);
```

## RunAll Command for Batch Processing

```javascript
// Compute all panel code check runs
PanelCc.RunAll();

// Or compute individual run
PanelCc.run(1).Compute();
```

## Complete Panel Code Check Setup Example

```javascript
// 1. Create CapacityManager
PanelCc = CapacityManager("CSR_BC_OT_PanelCheck");

// 2. Create a CSR BC & OT run
PanelCc.AddRun(CSR_BC_OT_Run());

// 3. Configure general options
PanelCc.run(1).generalOptions.yieldCheck = true;
PanelCc.run(1).generalOptions.bucklingCheck = true;

// 4. Configure panel options
PanelCc.run(1).panelOptions.panelThickness = ptAverageIdealisedPanel;
PanelCc.run(1).panelOptions.rotationBoundaryTop = rbClamped;
PanelCc.run(1).panelOptions.rotationBoundaryBottom = rbSimplySupported;

// 5. Permissible UF — use coarse input from FE
PanelCc.run(1).permissibleUF.coarseInputFromFE = true;

// 6. Yield screening — coarse input from FE
PanelCc.run(1).yield.coarseInputFromFE = true;

// 7. Run the code check
PanelCc.RunAll();
```

## Reporting Panel Results

Results are documented using `PlateCodeCheckChapter`:

```javascript
MyReport = Report("Panel_CC_Report");
MyReport.add(PlateCodeCheckChapter());
MyReport.element(1).run = PanelCc.allRuns;
MyReport.element(1).worstPosition = true;
MyReport.element(1).worstLoadCase = true;
MyReport.element(1).add(TablePanelResultBrief());
MyReport.element(1).add(TablePanelResultFull());
MyReport.saveAs("Panel_CC_Report.xls", mrExcelXML);
```

## Key Parameters Reference

| Parameter | Description | Typical Values |
|-----------|-------------|----------------|
| `panelThickness` | How panel thickness is determined | `ptAverageIdealisedPanel`, `ptAsBuilt` |
| `rotationBoundaryTop` | Top edge rotational restraint | `rbClamped`, `rbSimplySupported` |
| `rotationBoundaryBottom` | Bottom edge rotational restraint | `rbClamped`, `rbSimplySupported` |
| `safetyFactorBuckling` | Buckling safety factor | 1.0 (default per CSR) |
| `safetyFactorYield` | Yield safety factor | 1.0 (default per CSR) |
| `transverseStressOption` | How transverse stress is treated | `tsGeneralBending` |
| `poissonCorrectionOption` | Poisson effect correction | `psElementStress` |
| `parallelMode` | Enable parallel computation | `true` / `false` |
