# Compartment & Tank Loads - GeniE SESAM Snippets
> **SESAM 源**: `GeniE V8.8-08 Help\UserDocumentation\CreateCompartment\Compartment.htm`

## CompartmentManager Basics

```javascript
// CompartmentManager is the central object for all compartment operations
var cm = CompartmentManager();

// Set the compartmentation method - topology-based is most common
var method = CompartmentationMethod();
method.type = "TopologyBased";   // Compartments defined by bounding structure
method.autoDetectBoundary = true; // Auto-detect walls/decks/bulkheads
cm.compartmentationMethod = method;

// Material densities for common fluids
var rhoSeawater = 1025 kg/m^3;
var rhoFreshwater = 1000 kg/m^3;
var rhoDiesel = 850 kg/m^3;
var rhoCrudeOil = 870 kg/m^3;
var rhoBallast = 1025 kg/m^3;
```

## Liquid Content Compartment (Fuel Tank 80% Fill)

```javascript
var cm = CompartmentManager();

// Define fuel tank compartment boundaries
var fuelTank = cm.createCompartment("FUEL_TANK_PORT");
fuelTank.addBoundaryPoint(Point(5 m, 0 m, 2 m));
fuelTank.addBoundaryPoint(Point(12 m, 0 m, 2 m));
fuelTank.addBoundaryPoint(Point(12 m, 8 m, 2 m));
fuelTank.addBoundaryPoint(Point(5 m, 8 m, 2 m));
fuelTank.topZ = 10 m;              // Top of compartment
fuelTank.bottomZ = 2 m;            // Bottom of compartment
fuelTank.isClosed = true;

// Create liquid content load - 80% fill of diesel fuel
var fuelLoad = CompartmentLoads();
fuelLoad.name = "FUEL_TANK_80PCT";
fuelLoad.compartment = fuelTank;

// ContentLiquid defines the liquid properties and fill level
fuelLoad.content = ContentLiquid(rhoDiesel, 80);  // 80% fill percentage

// Alternative: ContentLiquid with absolute fill height
// fuelLoad.content = ContentLiquid(rhoDiesel, 6.4 m);  // Absolute 6.4m fill height

// Add to a load case
var lcFuel = LoadCase();
lcFuel.name = "LC_FUEL_TANK";
lcFuel.includeSelfWeight = false;
lcFuel.addCompartmentLoad(fuelLoad);

print("Fuel tank load created: " + lcFuel.name);
```

## Ballast Tank Flooding (Damage Stability)

```javascript
var cm = CompartmentManager();

// Ballast tank definition
var ballastTank = cm.createCompartment("BALLAST_TANK_STBD");
ballastTank.addBoundaryPoint(Point(15 m, 0 m, 0 m));
ballastTank.addBoundaryPoint(Point(22 m, 0 m, 0 m));
ballastTank.addBoundaryPoint(Point(22 m, 10 m, 0 m));
ballastTank.addBoundaryPoint(Point(15 m, 10 m, 0 m));
ballastTank.topZ = 15 m;
ballastTank.bottomZ = 0 m;
ballastTank.isClosed = true;

// Full flooding - compartment completely filled with seawater
var floodLoad = CompartmentLoads();
floodLoad.name = "FLOODING_BALLAST";
floodLoad.compartment = ballastTank;
floodLoad.content = ContentLiquid(rhoSeawater, 100);  // 100% filled = fully flooded

// Flooding load case for damage stability analysis
var lcFlood = LoadCase();
lcFlood.name = "LC_FLOODING_STBD";
lcFlood.includeSelfWeight = true;
lcFlood.gravity = Vector(0, 0, -9.81);

// Flooding object wraps the compartment load
var flooding = Flooding();
flooding.compartment = ballastTank;
flooding.fillPercentage = 100;
flooding.fluidDensity = rhoSeawater;
flooding.pressureType = "Hydrostatic";
flooding.includeDynamicPressure = false;  // Quasi-static flooding

lcFlood.add(flooding);

print("Damage flooding case created for: " + ballastTank.name);
```

## Solid Content Compartment (Bulk Cargo Hold)

