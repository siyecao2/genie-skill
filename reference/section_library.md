# Section Library Reference (DNV SESAM GeniE V8.8-08)

## 1. Library Formats

### KZY Format
- Binary or ASCII format, historically the primary GeniE section database.
- Loaded automatically at startup from the GeniE installation directory.
- Example path: `{SESAM_HOME}\Program Files\DNV\GeniE V8.8-08\bin\aisc_v3.kzy`

### XML Format
- Modern, human-readable format introduced in later GeniE versions.
- Supports richer metadata and easier external editing.
- Example path: `{SESAM_HOME}\Program Files\DNV\GeniE V8.8-08\bin\BS4_Sections_Part1_1993.xml`

## 2. Built-in Section Libraries

### AISC v3 (aisc_v3.kzy)
- American Institute of Steel Construction standard sections.
- Referenced in JS via the profile name string directly:

```javascript
var b = Beam(Point(0,0,0), Point(5,0,0));
b.section = Section("W21x44");
b.section = Section("HSS6x6x1/4");
b.section = Section("L4x4x1/4");
b.section = Section("C8x11.5");
b.section = Section("W14x90");
```

### Common AISC Naming Patterns
| Type | Pattern | Examples |
|------|---------|----------|
| Wide Flange | W{depth}x{weight} | W21x44, W14x90, W36x150 |
| Hollow Structural | HSS{width}x{height}x{thick} | HSS6x6x1/4, HSS8x8x5/16 |
| Angle | L{leg1}x{leg2}x{thick} | L4x4x1/4, L6x4x3/8 |
| Channel | C{depth}x{weight} | C8x11.5, C12x20.7 |
| Pipe | P{OD}x{thick} or Pipe{OD}x{thick} | P4.5x0.337 |
| WT (Tee) | WT{depth}x{weight} | WT6x13 |
| Rectangular HSS | HSS{width}x{height}x{thick} | HSS5x3x1/4 |

### NSF_EN (NSF_EN.KZY)
- Nordic Standard / Eurocode-based section library.
- Covers European profiles: HEA, HEB, HEM, IPE, UPN, UPE, etc.

```javascript
var b = Beam(Point(0,0,0), Point(6,0,0));
b.section = Section("HEA300");
b.section = Section("IPE200");
b.section = Section("UPN180");
```

### BS4 Sections (BS4_Sections_Part1_1993.xml)
- British Standard sections: UB, UC, PFC, SHS, RHS, CHS, etc.

```javascript
b.section = Section("UB203x133x25");
b.section = Section("UC152x152x23");
b.section = Section("PFC150x75x18");
```

### Special Section Libraries (XML)
- **anglebar.xml** — angle/L-shaped profiles
- **bulb.xml** — bulb profiles (common in shipbuilding)
- **tbar.xml** — T-bar profiles
- **flatbar.xml** — flat bar profiles

## 3. Using Sections in JS Scripts

### Section("ProfileName") Constructor
```javascript
// Direct by name – GeniE searches all loaded libraries
var sec = Section("W21x44");

// Query available sections (scripting console)
var libs = Section.getAllLibraryNames();
for (var i = 0; i < libs.length; i++) {
    print(libs[i]);
}
```

### Library Selection
- GeniE loads the default library set defined in `Tools > Preferences > Sections`.
- Use `Section.setDefaultLibrary("aisc_v3")` to switch default.
- The `Section()` constructor searches all currently loaded libraries.

## 4. Material Library

### material_library.xml
- Contains predefined material definitions (steel grades, concrete, etc.).
- Path: `{SESAM_HOME}\Program Files\DNV\GeniE V8.8-08\bin\material_library.xml`

```javascript
// Using a material from the library
var mat = Material("NV_36");
// Or create custom material
var customMat = MaterialLinear();
customMat.name = "STEEL_355";
customMat.density = 7850;     // kg/m3
customMat.youngsModulus = 2.1e11; // Pa
customMat.poissonsRatio = 0.3;
customMat.yieldStress = 355e6; // Pa
```

## 5. Corrosion Addition

### corr_add_to_gross_rev3_in.js
- Utility script for adding corrosion allowance to structural sections.
- Located in `{SESAM_HOME}\Program Files\DNV\GeniE V8.8-08\bin\`

```javascript
// Typical usage pattern
var b = Beam(Point(0,0,10), Point(10,0,10));
b.section = Section("W21x44");
b.corrosionAddition = 0.003; // 3 mm corrosion allowance
```

## 6. Programmatic Section Creation

When library sections don't match requirements:

```javascript
// I-section by dimensions
var iSec = ISection();
iSec.height = 0.5;
iSec.widthTop = 0.2;
iSec.widthBottom = 0.2;
iSec.thicknessTop = 0.02;
iSec.thicknessBottom = 0.02;
iSec.thicknessWeb = 0.012;
iSec.material = Material("NV_36");

// Pipe section by dimensions
var pSec = PipeSection();
pSec.outerDiameter = 0.508;
pSec.thickness = 0.0127;
pSec.material = Material("NV_36");

// Box section
var boxSec = BoxSection();
boxSec.height = 0.3;
boxSec.width = 0.3;
boxSec.thicknessTop = 0.012;
boxSec.thicknessBottom = 0.012;
boxSec.thicknessLeft = 0.012;
boxSec.thicknessRight = 0.012;
boxSec.material = Material("NV_36");

// Assign to beam
b.section = iSec;
```

## 7. Default Preferences

Accessible in GeniE GUI: `Edit > Rules > Preferences`
- **Default Section**: sets fallback profile when no section assigned
- **Default Material**: sets fallback material
- **Default Libraries**: controls which KZY/XML files load at startup

Always set defaults at the top of your script to prevent "no section/material defined" errors during modeling operations.
