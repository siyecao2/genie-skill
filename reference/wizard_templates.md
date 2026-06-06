# Excel Wizard Templates

GeniE ships with Excel-based wizard templates that generate JavaScript (.js) journal files for automated structural modelling. All templates are located at `Help\WizardTemplates\`.

## Available Templates

### Deck Wizard
| File | Version |
|------|---------|
| `Deck_wizard_Excel2003.xls` | Excel 2003 |
| `Deck_wizard_Excel2010.xlsm` | Excel 2010 (macro-enabled) |

Generates a complete topside deck model JS script:
- Plates and stiffened panels with configurable dimensions
- Beams (primary/secondary girders) with section assignments
- Stiffener layout (spacing, direction, section type)
- Equipment masses with footprint geometry
- Surface loads (area loads on deck)
- Load cases (dead load, live load, environmental)

**Key configurable parameters:**
- Deck length, width, and number of bays in each direction
- Plate thickness per bay/stiffener direction
- Stiffener type, spacing, and orientation
- Girder section profiles (I-beam, T-beam, L-section)
- Equipment mass, COG position, and footprint dimensions
- Material grade and density
- Boundary conditions at deck corners

### Jacket Wizard
| File | Version |
|------|---------|
| `Jacket_wizard_Excel2003.xls` | Excel 2003 |
| `Jacket_wizard_Excel2010.xlsm` | Excel 2010 (macro-enabled) |

Generates a 4-leg jacket structure model JS script:
- Legs with batter (inclination) and section profiles
- Horizontal and diagonal braces at each bay level
- Mudmats at base
- Pile group definitions with stick-up length
- Marine growth thickness and density zones
- Topside mass simulation as point mass

**Key configurable parameters:**
- Jacket height and top dimensions
- Number of bays and bay heights
- Leg batter angle (X and Y directions)
- Leg and brace section diameters and wall thicknesses
- Mudmat dimensions and plate thickness
- Marine growth thickness (typically 50-150 mm) and density
- Topside mass and COG location
- Material yield strength

### General Scripting Wizard
| File | Description |
|------|-------------|
| `Wizard for GeniE scripting_rev1.xls` | Multi-structure type wizard |

Supports 5 structure types:
1. **Tubular K-joint** — CHS chord/brace connections with stub length control
2. **Box-joint** — Rectangular hollow section connections
3. **Tanker panel** — Panel modelling with stiffeners for ship-type structures
4. **Corrugated Bulkhead** — Corrugated plate geometry with trough/crest dimensions
5. **Semi-Sub panel** — Semi-submersible pontoon/column panel modelling

**Key configurable parameters:**
- Joint member dimensions, wall thickness, intersection angles
- Stub length for chord and braces
- Panel dimensions, stiffener count and spacing
- Corrugation pitch, depth, and plate thickness
- Option to launch official or development GeniE version

## Workflow

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│ Copy wizard .xls│────▶│ Open in Excel        │────▶│ Modify parameters   │
│ to workspace    │     │ (enable macros)      │     │ in designated cells │
└─────────────────┘     └──────────────────────┘     └──────────┬──────────┘
                                                                │
                                              ┌─────────────────▼─────────────────┐
                                              │ Press "Write Genie Input" button  │
                                              │  → Generates .js journal file     │
                                              └─────────────────┬─────────────────┘
                                                                │
                           ┌────────────────────────────────────┼─────────────────────────────┐
                           │                                    │                             │
                  ┌────────▼────────┐               ┌───────────▼──────────┐     ┌───────────▼──────────┐
                  │ File → Read     │               │ Press "Run Genie"    │     │ GeniE.exe            │
                  │ Command File... │               │ button directly      │     │ -j journal.js        │
                  │ in GeniE GUI    │               │ → Launches GeniE     │     │ (headless execution)  │
                  └────────┬────────┘               │   & executes script  │     └─────────────────────┘
                           │                        └──────────────────────┘
                  ┌────────▼────────┐
                  │ Model appears   │
                  │ in GeniE        │
                  │ workspace       │
                  └─────────────────┘
```

## Usage Instructions

1. Create a new GeniE workspace: `File → New Workspace`
2. Copy the relevant wizard file into the workspace directory
3. Right-click the file → Properties → uncheck "Read-only"
4. Open the file in Excel; read the **Help** sheet in the wizard for guidance
5. Fill in all parameter cells (marked with labels or color coding)
6. Click **"Write Genie Input"** to generate the .js journal file
7. In GeniE, use **File → Read Command File...** to execute the journal

> **Tip:** Instead of steps 6-7, you can click **"Run Genie"** to automatically launch GeniE and execute the journal file. This option also supports selecting an alternative program version (e.g., development build).

## Generated Journal File Structure

The wizard output JS journal typically follows this pattern:

```javascript
// Auto-generated by Deck Wizard - GeniE V8.8-08
// Units: SI (m, kg, N)
GuidePlane = GuidePlane();
GuidePlane.setPlane(Vector3d(0, 0, 1), Point(0, 0, 0));

// Create joints at grid intersections
Joint1 = Joint(Point(0, 0, 0));
Joint2 = Joint(Point(6 m, 0, 0));
Joint3 = Joint(Point(12 m, 0, 0));
// ... more joints

// Create beams between joints
Beam1 = Beam(Joint1, Joint2);
Beam2 = Beam(Joint2, Joint3);
// Beam section assignment
Beam1.section = Section("HE400A");

// Create plates
Plate1 = Plate(Array(Joint1, Joint2, Joint5, Joint4));
Plate1.thickness = 12 mm;
Plate1.material = MaterialLinear("S355", 210 GPa, 0.3, 7850 kg/m^3);

// Assign stiffeners
Plate1.stiffener = Stiffener("HP200x10", 600 mm, XX);

// Create sets for grouping
DeckBeams = Set();
DeckBeams.add(Beam1, Beam2, Beam3);
```

## Important Notes

- Wizards require Excel with macros enabled (.xlsm files use VBA)
- Increase file protection before writing to wizard files (uncheck read-only)
- The `Readme.txt` in the WizardTemplates folder contains quick-reference instructions
- Wizard-generated scripts are a starting point — they should be reviewed and adjusted for your specific design
- Workspace file extension is `.js` — do not confuse with the journal command file
