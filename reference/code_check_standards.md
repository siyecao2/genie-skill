# Code Checking Standards Reference

GeniE supports comprehensive structural code checking against internationally recognized standards. The CapacityManager framework is used to create, configure, and run checks for both members and joints.

## Reference Documents (PDF)

All reference documents are at `Help\ReferenceDocuments\` and contain detailed implementation notes for each standard:

| PDF File | Standard |
|----------|----------|
| `AISC.pdf`, `AISC9th.pdf` | AISC steel design — implementation details and formula references |
| `API_WSD.pdf`, `API_WSD2014.pdf`, `API_LRFD.pdf` | API RP 2A — offshore fixed platform design |
| `EUROCODE.pdf` | EN 1993-1-1 Eurocode 3 — European steel design |
| `ISO.pdf` | ISO 19902 — fixed steel offshore structures |
| `NORSOK.pdf`, `NORSOK2013.pdf` | NORSOK N-004 — Norwegian standard |
| `DS.pdf` | Danish Standard DS412/DS449 |
| `Complete_CmDoc.pdf` | Comprehensive Cm coefficient documentation |

## All Code Check Run Classes

Use `CapacityManager.AddRun()` to add code check runs. The second constructor argument in some classes supports year-specific editions.

### AISC — American Institute of Steel Construction

| JS Class | Standard | Design Method |
|----------|----------|---------------|
| `AiscAsd9thRun()` | AISC ASD 9th Edition | Allowable Stress Design |
| `AiscAsdRun()` | AISC ASD 2005 | Allowable Stress Design |
| `AiscLrfdRun()` | AISC LRFD 2005 | Load & Resistance Factor Design |

```javascript
// AISC ASD 2005
Cc1 = CapacityManager("CC_AISC");
Cc1.AddRun(AiscAsdRun());
Cc1.run(1).generalOptions.tensionFactor = 1.67;
Cc1.run(1).generalOptions.compressionFactor = 1.67;
Cc1.run(1).generalOptions.bendingFactor = 1.67;
Cc1.run(1).generalOptions.shearFactor = 1.67;
Cc1.run(1).generalOptions.torsionFactor = 1.67;

// AISC LRFD 2005 (resistance factors < 1.0)
Cc1.AddRun(AiscLrfdRun());
Cc1.run(2).generalOptions.tensionFactor = 0.9;
Cc1.run(2).generalOptions.compressionFactor = 0.9;
Cc1.run(2).generalOptions.bendingFactor = 0.9;
Cc1.run(2).generalOptions.shearFactor = 0.9;
Cc1.run(2).generalOptions.torsionFactor = 0.9;
Cc1.run(2).memberOptions.topFlangeLSupportSpacing = 1;
Cc1.run(2).memberOptions.bottomFlangeLSupportSpacing = 2;

// AISC ASD 9th (legacy)
Cc1.AddRun(AiscAsd9thRun());
Cc1.run(3).generalOptions.referenceEmodKSI = 29000.0;
Cc1.run(3).memberOptions.bendingCoefficient = 2;
```

### API — American Petroleum Institute RP 2A

| JS Class | Standard | Design Method |
|----------|----------|---------------|
| `ApiWsdRun()` or `ApiWsdRun2002()` | API RP 2A WSD 2002 | Working Stress Design |
| `ApiWsdRun2005()` | API RP 2A WSD 2005 | Working Stress Design |
| `ApiWsdRun2014()` | API RP 2A WSD 2014 | Working Stress Design |
| `ApiLrfdRun()` | API RP 2A LRFD 2003 | Load & Resistance Factor Design |

```javascript
// API WSD 2014 — includes member and joint checks
Cc1.AddRun(ApiWsdRun2014());
Cc1.run(1).includeMembers = true;
Cc1.run(1).includeJoints = true;
Cc1.run(1).generalOptions.aisc.tensionFactor = 1.67;
Cc1.run(1).generalOptions.aisc.compressionFactor = 1.67;
Cc1.run(1).generalOptions.aisc.bendingFactor = 1.67;
Cc1.run(1).generalOptions.aisc.shearFactor = 1.67;
Cc1.run(1).generalOptions.aisc.torsionFactor = 1.67;
Cc1.run(1).memberOptions.bucklingZ = BucklingLength(moMemberLength, 0.5);