```javascript
var cm = CompartmentManager();

// Cargo hold definition
var cargoHold = cm.createCompartment("CARGO_HOLD_01");
cargoHold.addBoundaryPoint(Point(0 m, 5 m, 5 m));
cargoHold.addBoundaryPoint(Point(30 m, 5 m, 5 m));
cargoHold.addBoundaryPoint(Point(30 m, 25 m, 5 m));
cargoHold.addBoundaryPoint(Point(0 m, 25 m, 5 m));
cargoHold.topZ = 20 m;
cargoHold.bottomZ = 5 m;
cargoHold.isClosed = false;   // Open top compartment
cargoHold.name = "HOLD_01";

// Solid content - bulk cargo (iron ore)
var solidLoad = CompartmentLoads();
solidLoad.name = "IRON_ORE_CARGO";
solidLoad.compartment = cargoHold;

// ContentSolid applies uniform pressure from bulk material
// Uses active earth pressure coefficient Ka internally
solidLoad.content = ContentSolid(2500 kg/m^3);  // Iron ore density
solidLoad.fillHeight = 10 m;                     // Filled to 10m from bottom
solidLoad.angleOfRepose = 35 deg;                // Internal friction angle
solidLoad.wallFrictionAngle = 20 deg;             // Wall friction

var lcCargo = LoadCase();
lcCargo.name = "LC_CARGO_FULL";
lcCargo.includeSelfWeight = true;
lcCargo.gravity = Vector(0, 0, -9.81);
lcCargo.addCompartmentLoad(solidLoad);

// Alternative lighter bulk cargo (grain)
var grainLoad = CompartmentLoads();
grainLoad.name = "GRAIN_CARGO";
grainLoad.compartment = cargoHold;
grainLoad.content = ContentSolid(800 kg/m^3);
grainLoad.fillHeight = 12 m;
grainLoad.angleOfRepose = 25 deg;

var lcGrain = LoadCase();
lcGrain.name = "LC_GRAIN";
lcGrain.addCompartmentLoad(grainLoad);
```

## Manual Pressure Definition (Custom P(x,y,z))

```javascript
var cm = CompartmentManager();

// Compartment for manual pressure
var customTank = cm.createCompartment("CUSTOM_PRESSURE_TANK");
customTank.addBoundaryPoint(Point(0 m, 0 m, 0 m));
customTank.addBoundaryPoint(Point(10 m, 0 m, 0 m));
customTank.addBoundaryPoint(Point(10 m, 10 m, 0 m));
customTank.addBoundaryPoint(Point(0 m, 10 m, 0 m));
customTank.topZ = 8 m;
customTank.bottomZ = 0 m;
customTank.isClosed = true;

// Manual pressure definition as a function of position
var manualLoad = CompartmentLoads();
manualLoad.name = "MANUAL_PRESSURE";
manualLoad.compartment = customTank;

// Define custom pressure function P(x, y, z) in Pa
// Example: linearly increasing pressure with depth (z) plus x-gradient
function customPressure(x, y, z) {
    var hydrostatic = 1025 * 9.81 * (8 - z);  // Hydrostatic from top (z=8m)
    var surgeHead = 5000 * (x / 10);           // Linear surge gradient in X
    return hydrostatic + surgeHead;
}

manualLoad.definePressure(customPressure);

var lcCustom = LoadCase();
lcCustom.name = "LC_CUSTOM_TANK";
lcCustom.addCompartmentLoad(manualLoad);

print("Custom pressure function applied to: " + customTank.name);
```

## DipoleSheet Method for Hydrostatic Pressure

```javascript
var cm = CompartmentManager();

// DipoleSheet - efficient method for hydrostatic pressure distribution
// Creates a zero-thickness pressure sheet at the liquid surface
var tank = cm.createCompartment("DIPOLESHEET_TANK");
tank.addBoundaryPoint(Point(0 m, 0 m, 0 m));
tank.addBoundaryPoint(Point(8 m, 0 m, 0 m));
tank.addBoundaryPoint(Point(8 m, 6 m, 0 m));
tank.addBoundaryPoint(Point(0 m, 6 m, 0 m));
tank.topZ = 12 m;
tank.bottomZ = 0 m;

var dipoleLoad = CompartmentLoads();
dipoleLoad.name = "DIPOLE_HYDROSTATIC";
dipoleLoad.compartment = tank;
dipoleLoad.content = ContentLiquid(rhoSeawater, 75);  // 75% fill

// Enable DipoleSheet method for faster pressure computation
dipoleLoad.pressureMethod = "DipoleSheet";
dipoleLoad.freeSurfaceZ = 9 m;  // Z-coordinate of liquid free surface

var lcDipole = LoadCase();
lcDipole.name = "LC_DIPOLE_TANK";
lcDipole.addCompartmentLoad(dipoleLoad);

print("DipoleSheet pressure method used for: " + tank.name);
```

