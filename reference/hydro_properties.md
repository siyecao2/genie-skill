# Hydrodynamic & Environmental Properties

GeniE supports comprehensive hydrodynamic and environmental property assignment for wave load analysis (Wadam/Wajac integration). All hydro properties are documented in `Help\UserDocumentation\Properties\Hydro\`.

## Property Hierarchy

```
Property
 ├── Morison ──────────────────── Wave load coefficients
 │    ├── MorisonCoefficients ── Direct Cd/Cm input
 │    ├── MorisonByRule ──────── API rule-based coefficients
 │    ├── MorisonDiameterFunction ─── Diameter-dependent Cd/Cm
 │    ├── MorisonGlobalDirection ─── Direction/seastate-dependent Cd/Cm
 │    ├── MorisonRoughnessKCFunction ── KC & roughness dependent
 │    └── MorisonRoughnessReynoldFunction ── Re & roughness dependent
 ├── AirDrag ─────────────────── Wind load drag coefficients
 │    ├── AirDragConstant ────── Fixed Cd value
 │    └── AirDragReynoldFunction ── Re-dependent Cd
 ├── MarineGrowth ───────────── Marine fouling
 │    ├── MarineGrowthConstant ── Constant thickness & density
 │    └── MarineGrowthZLevelFunction ── Z-dependent growth
 ├── HydroDynamicDiameter ───── Override hydrodynamic diameter
 ├── HydroBuoyancyArea ─────── Custom buoyancy area
 ├── Flooding ───────────────── Flooding ratio (0=empty, 1=full)
 ├── ElementRefinement ─────── Hydro mesh element subdivision
 └── ConductorShielding ────── Conductor array shielding effects
```

## Morison Coefficients

Morison properties define drag (Cd) and inertia (Cm) coefficients for wave load calculations on beams/segments.

### MorisonConstant / MorisonCoefficients — Direct Input

```javascript
// Cd_normal=0, Cd_tangential=0.7, Cd_lift=0.7, Cm_normal=0, Cm_tangential=2, Cm_lift=2
// Parameter order: Cd_n, Cd_t, Cd_l, Cm_n, Cm_t, Cm_l
MyMorison = MorisonCoefficients(0, 0.7, 0.7, 0, 2, 2);

// Assign to a beam
Beam1.morison = MyMorison;

// Assign to a set of beams
Legs.morison = MyMorison;
```

### MorisonByRule — API Rule-Based

```javascript
// Use API RP 2A recommended Morison coefficients based on member diameter
// Coefficients are automatically selected per member
MyAPIMorison = MorisonByRule();
ConductorBay.morison = MyAPIMorison;
```

### MorisonDiameterFunction — Diameter-Dependent Coefficients

```javascript
// For 2 diameter values: at D=2m → Cd=1.0, Cm=2.0; at D=3m → Cd=0.9, Cm=2.1
// Parameters: (Array(diameters...), Array(Cd_values...), Array(Cm_values...))
MyMorisonDiameter = MorisonDiameterFunction(
    Array(2, 3),
    Array(1, 0.9),
    Array(2, 2.1)
);
```

### MorisonGlobalDirection — Direction & Seastate Dependent

```javascript
// For 2 directions/seastates:
// Direction 1 (0°), Seastate 1: Cd_n=0, Cd_t=0.7, Cd_l=0.7, Cm_n=0, Cm_t=2, Cm_l=2
// Direction 2 (0°), Seastate 4: Cd_n=0, Cd_t=0.9, Cd_l=0.9, Cm_n=0, Cm_t=2.5, Cm_l=4
MyMorisonGlobal = MorisonGlobalDirection(
    Array(0, 0),           // Cd_normal per direction/seastate
    Array(0.7, 0.9),       // Cd_tangential
    Array(0.7, 0.9),       // Cd_lift
    Array(0, 0),           // Cm_normal
    Array(2, 2.5),         // Cm_tangential
    Array(2, 4),           // Cm_lift
    Array(0 deg, 0 deg),   // Wave directions
    Array(1, 4)            // Seastate indices
);
```

### MorisonKC — KC Number Dependent

`MorisonRoughnessKCFunction` defines coefficients as a function of Keulegan-Carpenter number and surface roughness. Used when wake amplification effects are significant (e.g., slender members in large waves).

### MorisonReynolds — Reynolds Number Dependent

`MorisonRoughnessReynoldFunction` defines coefficients as a function of Reynolds number and surface roughness. Used for high-Re flows where drag crisis effects matter.

## Air Drag Coefficients

Air drag properties for wind load calculations above the water surface.

### AirDragConstant — Fixed Coefficient

```javascript
// Cd=1.0 (normal direction), Cd=0.65 (tangential direction)
MyAirDrag = AirDragConstant(1, 0.65);