// API LRFD — member resistance factors
Cc1.AddRun(ApiLrfdRun());
Cc1.run(2).generalOptions.RFPipeTens = 0.95;
Cc1.run(2).generalOptions.RFPipeComp = 0.85;
Cc1.run(2).generalOptions.RFPipeBend = 0.95;
Cc1.run(2).generalOptions.RFPipeShear = 0.95;
Cc1.run(2).generalOptions.RFPipeXPress = 0.8;
// Joint factors
Cc1.run(2).generalOptions.RFJointYield = 0.95;
Cc1.run(2).generalOptions.RFJointWeld = 0.54;
Cc1.run(2).generalOptions.RFJointYTens = 0.9;
Cc1.run(2).generalOptions.RFJointYComp = 0.95;
Cc1.run(2).generalOptions.RFJointYipb = 0.95;
Cc1.run(2).generalOptions.RFJointYopb = 0.95;
```

### EN 1993-1-1 — Eurocode 3

| JS Class | Standard |
|----------|----------|
| `EN199311Run()` | EN 1993-1-1:2005 Eurocode 3 |

Supports multiple National Annexes:
- `naStandard` — CEN standard partial factors
- `DanishNormal` — Danish normal control class
- `naDanishStricter` — Danish stricter control class
- `naNorwegianGrouse` — Norwegian national annex

```javascript
Cc1.AddRun(EN199311Run());
Cc1.run(1).generalOptions.nationalAnnex = naStandard;
Cc1.run(1).generalOptions.partialFactorM0 = 1.0;
Cc1.run(1).generalOptions.partialFactorM1 = 1.0;
Cc1.run(1).generalOptions.method1 = true;

// Danish stricter
Cc1.AddRun(EN199311Run());
Cc1.run(2).generalOptions.nationalAnnex = naDanishStricter;
Cc1.run(2).generalOptions.partialFactorM0 = 1.045;
Cc1.run(2).generalOptions.partialFactorM1 = 1.14;
Cc1.run(2).generalOptions.method1 = true;

// Norwegian
Cc1.AddRun(EN199311Run());
Cc1.run(3).generalOptions.nationalAnnex = naNorwegianGrouse;
```

### ISO 19902 — Fixed Steel Offshore Structures

| JS Class | Standard |
|----------|----------|
| `ISO19902Run_2007()` | ISO 19902:2007 |
| `ISO19902Run_2020()` | ISO 19902:2020 |

```javascript
// ISO 19902:2020 — member pipe factors
Cc1.AddRun(ISO19902Run_2020());
Cc1.run(1).generalOptions.RFPipeTens = 1.05;
Cc1.run(1).generalOptions.RFPipeComp = 1.15;
Cc1.run(1).generalOptions.RFPipeBend = 1.05;
Cc1.run(1).generalOptions.RFPipeShear = 1.05;
Cc1.run(1).generalOptions.RFPipeXPress = 1.25;
Cc1.run(1).memberOptions.momentReductionY = moCase1;
Cc1.run(1).memberOptions.momentReductionZ = moCase3;