## Non-Watertight Bulkheads & Open Compartments

```javascript
var cm = CompartmentManager();

// Open compartment - top is open to atmosphere
var openTank = cm.createCompartment("OPEN_TOP_TANK");
openTank.addBoundaryPoint(Point(0 m, 0 m, 0 m));
openTank.addBoundaryPoint(Point(10 m, 0 m, 0 m));
openTank.addBoundaryPoint(Point(10 m, 8 m, 0 m));
openTank.addBoundaryPoint(Point(0 m, 8 m, 0 m));
openTank.bottomZ = 0 m;
openTank.topZ = 10 m;
openTank.isClosed = false;          // Open compartment - no top boundary
openTank.name = "MOONPOOL";

// Water load in open compartment (connected to sea)
var moonpoolLoad = CompartmentLoads();
moonpoolLoad.name = "MOONPOOL_WATER";
moonpoolLoad.compartment = openTank;
moonpoolLoad.content = ContentLiquid(rhoSeawater, 5 m);  // Water column of 5m
moonpoolLoad.useExternalPressure = true;  // Pressure from external sea level

// NonWatertightBulkheads - partially bounded compartment
var partialTank = cm.createCompartment("PARTIAL_BOUNDARY_TANK");
partialTank.addBoundaryPoint(Point(10 m, 0 m, 2 m));
partialTank.addBoundaryPoint(Point(18 m, 0 m, 2 m));
partialTank.addBoundaryPoint(Point(18 m, 6 m, 2 m));
partialTank.addBoundaryPoint(Point(10 m, 6 m, 2 m));
partialTank.topZ = 10 m;
partialTank.bottomZ = 2 m;

// Define non-watertight boundaries (perforated/swash bulkheads)
var nonWT = NonWatertightBulkheads();
nonWT.addBoundary("side_bulkhead_stbd");   // Starboard side is non-watertight
nonWT.permeability = 0.3;                  // 30% permeable (swash bulkhead)
partialTank.nonWatertightBoundary = nonWT;

var partialLoad = CompartmentLoads();
partialLoad.name = "SWASH_BULKHEAD_LOAD";
partialLoad.compartment = partialTank;
partialLoad.content = ContentLiquid(rhoSeawater, 60);
partialLoad.reductionFactor = 0.7;  // Pressure reduced due to permeability

var lcPartial = LoadCase();
lcPartial.name = "LC_PARTIAL_TANK";
lcPartial.addCompartmentLoad(partialLoad);
```

## Corrosion Addition & ModifyStructure

```javascript
var cm = CompartmentManager();

// CorrosionAdditionCompartments - add corrosion margin to compartment boundaries
var ballastTank = cm.getCompartment("BALLAST_TANK_STBD");
var ca = CorrosionAdditionCompartments();
ca.compartment = ballastTank;
ca.corrosionAddition = 3 mm;       // 3mm corrosion allowance on all boundaries
ca.sideShellAddition = 4 mm;       // 4mm on side shell
ca.bottomShellAddition = 4 mm;     // 4mm on bottom shell
ca.deckAddition = 2 mm;            // 2mm on deck

cm.applyCorrosionAddition(ca);
print("Corrosion addition applied to: " + ballastTank.name);

// ModifyStructure - add or remove compartment boundary plates
var newTank = cm.createCompartment("MODIFIED_TANK");
newTank.addBoundaryPoint(Point(20 m, 0 m, 0 m));
newTank.addBoundaryPoint(Point(28 m, 0 m, 0 m));
newTank.addBoundaryPoint(Point(28 m, 10 m, 0 m));
newTank.addBoundaryPoint(Point(20 m, 10 m, 0 m));
newTank.topZ = 12 m;
newTank.bottomZ = 0 m;

// Add an internal bulkhead to split the tank
var mod = ModifyStructure();
mod.compartment = newTank;

// Add a new boundary plate at Y=5m (longitudinal bulkhead)
var internalBhd = BoundaryPlate();
internalBhd.point1 = Point(20 m, 5 m, 0 m);
internalBhd.point2 = Point(28 m, 5 m, 0 m);
internalBhd.point3 = Point(28 m, 5 m, 12 m);
internalBhd.point4 = Point(20 m, 5 m, 12 m);
internalBhd.name = "INTERNAL_LONG_BHD";
mod.addBoundary(internalBhd);

// Remove a boundary
// mod.removeBoundary("side_bhd_port");
cm.applyModifyStructure(mod);
```

## VisualizeAndRename & Multiple Compartment Loading

