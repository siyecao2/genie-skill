# Report Generation

GeniE supports automated report generation via JavaScript scripting. Reports are constructed from **chapters** containing **tables**, **figures**, and **graphics**. References: `Help\UserDocumentation\Reporting\`

## Report Chapter Types

| Chapter Class | Content Documented |
|---------------|--------------------|
| `ChapterStructure()` | Structural parts: beam coordinates, properties, buckling data, hydro properties |
| `ChapterFEMResults()` | FEM analysis results: beam forces, stresses, node reactions, displacements |
| `FrameCodeCheckChapter()` | Frame code check results: member/joint utilisation factors, code options |
| `ChapterGraphics()` | Model graphics: displacement plots, stress contour plots, view captures |
| `ChapterLoads()` | Load documentation: load cases, load summary, load combinations |
| `ChapterMasses()` | Mass properties: total mass, COG, set-level mass breakdown, bounding boxes |
| `ChapterProperties()` | Section and hydro properties assigned to structural members |
| `PlateCodeCheckChapter()` | Plate/panel code check results (CSR BC & OT) |

## Making a Report — Basic Workflow

The `MakingAReport` workflow:
1. Create a `Report` object with a name
2. Add chapters to the report
3. Configure each chapter (analysis, load cases, sets, filters)
4. Add tables/figures to each chapter
5. Save/serialize the report to file

```javascript
// Step 1: Create Report object
MyReport = Report("DesignReport");

// Step 2: Add chapters
MyReport.add(ChapterStructure());
MyReport.add(ChapterFEMResults());
MyReport.add(FrameCodeCheckChapter());
MyReport.add(ChapterMasses());
MyReport.add(ChapterGraphics());

// Step 3: Configure each chapter
// ... (see individual chapter sections below)

// Step 5: Save to file
MyReport.saveAs("DesignReport.doc", mrWordXML);
```

## Export Options

| Format | Enum Value | Extension |
|--------|-----------|-----------|
| Word (XML) | `mrWordXML` | `.doc` |
| Excel (XML) | `mrExcelXML` | `.xls` |
| PDF | `mrPDF` | `.pdf` |

## Cut/Copy/Paste Chapters

Rearrange chapter order using `CutCopyAndPasteChapters`:

```javascript
MyReport.cutChapter(1);           // Cut chapter at position 1
MyReport.copyChapter(2);          // Copy chapter at position 2
MyReport.pasteChapter(3);         // Paste cut/copied chapter at position 3
```

## Chapter Configuration Details

### ChapterStructure — Structural Properties

```javascript
MyReport.add(ChapterStructure());
MyReport.element(1).setLoopSets(true);
MyReport.element(1).setSets(Array(BottomDeck, MainDeck));
MyReport.element(1).add(TableBeamCoordinate());
MyReport.element(1).add(TableBeamProperty());
MyReport.element(1).add(TableBeamBucklingData());
MyReport.element(1).add(TableBeamHydroProperty());
```

**Available tables:**
- `TableBeamCoordinate()` — beam end joint coordinates
- `TableBeamProperty()` — section assignment and material
- `TableBeamBucklingData()` — buckling length factors
- `TableBeamHydroProperty()` — Morison, marine growth, flooding
- `TablePlateProperty()` — plate thickness and material
- `TablePlateStiffener()` — stiffener layout and section type

### ChapterFEMResults — FEM Analysis Results

```javascript
MyReport.add(ChapterFEMResults());
MyReport.element(2).setCasePairs(Analysis1, Array(LC1_eqpm));
MyReport.element(2).setLoopLoads(true);
MyReport.element(2).setLoopSets(true);
MyReport.element(2).setSets(Array(ColumnSet, BraceSet));
MyReport.element(2).add(TableFEMBeamForceEnvelope());
MyReport.element(2).add(TableFEMBeamStressEnvelope());
MyReport.element(2).add(TableFEMNodeReaction());
MyReport.element(2).add(TableFEMNodeDisplacement());
MyReport.element(2).add(TableFEMBeamForce());
MyReport.element(2).add(TableFEMBeamStress());
```

**Important:** Reaction forces are printed for FE nodes with boundary conditions. Node displacements are listed for specified joints only.

### FrameCodeCheckChapter — Code Check Results

```javascript
MyReport.add(FrameCodeCheckChapter());
MyReport.element(3).run = Cc1.allRuns;
MyReport.element(3).worstLoadCase = true;
MyReport.element(3).limit = LimitLower("UfTot", 0.75);
MyReport.element(3).add(TableSummaryResult());
MyReport.element(3).add(TableMemberOptionsFull());
MyReport.element(3).add(TableMemberResultBrief());
```

**Available tables:**
- `TableSummaryResult()` — UF summary across all members
- `TableMemberResultBrief()` — key UF per member per load case
- `TableMemberResultFull()` — full detailed result per member
- `TableMemberOptionsFull()` — all member-specific options used
- `TableJointResultBrief()` — joint capacity check summary
- `TableJointResultFull()` — full joint utilisation details

**Filters:**
- `LimitLower("propertyName", threshold)` — filter rows where property >= threshold
- `LimitUpper("propertyName", threshold)` — filter rows where property <= threshold

### ChapterGraphics — Model Graphics/Plots

```javascript
MyReport.add(ChapterGraphics());
chap = MyReport.element(4);
chap.setName("Graphics");
chap.setLoopLoads(true);
chap.setLoopSets(true);
chap.setSets(Array(CellarDeck, MainDeck, Row_1, Row_2));
chap.setLoadCases(Analysis1, Array(LC_flare, LC_heli, LC_mass, LC_Operation));
chap.setCapacityManager(CapMan1);
chap.setCapacityRun(CapMan1.run(1));
chap.setWorstLoadcase(false);
```

### ChapterLoads — Load Documentation

```javascript
MyReport.add(ChapterLoads());
MyReport.element(5).setLoadCases(Analysis1, Array(LC1_eqpm, LC2_operation, LC3_storm));
MyReport.element(5).setLoopLoads(true);
MyReport.element(5).add(TableLoadSummary());
MyReport.element(5).add(TableLoadCase());
```

#### Center of Force (Load Chapter)

From `Help\UserDocumentation\Reporting\LoadChapter\CenterOfForce\`: documents the geometric center where the resultant force of a load case acts on the structure.

#### LoadCombConv2Mass (Load Chapter)

From `Help\UserDocumentation\Reporting\LoadChapter\LoadCombConv2Mass\`: documents how load combinations are converted to equivalent mass distributions for dynamic analysis.

### ChapterMasses — Mass Report

```javascript
MyReport.add(ChapterMasses());
MyReport.element(6).add(TableTotalMassAndCOG());
MyReport.element(6).add(TableSetMassAndCOG());
MyReport.element(6).add(TableSetContents());
MyReport.element(6).add(TableSetBoundingBox());
```

**Available tables:**
- `TableTotalMassAndCOG()` — total structural mass and center of gravity
- `TableSetMassAndCOG()` — mass and COG per named set
- `TableSetContents()` — detailed contents of each set
- `TableSetBoundingBox()` — geometric bounding box per set

### ChapterProperties — Section & Hydro Properties

```javascript
MyReport.add(ChapterProperties());
MyReport.element(7).add(TablePropertySection());
MyReport.element(7).add(TablePropertyHydro());
MyReport.element(7).add(TableSectionProperty());
```

### PlateCodeCheckChapter — Panel Results

```javascript
MyReport.add(PlateCodeCheckChapter());
MyReport.element(8).run = PanelCc.allRuns;
MyReport.element(8).worstPosition = true;
MyReport.element(8).worstLoadCase = true;
MyReport.element(8).add(TablePanelResultBrief());
MyReport.element(8).add(TablePanelResultFull());
```

## Looping Loads and Sets in Chapters

Enable automatic iteration over loads and sets to generate per-case and per-set sections:

```javascript
MyReport.add(ChapterFEMResults());
MyReport.element(1).setLoopLoads(true);       // Iterate over all load cases
MyReport.element(1).setLoopSets(true);        // Iterate over all named sets
MyReport.element(1).setLoadCases(Analysis1, Array(LC_Op, LC_Storm, LC_Acc));
MyReport.element(1).setSets(Array(PortSide, Starboard, Bow));
```

## Report Templates

### Built-in Templates

From `Help\UserDocumentation\Reporting\ReportTemplates\`:
- **ShipCargoholdTemplate** — pre-configured template for ship cargo hold analysis reports (XML-based `.xrp` format)

### Creating Custom Report Templates

Custom report templates are XML-based `.xrp` files. A reference template is at:
`Help\Tutorials\ReportTemplates\ShipCargoholdAnalysis.xml`

Templates pre-define:
- Chapter types and their order
- Table selections within each chapter
- Loop settings (load cases, sets)
- Export format preferences
- Filter criteria

To apply a template:
```javascript
MyReport.loadTemplate("ShipCargoholdAnalysis.xml");
MyReport.saveAs("CargoHoldReport.doc", mrWordXML);
```

## Complete Automated Reporting Example

```javascript
// Create a comprehensive design report
DesignReport = Report("Jacket_Design_Report");

