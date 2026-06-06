# Loads - GeniE SESAM Snippets
> **SESAM 源**: `GeniE V8.8-08 Help\UserDocumentation\CreateLoadCases\CreateLoadCases.htm`

## LoadCase Creation & Gravity

```javascript
// Create a load case
var lc1 = LoadCase();
lc1.name = "DEAD_LOAD";
lc1.includeSelfWeight = true;

// Gravity - global acceleration
lc1.gravity = Vector(0, 0, -9.81);  // m/s^2, negative Z = downward

// Gravity with direction vector (alternative)
lc1.setGravity(Vector(0, 0, -1), 9.81);  // direction + magnitude

// Multiple load cases
var lcDead = LoadCase();
lcDead.name = "LC_DEAD";
lcDead.includeSelfWeight = true;
lcDead.gravity = Vector(0, 0, -9.81);

var lcLive = LoadCase();
lcLive.name = "LC_LIVE";
lcLive.includeSelfWeight = false;
```

## PrismEquipment & Placement

```javascript
// PrismEquipment - represents a rigid equipment item
var eq1 = PrismEquipment();
eq1.name = "TOP_SIDE_MODULE";
eq1.mass = 250000 kg;
eq1.cog = Point(0 m, 0 m, 2.5 m);  // COG relative to equipment reference
eq1.massMomentOfInertia = [150000, 200000, 180000, 0, 0, 0];  // Ixx,Iyy,Izz,Ixy,Ixz,Iyz

// placeAtPoint - place equipment at a single point
eq1.placeAtPoint(Point(10 m, 5 m, 30 m));

// placeBBox - place equipment over a bounding box area
var eq2 = PrismEquipment();
eq2.name = "DECK_EQUIPMENT";
eq2.mass = 50000 kg;
eq2.cog = Point(0 m, 0 m, 1.0 m);

// Bounding box: min point to max point
var pMin = Point(20 m, 10 m, 30 m);
var pMax = Point(26 m, 14 m, 30 m);
eq2.placeBBox(pMin, pMax);  // Distributes load over the box area
```

## WeightList XML Import

```javascript
// Import equipment weights from XML file
var weightList = ImportWeightList("equipment_weights.xml");

// Iterate and create equipment from imported data
for (var i = 0; i < weightList.count(); i++) {
    var item = weightList.item(i);
    var eq = PrismEquipment();
    eq.name = item.name;
    eq.mass = item.mass;
    eq.cog = item.cog;
    eq.placeAtPoint(item.position);
    print("Imported equipment: " + eq.name + " - " + eq.mass + " kg");
}
```

## PointLoad with FootprintPoint & PointForceMoment

```javascript
// PointLoad on a specific location
var pl1 = PointLoad();
pl1.name = "WINCH_LOAD";
pl1.footprint = FootprintPoint(Point(5 m, 3 m, 30 m));

// PointForceMoment defines the force
var pfm1 = PointForceMoment();
pfm1.Fx = 20000 N;
pfm1.Fy = 10000 N;
pfm1.Fz = 0 N;
pfm1.Mx = 0 Nm;
pfm1.My = 0 Nm;
pfm1.Mz = 15000 Nm;
pl1.forceMoment = pfm1;

// Add load to load case
lcLive.add(pl1);

// Vertical load only
var plVert = PointLoad();
plVert.name = "VERTICAL_POINT";
plVert.footprint = FootprintPoint(Point(10 m, 4 m, 30 m));
var pfmVert = PointForceMoment();
pfmVert.Fz = -150000 N;  // Negative = downward
plVert.forceMoment = pfmVert;
lcLive.add(plVert);
```

## LineLoad with FootprintLine & Component1dLinear

```javascript
// LineLoad distributed along a line
var ll1 = LineLoad();
ll1.name = "PIPE_WEIGHT";
ll1.footprint = FootprintLine(Point(0 m, 0 m, 10 m), Point(12 m, 0 m, 10 m));

// Component1dLinear - linearly varying load along the line
var comp1d = Component1dLinear();
comp1d.Fx0 = 0 N/m;       // Fx at start
comp1d.Fx1 = 0 N/m;       // Fx at end
comp1d.Fy0 = 0 N/m;
comp1d.Fy1 = 0 N/m;
comp1d.Fz0 = -5000 N/m;   // Uniform downward load
comp1d.Fz1 = -5000 N/m;
ll1.component = comp1d;

lcLive.add(ll1);

// Triangular distributed load (ramped)
var llRamp = LineLoad();
llRamp.name = "RAMPED_LOAD";
llRamp.footprint = FootprintLine(Point(0 m, 5 m, 10 m), Point(15 m, 5 m, 10 m));
var compRamp = Component1dLinear();
compRamp.Fz0 = 0 N/m;
compRamp.Fz1 = -10000 N/m;  // Linearly increasing downward load
llRamp.component = compRamp;
lcLive.add(llRamp);
```

## SurfaceLoad with FootprintPolygon & Pressure2dConstant

