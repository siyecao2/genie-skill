# Materials & Sections - GeniE SESAM Snippets
> **SESAM 源**: `GeniE V8.8-08 Help\UserDocumentation\CreateMaterial\CreateMaterial.htm + Help\UserDocumentation\CreateSection\`

## MaterialLinear Constructor (Recommended)

```javascript
// MaterialLinear(name, E, poisson, density, thermal_expansion)
var S355 = MaterialLinear("S355");
S355.youngsModulus = 2.1e11 Pa;
S355.poissonRatio = 0.3;
S355.density = 7850 kg/m^3;
S355.thermalExpansionCoefficient = 1.17e-5 1/deg;

// Direct constructor with all parameters
var S420 = MaterialLinear(420e6 Pa, 2.1e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg);
S420.name = "S420";

// Set yield stress after creation
var DH36 = MaterialLinear("DH36");
DH36.youngsModulus = 2.05e11 Pa;
DH36.poissonRatio = 0.3;
DH36.density = 7850 kg/m^3;
DH36.yieldStress = 355e6 Pa;
DH36.thermalExpansionCoefficient = 1.17e-5 1/deg;
```

## Material Constructor (Legacy)

```javascript
// Legacy Material() - newer scripts prefer MaterialLinear
var matSt37 = Material();
matSt37.name = "St37";
matSt37.E = 2.1e11 Pa;
matSt37.poissonRatio = 0.3;
matSt37.density = 7850 kg/m^3;
matSt37.fy = 235e6 Pa;
```

## Section Constructors

```javascript
// PipeSection(diameter, wall_thickness)
var pipeMain = PipeSection(0.762 m, 0.0254 m);
pipeMain.name = "PIPE_762x25.4";

var pipeBrace = PipeSection(0.508 m, 0.019 m);
pipeBrace.name = "PIPE_508x19";

// ISection(h, b_top, b_bot, t_web, t_fl_top, t_fl_bot)
var beamDeck = ISection(0.6 m, 0.3 m, 0.3 m, 0.012 m, 0.02 m, 0.02 m);
beamDeck.name = "HE600B";

// BoxSection(h, w, t_web, t_fl_top, t_fl_bot)
var boxCol = BoxSection(0.4 m, 0.4 m, 0.02 m, 0.025 m, 0.025 m);
boxCol.name = "BOX_400x400";

// ChannelSection(h, w, t_web, t_flange)
var chStrut = ChannelSection(0.3 m, 0.1 m, 0.008 m, 0.012 m);
chStrut.name = "CH300";

// BarSection(area) - simple for truss/tension-only elements
var barRod = BarSection(0.005 m^2);
barRod.name = "ROD_D50";
```

## UnsymISection & GeneralSection

```javascript
// UnsymISection for T-bars or unsymmetric I
var tBar = UnsymISection(0.3 m, 0.15 m, 0.1 m, 0.008 m, 0.012 m, 0.01 m);
tBar.name = "TEAR_BAR";

// GeneralSection fallback for custom geometry
var customSect = GeneralSection();
customSect.name = "CUSTOM_CHORD";
customSect.area = 0.025 m^2;
customSect.Iy = 4.5e-3 m^4;
customSect.Iz = 4.5e-3 m^4;
customSect.It = 5.2e-4 m^4;
customSect.WyMin = 1.5e-3 m^3;
customSect.WzMin = 1.5e-3 m^3;
```

## ConeSection (Tapered)

```javascript
// ConeSection(start_diameter, end_diameter, wall_thickness)
var coneTrans = ConeSection(0.762 m, 0.508 m, 0.025 m);
coneTrans.name = "CONE_762_508";

// For transition pieces between jacket legs
var transPiece = ConeSection(1.2 m, 0.8 m, 0.05 m);
transPiece.name = "TP_1200_800";
```

## Thickness Definition

```javascript
// Plate thickness with unit suffix
var twall = 12 mm;
var tdeck = 20 mm;
var tbulkhead = 8 mm;
var tstiffener = 15 mm;

// Thickness as a named variable for assignment
var thickBottom = 25 mm;
var thickSide = 18 mm;
var thickDeck = 30 mm;

// Corrosion addition example
var tNominal = 20 mm;
var corrAllow = 3 mm;
var tGross = tNominal + corrAllow;  // 23 mm gross scantling
```

## setDefault() Usage

```javascript
// Set default material and section for subsequent beam creation
setDefault("Material", S355);
setDefault("PipeSection", pipeMain);

// All beams created after this inherit these defaults unless overridden
var b1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
// b1 automatically gets S355 material and pipeMain section

// Override default for a specific beam
var b2 = StraightBeam(Point(0 m, 5 m, 0 m), Point(10 m, 5 m, 0 m));
b2.section = pipeBrace;
```

## Material_library.xml Usage

```javascript
// Import materials from DNV library XML
ImportMaterialLibrary("Material_library.xml");

// Access pre-defined materials by name
var S235 = MaterialLinear("S 235");
S235.name = "S235";

var S460 = MaterialLinear("S 460");
S460.name = "S460";

// Create all common offshore grades
var steelGrades = {
    S235: MaterialLinear("S235", 2.1e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg),
    S355: MaterialLinear("S355", 2.1e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg),
    S420: MaterialLinear("S420", 2.1e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg),
    S460: MaterialLinear("S460", 2.1e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg)
};

// Grade-to-yield mapping
steelGrades.S235.yieldStress = 235e6 Pa;
steelGrades.S355.yieldStress = 355e6 Pa;
steelGrades.S420.yieldStress = 420e6 Pa;
steelGrades.S460.yieldStress = 460e6 Pa;

// Offshore shipbuilding grades
var DH36 = MaterialLinear("DH36", 2.05e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg);
DH36.yieldStress = 355e6 Pa;

var EH36 = MaterialLinear("EH36", 2.05e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg);
EH36.yieldStress = 355e6 Pa;

// Legacy German grades
var St37 = MaterialLinear("St37", 2.1e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg);
St37.yieldStress = 235e6 Pa;

var St52 = MaterialLinear("St52", 2.1e11 Pa, 0.3, 7850 kg/m^3, 1.17e-5 1/deg);
St52.yieldStress = 355e6 Pa;
```

## Corrosion Addition

```javascript
// Load the corrosion addition revision script
// corr_add_to_gross_rev3_in.js - applies corrosion addition to gross scantlings
//
// Typical workflow:
// 1. Define corrosion allowance zones
var splashZoneCorrosion = 4 mm;
var atmosphericCorrosion = 2 mm;
var submergedCorrosion = 3 mm;

// 2. Apply to sections
// For pipe:
var pipeSection = PipeSection(0.610 m, 0.022 m);  // gross thickness
// After corrosion: effective thickness may be reduced to ~18 mm in splash zone
pipeSection.material = S355;

// 3. In code checking, corrosion is typically handled as parameter input
// rather than modifying the section directly
```