// Assign to a set of above-water members
Topside.mairDrag = MyAirDrag;
```

### Reynolds-Dependent Air Drag

`AirDragReynoldFunction` provides Cd values that vary with Reynolds number — important for large-diameter risers or flare booms where Re effects are significant.

```javascript
// AirDrag class hierarchy:
// AirDrag → AirDragConstant (fixed Cd)
//        → AirDragReynoldFunction (Re-dependent Cd)
```

## Buoyancy Area Specification

Override the default buoyancy area for beam elements (default uses structural cross-section area).

```javascript
// BuoyancyArea is a read-only property representing buoyancyAreaFromStructure()
MyBuoyancy = BuoyancyArea;
Beam1.buoyancyArea = MyBuoyancy;  // Use structural section area

// Custom buoyancy area for flooded/non-flooded conditions
MyBuoancyCustom = HydroBuoyancyArea(0.75 m^2, 0.15 m^2);
// Parameters: (nonFloodedArea, floodedArea) — or (Area, Area)
Beam1.buoyancyArea = MyBuoancyCustom;
```

## Conductor Shielding Effects

Model the shielding effect of conductor arrays on wave loading. When conductors are closely spaced, downstream members experience reduced wave forces.

```javascript
// Define a plate/shell as permeable (non-watertight) — used for conductor shielding
MyPermeable = Permeable(true);
// Assign to plates that should be ignored during compartment generation
// but still receive loads
ConductorPlate.permeable = MyPermeable;
```

Note: The `ConductorShielding` class description in GeniE's JS documentation states it is for "indicating that a plate is not watertight" — this is used in compartment load generation to ignore certain plates.

## Marine Growth

Marine growth adds effective diameter and mass to submerged members, affecting wave loading and structural weight.

### MarineGrowthConstant — Constant Thickness

```javascript
// thickness=20cm, density=1cm, Cd_increase_factor=2
// Parameters: (thickness, roughness, densityFactorOrCdFactor)
MyMarineGrowth = MarineGrowthConstant(20 cm, 1 cm, 2);
MyMarineGrowth.useInForceCalculations(false); // Exclude from force calcs if needed

// Assign to jacket members
JacketSubmerged.marineGrowth = MyMarineGrowth;
```

Typical marine growth parameters for offshore structures:
| Location | Thickness | Density |
|----------|-----------|---------|
| North Sea (top 40m) | 50-100 mm | 1.0-1.4 t/m³ |
| Gulf of Mexico | 25-50 mm | 1.0-1.3 t/m³ |
| Tropical waters | 100-200 mm | 1.1-1.4 t/m³ |

### Z-Level Dependent Marine Growth

`MarineGrowthZLevelFunction` allows thickness and density to vary with depth (e.g., thicker growth near surface, thinner at depth).

## Hydrodynamic Diameter

For non-circular sections or members with coatings, override the default hydrodynamic diameter used in Morison equation:

```javascript
// Set explicit hydrodynamic diameter of 0.75 m
MyHydroDiameter = HydroDynamicDiameter(0.75 m);
NonCircularMember.hydroDynamicDiameter = MyHydroDiameter;
```

This is critical for:
- Rectangular hollow sections (RHS) — provide equivalent circular diameter
- Built-up sections with non-circular envelopes
- Members with anodes, caissons, or attachment pipes

## Flooding Status

Define member flooding ratio for damaged condition analysis (0 = not flooded, 1 = fully flooded):

```javascript
// Fully flooded member
MyFlooding = Flooding(1);
DamagedBrace.flooding = MyFlooding;