```javascript
var cm = CompartmentManager();

// VisualizeAndRename - review compartment geometry and rename
var tankList = cm.getAllCompartments();
for (var i = 0; i < tankList.length; i++) {
    var tank = tankList[i];
    // Visualize the compartment for verification
    tank.VisualizeAndRename(tank.name + "_REVIEWED");
    print("Compartment verified: " + tank.name +
          " | Status: " + (tank.isClosed ? "Closed" : "Open"));
}

// Multiple compartments loaded simultaneously
// Ballast tanks on both sides with different fill levels
var tankPort = cm.createCompartment("BALLAST_PORT");
tankPort.addBoundaryPoint(Point(0 m, 0 m, 2 m));
tankPort.addBoundaryPoint(Point(5 m, 0 m, 2 m));
tankPort.addBoundaryPoint(Point(5 m, 10 m, 2 m));
tankPort.addBoundaryPoint(Point(0 m, 10 m, 2 m));
tankPort.topZ = 12 m;
tankPort.bottomZ = 2 m;

var tankStbd = cm.createCompartment("BALLAST_STBD");
tankStbd.addBoundaryPoint(Point(5 m, 0 m, 2 m));
tankStbd.addBoundaryPoint(Point(10 m, 0 m, 2 m));
tankStbd.addBoundaryPoint(Point(10 m, 10 m, 2 m));
tankStbd.addBoundaryPoint(Point(5 m, 10 m, 2 m));
tankStbd.topZ = 12 m;
tankStbd.bottomZ = 2 m;

// Load port tank 90%, stbd tank 30% - asymmetric loading condition
var loadPort = CompartmentLoads();
loadPort.name = "BALLAST_PORT_90";
loadPort.compartment = tankPort;
loadPort.content = ContentLiquid(rhoSeawater, 90);

var loadStbd = CompartmentLoads();
loadStbd.name = "BALLAST_STBD_30";
loadStbd.compartment = tankStbd;
loadStbd.content = ContentLiquid(rhoSeawater, 30);

// Combine in one load case
var lcAsymmetric = LoadCase();
lcAsymmetric.name = "LC_BALLAST_ASYM";
lcAsymmetric.includeSelfWeight = true;
lcAsymmetric.gravity = Vector(0, 0, -9.81);
lcAsymmetric.addCompartmentLoad(loadPort);
lcAsymmetric.addCompartmentLoad(loadStbd);

// Fuel oil tanks simultaneously loaded
var fuelFwd = cm.createCompartment("FUEL_FWD");
fuelFwd.addBoundaryPoint(Point(3 m, 2 m, 5 m));
fuelFwd.addBoundaryPoint(Point(7 m, 2 m, 5 m));
fuelFwd.addBoundaryPoint(Point(7 m, 8 m, 5 m));
fuelFwd.addBoundaryPoint(Point(3 m, 8 m, 5 m));
fuelFwd.topZ = 10 m;
fuelFwd.bottomZ = 5 m;

var fuelAft = cm.createCompartment("FUEL_AFT");
fuelAft.addBoundaryPoint(Point(3 m, 2 m, 10 m));
fuelAft.addBoundaryPoint(Point(7 m, 2 m, 10 m));
fuelAft.addBoundaryPoint(Point(7 m, 8 m, 10 m));
fuelAft.addBoundaryPoint(Point(3 m, 8 m, 10 m));
fuelAft.topZ = 15 m;
fuelAft.bottomZ = 10 m;

var loadFuelFwd = CompartmentLoads();
loadFuelFwd.name = "FUEL_FWD_95";
loadFuelFwd.compartment = fuelFwd;
loadFuelFwd.content = ContentLiquid(rhoDiesel, 95);

var loadFuelAft = CompartmentLoads();
loadFuelAft.name = "FUEL_AFT_70";
loadFuelAft.compartment = fuelAft;
loadFuelAft.content = ContentLiquid(rhoDiesel, 70);

// All loads in one operating condition
var lcOperating = LoadCase();
lcOperating.name = "LC_OPERATING_CONDITION";
lcOperating.includeSelfWeight = true;
lcOperating.gravity = Vector(0, 0, -9.81);
lcOperating.addCompartmentLoad(loadPort);
lcOperating.addCompartmentLoad(loadStbd);
lcOperating.addCompartmentLoad(loadFuelFwd);
lcOperating.addCompartmentLoad(loadFuelAft);

print("Operating condition created with " +
      lcOperating.compartmentLoadCount() + " compartment loads");
```