// 1. Structure properties by set
DesignReport.add(ChapterStructure());
DesignReport.element(1).setLoopSets(true);
DesignReport.element(1).setSets(Array(Legs, Braces, Conductors));
DesignReport.element(1).add(TableBeamCoordinate());
DesignReport.element(1).add(TableBeamProperty());

// 2. FEM results for all operational load cases
DesignReport.add(ChapterFEMResults());
DesignReport.element(2).setCasePairs(Analysis1, Array(LC_Operating, LC_Storm, LC_Earthquake));
DesignReport.element(2).setLoopLoads(true);
DesignReport.element(2).setSets(Array(Legs));
DesignReport.element(2).add(TableFEMBeamForceEnvelope());
DesignReport.element(2).add(TableFEMNodeReaction());

// 3. Code check results for all runs
DesignReport.add(FrameCodeCheckChapter());
DesignReport.element(3).run = Cc1.allRuns;
DesignReport.element(3).worstLoadCase = true;
DesignReport.element(3).limit = LimitLower("UfTot", 0.5);
DesignReport.element(3).add(TableSummaryResult());
DesignReport.element(3).add(TableMemberResultBrief());

// 4. Displacement plots
DesignReport.add(ChapterGraphics());
DesignReport.element(4).setName("Displacement_Plots");
DesignReport.element(4).setLoopLoads(true);
DesignReport.element(4).setLoadCases(Analysis1, Array(LC_Storm));

// 5. Mass summary
DesignReport.add(ChapterMasses());
DesignReport.element(5).add(TableTotalMassAndCOG());
DesignReport.element(5).add(TableSetMassAndCOG());

// Export to Word and Excel
DesignReport.saveAs("Jacket_Design_Report.doc", mrWordXML);
DesignReport.saveAs("Jacket_Design_Report.xls", mrExcelXML);
```