```javascript
// SurfaceLoad on a polygonal area
var slDeck = SurfaceLoad();
slDeck.name = "DECK_PRESSURE";
slDeck.footprint = FootprintPolygon();
slDeck.footprint.addPoint(Point(0 m, 0 m, 30 m));
slDeck.footprint.addPoint(Point(20 m, 0 m, 30 m));
slDeck.footprint.addPoint(Point(20 m, 15 m, 30 m));
slDeck.footprint.addPoint(Point(0 m, 15 m, 30 m));

// Pressure2dConstant - uniform pressure
var pConst = Pressure2dConstant();
pConst.pressure = 5000 Pa;  // 5 kPa uniform pressure
slDeck.pressure = pConst;

lcLive.add(slDeck);

// Pressure on a cylindrical surface
var slTank = SurfaceLoad();
slTank.name = "TANK_HYDROSTATIC";
slTank.footprint = FootprintPolygon();
slTank.footprint.addPoint(Point(5 m, 0 m, 5 m));
slTank.footprint.addPoint(Point(10 m, 0 m, 5 m));
slTank.footprint.addPoint(Point(10 m, 0 m, 10 m));
slTank.footprint.addPoint(Point(5 m, 0 m, 10 m));
slTank.pressure = Pressure2dConstant(15000 Pa);
lcLive.add(slTank);
```

## LoadCombination

```javascript
// Create load combination from individual load cases
var combULS = LoadCombination();
combULS.name = "ULS_COMB_01";

// Add load cases with partial safety factors
combULS.addCase(lcDead, 1.35);   // Permanent load factor
combULS.addCase(lcLive, 1.50);   // Variable load factor
combULS.addCase(lcWind, 1.50);   // Wind included in combination

// Serviceability combination
var combSLS = LoadCombination();
combSLS.name = "SLS_COMB_01";
combSLS.addCase(lcDead, 1.0);
combSLS.addCase(lcLive, 1.0);

// Accident combination
var combALS = LoadCombination();
combALS.name = "ALS_COMB_01";
combALS.addCase(lcDead, 1.0);
combALS.addCase(lcLive, 1.0);
combALS.addCase(lcAccident, 1.0);
```

## Equipment Representation Options

```javascript
// EquipmentAsLineLoads - distribute equipment as line loads
var lcEquipLine = LoadCase();
lcEquipLine.name = "EQUIP_LINE";
lcEquipLine.equipmentRepresentation = EquipmentAsLineLoads;
lcEquipLine.includeSelfWeight = false;

// EquipmentAsConcentratedLoads - distribute equipment as point loads
var lcEquipConc = LoadCase();
lcEquipConc.name = "EQUIP_CONCENTRATED";
lcEquipConc.equipmentRepresentation = EquipmentAsConcentratedLoads;
lcEquipConc.includeSelfWeight = false;

// convertLoadToMass option
var lcDynamic = LoadCase();
lcDynamic.name = "DYNAMIC_LOADS";
lcDynamic.includeSelfWeight = true;
lcDynamic.convertLoadToMass = true;  // Convert loads to masses for dynamic analysis
```

## Self-Weight & Structure Mass

```javascript
// includeSelfWeight - automatic structure self-weight
var lcSW = LoadCase();
lcSW.name = "SELF_WEIGHT";
lcSW.includeSelfWeight = true;
lcSW.gravity = Vector(0, 0, -9.81);

// includeStructureMassWithRotationField - for rotating equipment
var lcRotating = LoadCase();
lcRotating.name = "ROTATING_EQUIPMENT";
lcRotating.includeStructureMassWithRotationField = true;
lcRotating.rotationSpeed = 1500 rpm;
lcRotating.rotationAxis = Vector(0, 0, 1);
```

## Wind Load Case

```javascript
// ConstantWindPressureLoadCase
var lcWind = LoadCase();
lcWind.name = "WIND_LOAD";

// Wind parameters
var windLoad = ConstantWindPressureLoadCase();
windLoad.windSpeed = 45 m/s;             // 10-minute mean wind speed
windLoad.windDirection = Vector(1, 0, 0); // Wind from positive X
windLoad.pressureCoefficient = 1.5;       // Drag coefficient
windLoad.referenceHeight = 10 m;          // Reference height for wind profile

// Apply to structural members
var windPressure = 2500 Pa;  // Equivalent static wind pressure
// Wind load is applied via surface loads on exposed areas
var slWind = SurfaceLoad();
slWind.name = "WIND_X";
slWind.footprint = FootprintPolygon();
slWind.footprint.addPoint(Point(0 m, 0 m, 10 m));
slWind.footprint.addPoint(Point(0 m, 20 m, 10 m));
slWind.footprint.addPoint(Point(0 m, 20 m, 30 m));
slWind.footprint.addPoint(Point(0 m, 0 m, 30 m));
slWind.pressure = Pressure2dConstant(windPressure);
lcWind.add(slWind);
```

## Compartment & Flooding Load

```javascript
// Compartment definition for hydrostatic pressure
var compTank = Compartment();
compTank.name = "BALLAST_TANK";
compTank.addPoint(Point(0 m, 0 m, 0 m));
compTank.addPoint(Point(8 m, 0 m, 0 m));
compTank.addPoint(Point(8 m, 6 m, 0 m));
compTank.addPoint(Point(0 m, 6 m, 0 m));
compTank.height = 10 m;
compTank.fillLevel = 8 m;        // 80% filled
compTank.fluidDensity = 1025 kg/m^3;  // Seawater

// Flooding - applied to load case
var lcFlood = LoadCase();
lcFlood.name = "FLOODING";
var flooding = Flooding();
flooding.compartment = compTank;
flooding.pressureType = "hydrostatic";
lcFlood.add(flooding);
```