// ISO 19902:2020 — joint capacity check
Cc1.run(1).generalOptions.C1_Y_ax = 25;
Cc1.run(1).generalOptions.C1_X_ax = 20;
Cc1.run(1).generalOptions.C1_K_ax = 14;
Cc1.run(1).generalOptions.C1_mom = 25;
Cc1.run(1).generalOptions.C2_Y_ax = 11;
Cc1.run(1).generalOptions.C2_X_ax = 22;
Cc1.run(1).generalOptions.C2_K_ax = 43;
Cc1.run(1).generalOptions.C2_mom = 43;
Cc1.run(1).generalOptions.RFJointYield = 1.05;
Cc1.run(1).generalOptions.RFJoint = 1.05;
Cc1.run(1).generalOptions.RFJointZj = 1.17;
```

### NORSOK N-004 — Norwegian Standard

| JS Class | Standard |
|----------|----------|
| `DNVrulesRun()` | NORSOK N-004 |

```javascript
// NORSOK N-004
Cc1.AddRun(DNVrulesRun());
```

### DS — Danish Standard

| JS Class | Standard | Notes |
|----------|----------|-------|
| `DSRun()` | DS412 / DS449 | Supports both DS412 and DS449 standards |

```javascript
Cc1.AddRun(DSRun());
Cc1.run(1).generalOptions.isDS449CodeCheck = true;
Cc1.run(1).generalOptions.gammaFy = 1.15;
Cc1.run(1).generalOptions.gammaFu = 1.41;
Cc1.run(1).memberOptions.bucklingY = BucklingLength(1, 1);
Cc1.run(1).memberOptions.bucklingZ = BucklingLength(1, 1);
```

### CSR — Common Structural Rules (IACS)

| JS Class | Standard | Applied To |
|----------|----------|------------|
| `CSRBulkRun()` | CSR for Bulk Carriers | Stiffened panels (yield + buckling) |
| `CSRTankRun()` | CSR for Oil Tankers | Stiffened/unstiffened panels |
| `CSR_BC_OT_Run()` | CSR BC & OT Harmonised | Panel code check (new harmonised CSR) |

```javascript
// CSR Bulk Carrier
Cc1.AddRun(CSRBulkRun());
Cc1.run(1).generalOptions.checkBuckling = true;
Cc1.run(1).generalOptions.checkYield = true;
Cc1.run(1).generalOptions.safetyFactorBuckling = 1.0;
Cc1.run(1).generalOptions.safetyFactorYield = 1.0;
Cc1.run(1).generalOptions.transverseStressOption = tsGeneralBending;
Cc1.run(1).generalOptions.poissonCorrectionOption = psElementStress;
Cc1.run(1).panelOptions.panelThickness = ptAverageIdealisedPanel;
Cc1.run(1).panelOptions.rotationBoundaryTop = rbClamped;

// CSR Tank
Cc1.AddRun(CSRTankRun());
Cc1.run(2).generalOptions.parallelMode = true;
Cc1.run(2).panelOptions.unstiffened.idealizedLength = 1;

// CSR BC & OT Harmonised
Cc1.AddRun(CSR_BC_OT_Run());
```

## Selecting a Standard in CapacityManager

To use a standard in the GUI:
1. Right-click the model in the Browser → **Create Capacity Manager**
2. In the Capacity Manager dialog, click **Add Run**
3. Select the desired standard from the **Code Check Standard** dropdown
4. Configure general options, member-specific options, and joint-specific options in the tabs
5. Click **Compute Code Checks** or script `Cc1.RunAll()`

## Member vs Joint Checking

The `includeMembers` and `includeJoints` properties control which elements are checked:

```javascript
// Check both members and joints (API WSD)
Cc1.AddRun(ApiWsdRun2014());
Cc1.run(1).includeMembers = true;
Cc1.run(1).includeJoints = true;

// Joint-specific options
Cc1.run(1).braceOptions.braceType = braceTypeX;
Cc1.run(1).braceOptions.loadTransfer = true;
Cc1.run(1).braceOptions.throughBrace = true;

// Member-specific options
Cc1.run(1).memberOptions.flooding = cfNotFlooded;
Cc1.run(1).memberOptions.bucklingZ = BucklingLength(600 cm, 1);
```

## Running Code Checks

```javascript
// Run all code checks defined in the CapacityManager
Cc1.RunAll();

// Or run individual checks
Cc1.run(1).Compute();
```

## PDF Reference Documents Detail

Each PDF in `Help\ReferenceDocuments\` contains:
- **Formula references** — exact code clause citations with equation numbers
- **Implementation notes** — DNV assumptions, simplifications, and numerical methods
- **Validation cases** — test models with known hand-calculation results
- **Section coverage** — which profile types are supported (I, H, CHS, RHS, L, T, etc.)
- **Limitations** — known constraints (e.g., D/t ratio limits, slenderness ratio caps)