// 50% flooded
PartialFlood = Flooding(0.5);
```

Flooding affects:
- Member buoyancy (added mass of entrained water)
- Wave loading (modified hydrodynamic mass)
- Structural mass (water weight inside flooded members)

## Element Refinement for Hydrodynamic Mesh

Control the subdivision of beam elements for hydrodynamic analysis — higher refinement gives more accurate distributed wave loads:

```javascript
// Subdivide each beam element into 5 sub-elements for hydro analysis
MyRefinement = ElementRefinement(5);
Riser.beamRefinement = MyRefinement;
```

Higher refinement values are recommended for:
- Members near the water surface (wave zone)
- Long members experiencing significant load variation
- Members with rapidly varying marine growth or hydrodynamic diameter

## Wet Surface Specification

`FootprintWetSurface` identifies which element surfaces are exposed to water — used for panel-based hydro loading (diffraction analysis in Wadam):

```javascript
// Define wet surface on a panel
WetPanel.wetSurface = FootprintWetSurface();
```

## Integration with Wadam/Wajac Wave Load Analysis

Hydro properties feed directly into the wave load analysis workflow:

```javascript
// 1. Define Morison properties
JacketMorison = MorisonByRule();

// 2. Define marine growth
MgNorthSea = MarineGrowthConstant(80 mm, 1 cm, 2);

// 3. Assign to structural sets
Legs.morison = JacketMorison;
Legs.marineGrowth = MgNorthSea;
Braces.morison = MorisonCoefficients(0, 0.7, 0.7, 0, 2, 2);
Braces.marineGrowth = MgNorthSea;

// 4. Define air drag for above-water structure
TopsideAirDrag = AirDragConstant(1.2, 0.8);
TopsideBeams.airDrag = TopsideAirDrag;

// 5. Define flooding for damaged legs
LegA.flooding = Flooding(1);   // Fully flooded
LegB.flooding = Flooding(0);   // Dry

// 6. Create wave load condition
WaveCondition = WaveLoadCondition("DesignWave");
WaveCondition.loadCase = LC_Wave;

// 7. Create Wadam/Wajac analysis
WaveAnalysis = Analysis("WaveAnalysis");
// The hydro properties on members are automatically picked up
```

## Typical Offshore Platform Hydro Assignment

```javascript
// Jacket hydro property assignment
// Splash zone: +5m to -5m
SplashZone = Set("SplashZone");
// Submerged zone: -5m to mudline
SubmergedZone = Set("SubmergedZone");
// Atmospheric zone: above +5m
AtmosphericZone = Set("AtmosphericZone");

// Morison coefficients (API RP 2A recommended)
APIMorison = MorisonByRule();
SplashZone.morison = APIMorison;
SubmergedZone.morison = APIMorison;

// Marine growth (North Sea)
SplashMg = MarineGrowthConstant(100 mm, 1 cm, 2);
SubMg = MarineGrowthConstant(60 mm, 1 cm, 2);
SplashZone.marineGrowth = SplashMg;
SubmergedZone.marineGrowth = SubMg;

// Air drag (above water only)
WindDrag = AirDragConstant(1.0, 0.65);
AtmosphericZone.airDrag = WindDrag;

// Hydro element refinement for splash zone
SplashRefine = ElementRefinement(8);
SplashZone.beamRefinement = SplashRefine;

// Flooded members for damaged case
DamagedMembers = Set("Damaged");
DamagedMembers.flooding = Flooding(1);
```
