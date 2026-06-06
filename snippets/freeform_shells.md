# Freeform Shells & Hull Surfaces - GeniE SESAM Snippets

## SkinCurves - Lofting Through Guide Curves

```javascript
// Create a set of guide curves for lofting a hull panel
var curveBottom = ModelCurve();
curveBottom.addPoint(Point(0 m, 0 m, 0 m));
curveBottom.addPoint(Point(3 m, 0 m, 0.5 m));
curveBottom.addPoint(Point(6 m, 0 m, 0 m));
curveBottom.name = "BOTTOM_PROFILE";

var curveMiddle = ModelCurve();
curveMiddle.addPoint(Point(0 m, 4 m, 1.5 m));
curveMiddle.addPoint(Point(3 m, 4 m, 2.5 m));
curveMiddle.addPoint(Point(6 m, 4 m, 1.5 m));
curveMiddle.name = "MIDDLE_PROFILE";

var curveTop = ModelCurve();
curveTop.addPoint(Point(0 m, 8 m, 3 m));
curveTop.addPoint(Point(3 m, 8 m, 4 m));
curveTop.addPoint(Point(6 m, 8 m, 3 m));
curveTop.name = "TOP_PROFILE";

// SkinCurves - loft a surface through an array of guide curves
var hullPanel = SkinCurves([curveBottom, curveMiddle, curveTop]);
hullPanel.thickness = 18 mm;
hullPanel.material = DH36;
hullPanel.name = "HULL_PANEL_SKIN";

// SkinCurves with more sections for better control
var curve1 = ModelCurve();
curve1.addPoint(Point(0 m, 0 m, 0 m));
curve1.addPoint(Point(10 m, 0 m, 0 m));
curve1.name = "RAIL_1";

var curve2 = ModelCurve();
curve2.addPoint(Point(0 m, 5 m, 3 m));
curve2.addPoint(Point(10 m, 5 m, 3 m));
curve2.name = "RAIL_2";

var curve3 = ModelCurve();
curve3.addPoint(Point(0 m, 10 m, 4 m));
curve3.addPoint(Point(10 m, 10 m, 4 m));
curve3.name = "RAIL_3";

var curve4 = ModelCurve();
curve4.addPoint(Point(0 m, 15 m, 2 m));
curve4.addPoint(Point(10 m, 15 m, 2 m));
curve4.name = "RAIL_4";

var bowPanel = SkinCurves([curve1, curve2, curve3, curve4]);
bowPanel.thickness = 22 mm;
bowPanel.material = DH36;
bowPanel.name = "BOW_PANEL_4RAIL";
```

## SkinLoftCurves - Advanced Lofting

```javascript
// SkinLoftCurves with lofting options for more control
var guideCurves = [];
for (var i = 0; i < 6; i++) {
    var z = i * 2 m;
    var yOffset = Math.sin(i * 0.5) * 2 m;
    var curve = ModelCurve();
    curve.addPoint(Point(0 m, yOffset, z));
    curve.addPoint(Point(4 m, yOffset + 0.5 m, z));
    curve.addPoint(Point(8 m, yOffset, z));
    curve.name = "SECTION_" + i;
    guideCurves.push(curve);
}

var loftedHull = SkinLoftCurves(guideCurves);
loftedHull.loftOptions = {
    interpolationType: "Cubic",     // Cubic interpolation between sections
    preserveShape: true,             // Maintain curve shapes
    alignCurveDirections: true,      // Auto-align curve parameterization
    surfaceDegree: 3                 // Cubic NURBS surface
};
loftedHull.thickness = 20 mm;
loftedHull.material = VL36;
loftedHull.name = "LOFTED_HULL_SURFACE";
```

## SweepAProfileAlongATrajectory

```javascript
// Define a profile curve (stiffener cross-section shape)
var profileCurve = ModelCurve();
profileCurve.addPoint(Point(0 m, 0 m, 0 m));          // Bottom center
profileCurve.addPoint(Point(0.15 m, 0 m, 0 m));       // Bottom right
profileCurve.addPoint(Point(0.15 m, 0.3 m, 0 m));     // Vertical web
profileCurve.addPoint(Point(-0.15 m, 0.3 m, 0 m));    // Top flange left
profileCurve.addPoint(Point(-0.15 m, 0 m, 0 m));      // Bottom left
profileCurve.name = "T_PROFILE";

// Define a trajectory along deck edge
var trajectory = ModelCurve();
trajectory.addPoint(Point(0 m, 0 m, 20 m));
trajectory.addPoint(Point(5 m, 2 m, 20 m));
trajectory.addPoint(Point(10 m, 5 m, 20.5 m));
trajectory.addPoint(Point(15 m, 8 m, 21 m));
trajectory.addPoint(Point(20 m, 13 m, 22 m));
trajectory.name = "DECK_EDGE_PATH";

// SweepAProfileAlongATrajectory - extrude profile along 3D path
var deckStringer = SweepAProfileAlongATrajectory(profileCurve, trajectory);
deckStringer.thickness = 12 mm;
deckStringer.material = S355;
deckStringer.name = "DECK_STRINGER_SWEPT";

// Pipe as trajectory for curved pipe sweep
var pipeProfile = ModelCurve();
pipeProfile.addPoint(Point(0 m, 0.3 m, 0 m));   // Circle approximated
pipeProfile.addPoint(Point(0.3 m, 0 m, 0 m));
pipeProfile.name = "PIPE_PROFILE";

var pipePath = ModelCurve();
pipePath.curveType = "Spline";
pipePath.addPoint(Point(0 m, 0 m, 5 m));
pipePath.addPoint(Point(2 m, 3 m, 8 m));
pipePath.addPoint(Point(5 m, 5 m, 12 m));
pipePath.addPoint(Point(8 m, 3 m, 15 m));
pipePath.name = "RISER_PATH";

var curvedRiser = SweepAProfileAlongATrajectory(pipeProfile, pipePath);
curvedRiser.name = "CURVED_RISER";
```

## SweepExtrudeAProfileAlongAVector

```javascript
// Simple profile for a bulb flat stiffener
var bulbProfile = ModelCurve();
bulbProfile.addPoint(Point(0 m, 0 m, 0 m));
bulbProfile.addPoint(Point(0.012 m, 0 m, 0 m));
bulbProfile.addPoint(Point(0.012 m, 0.2 m, 0 m));
bulbProfile.addPoint(Point(0.02 m, 0.2 m, 0 m));    // Bulb
bulbProfile.addPoint(Point(0.0 m, 0.22 m, 0 m));
bulbProfile.name = "BULB_FLAT_PROFILE";

// Linear extrusion along a vector for a specified length
var stiffener1 = SweepExtrudeAProfileAlongAVector(
    bulbProfile,
    Vector(0, 1, 0),    // Extrude in Y direction
    8 m                   // Length of extrusion
);
stiffener1.name = "BULB_STIFFENER_H8M";

// Array of longitudinal stiffeners
var stiffeners = [];
var spacing = 0.8 m;
for (var i = 0; i < 10; i++) {
    var yPos = i * spacing;
    // Position the profile at start point before extrusion
    var positionedProfile = bulbProfile.copyTranslate(Vector(0, yPos, 15 m));
    var stf = SweepExtrudeAProfileAlongAVector(
        positionedProfile,
        Vector(1, 0, 0),    // Extrude in X direction
        20 m                  // Full length
    );
    stf.name = "LONG_STIFFENER_" + (i + 1);
    stf.thickness = 10 mm;
    stf.material = S355;
    stiffeners.push(stf);
}
print("Created " + stiffeners.length + " longitudinal stiffeners");
```

## CurveNetInterpolation & PointNetFitting

```javascript
// CurveNetInterpolation - surface from intersecting curve network
// Create curves in U direction (longitudinal)
var uCurves = [];
for (var i = 0; i < 4; i++) {
    var uc = ModelCurve();
    uc.addPoint(Point(0 m, i * 3 m, 0 m));
    uc.addPoint(Point(5 m, i * 3 m, 1.5 m));
    uc.addPoint(Point(10 m, i * 3 m, 3 m));
    uc.addPoint(Point(15 m, i * 3 m, 1 m));
    uc.addPoint(Point(20 m, i * 3 m, 0 m));
    uc.name = "U_CURVE_" + i;
    uCurves.push(uc);
}

// Create curves in V direction (transverse)
var vCurves = [];
for (var j = 0; j < 5; j++) {
    var vc = ModelCurve();
    vc.addPoint(Point(j * 5 m, 0 m, 0 m));
    vc.addPoint(Point(j * 5 m, 3 m, 1.5 m));
    vc.addPoint(Point(j * 5 m, 6 m, 3 m));
    vc.addPoint(Point(j * 5 m, 9 m, 1.5 m));
    vc.name = "V_CURVE_" + j;
    vCurves.push(vc);
}

// Combine into a curve network
var curveNet = CurveNetwork();
curveNet.uCurves = uCurves;
curveNet.vCurves = vCurves;

var netSurface = CurveNetInterpolation(curveNet);
netSurface.thickness = 16 mm;
netSurface.material = S355;
netSurface.name = "NET_INTERP_SURFACE";

// PointNetFitting - fit surface to scattered point cloud
var pointCloud = [];
for (var row = 0; row < 8; row++) {
    for (var col = 0; col < 12; col++) {
        var x = col * 2 m;
        var y = row * 1.5 m;
        var z = 2 * Math.sin(col * 0.3) * Math.cos(row * 0.5);
        pointCloud.push(Point(x, y, z));
    }
}

var fittedSurface = PointNetFitting(pointCloud);
fittedSurface.tolerance = 0.01 m;    // 10mm fitting tolerance
fittedSurface.name = "POINT_FIT_SURFACE";

// ShapePreservingPointNetInterpolation - maintains local curvature
var hullPoints = [];
// Hull offset table points from lines plan
hullPoints.push([Point(0 m, 0 m, 1 m), Point(5 m, 2 m, 0.8 m), Point(10 m, 0 m, 1 m)]);
hullPoints.push([Point(0 m, 2 m, 2 m), Point(5 m, 4 m, 1.5 m), Point(10 m, 2 m, 2 m)]);
hullPoints.push([Point(0 m, 4 m, 3 m), Point(5 m, 6 m, 2.2 m), Point(10 m, 4 m, 3 m)]);

var hullForm = ShapePreservingPointNetInterpolation(hullPoints);
hullForm.degree = 3;
hullForm.name = "SHAPE_PRESERVING_HULL";
```

## ControlPointNet for NURBS Surfaces

```javascript
// ControlPointNet - define a NURBS surface via control points + weights
var cpn = ControlPointNet();

// 4x4 control point grid for a doubly-curved shell
// Each control point: position + weight
var degreeU = 3, degreeV = 3;
cpn.setDimensions(4, 4, degreeU, degreeV);

// Row 0 (Y=0)
cpn.setControlPoint(0, 0, Point(0 m, 0 m, 0 m), 1.0);
cpn.setControlPoint(0, 1, Point(5 m, 0 m, 1.5 m), 1.0);
cpn.setControlPoint(0, 2, Point(10 m, 0 m, 2 m), 0.8);
cpn.setControlPoint(0, 3, Point(15 m, 0 m, 1 m), 1.0);

// Row 1 (Y=5)
cpn.setControlPoint(1, 0, Point(0 m, 5 m, 1 m), 1.0);
cpn.setControlPoint(1, 1, Point(5 m, 5 m, 2.5 m), 1.0);
cpn.setControlPoint(1, 2, Point(10 m, 5 m, 3 m), 1.0);
cpn.setControlPoint(1, 3, Point(15 m, 5 m, 2 m), 1.0);

// Row 2 (Y=10)
cpn.setControlPoint(2, 0, Point(0 m, 10 m, 2 m), 1.0);
cpn.setControlPoint(2, 1, Point(5 m, 10 m, 3 m), 1.0);
cpn.setControlPoint(2, 2, Point(10 m, 10 m, 3.5 m), 0.7);
cpn.setControlPoint(2, 3, Point(15 m, 10 m, 2.5 m), 1.0);

// Row 3 (Y=15)
cpn.setControlPoint(3, 0, Point(0 m, 15 m, 0 m), 1.0);
cpn.setControlPoint(3, 1, Point(5 m, 15 m, 1 m), 1.0);
cpn.setControlPoint(3, 2, Point(10 m, 15 m, 1.5 m), 1.0);
cpn.setControlPoint(3, 3, Point(15 m, 15 m, 0.5 m), 1.0);

// Knot vectors
cpn.setUKnots([0, 0, 0, 0, 1, 1, 1, 1]);
cpn.setVKnots([0, 0, 0, 0, 1, 1, 1, 1]);

var nurbsShell = cpn.createSurface();
nurbsShell.thickness = 25 mm;
nurbsShell.material = S355;
nurbsShell.name = "NURBS_DOUBLY_CURVED";
```

## ThreeSidedHoleFilling & Circular Shells

```javascript
// ThreeSidedHoleFilling - repair gaps in shell surfaces
// Define the three boundary curves of the hole
var edgeA = ModelCurve();
edgeA.addPoint(Point(0 m, 0 m, 5 m));
edgeA.addPoint(Point(4 m, 0 m, 5 m));
edgeA.name = "HOLE_EDGE_A";

var edgeB = ModelCurve();
edgeB.addPoint(Point(4 m, 0 m, 5 m));
edgeB.addPoint(Point(2 m, 3 m, 5 m));
edgeB.name = "HOLE_EDGE_B";

var edgeC = ModelCurve();
edgeC.addPoint(Point(2 m, 3 m, 5 m));
edgeC.addPoint(Point(0 m, 0 m, 5 m));
edgeC.name = "HOLE_EDGE_C";

var holeFill = ThreeSidedHoleFilling(edgeA, edgeB, edgeC);
holeFill.thickness = 12 mm;
holeFill.material = S355;
holeFill.name = "HOLE_FILL_PATCH";

// CircularConeCylinder - axisymmetric cylindrical/tapered shell
var column = CircularConeCylinder(
    Point(0 m, 0 m, 0 m),      // Bottom center
    0.5 m,                        // Bottom radius
    Point(0 m, 0 m, 15 m),      // Top center
    0.3 m,                        // Top radius (different = tapered/cone)
    16                             // Number of circumferential facets
);
column.thickness = 15 mm;
column.material = S355;
column.name = "TAPERED_COLUMN_SHELL";

// EllipticConeCylinder - elliptical cross-section shell
var ellipticShell = EllipticConeCylinder(
    Point(5 m, 0 m, 0 m),       // Bottom center
    2 m, 1 m,                     // Semi-major X, semi-minor Y
    Point(5 m, 0 m, 10 m),       // Top center
    1.5 m, 0.8 m,                 // Smaller at top
    24                             // Circumferential facets
);
ellipticShell.thickness = 10 mm;
ellipticShell.material = S355;
ellipticShell.name = "ELLIPTIC_TOWER";

// Pipe shell (cylindrical, constant radius)
var pipeShell = Pipe(Point(0 m, 0 m, 0 m), Point(0 m, 0 m, 8 m), 0.35 m, 20);
pipeShell.thickness = 12 mm;
pipeShell.name = "PIPE_SHELL_STRUCTURAL";
```

## RevolveAProfileAroundAnAxis & Sphere

```javascript
// Define a profile curve for a pressure vessel head
var headProfile = ModelCurve();
headProfile.addPoint(Point(0 m, 0 m, 0 m));      // Center at axis
headProfile.addPoint(Point(1.5 m, 0 m, 0 m));    // 1.5m radius flange
headProfile.addPoint(Point(1.5 m, 0.5 m, 0 m));  // Flange flat
headProfile.addPoint(Point(0 m, 1.5 m, 0 m));    // Hemispherical dome
headProfile.name = "VESSEL_HEAD_PROFILE";

// RevolveAProfileAroundAnAxis - rotational surface
var rotationAxis = Axis(Point(0 m, 0 m, 0 m), Vector(0, 0, 1));  // Z-axis
var vesselHead = RevolveAProfileAroundAnAxis(headProfile, rotationAxis, 360 deg);
vesselHead.thickness = 20 mm;
vesselHead.material = S420;
vesselHead.name = "PRESSURE_VESSEL_HEAD";

// Semi-spherical dome (180 degree revolve)
var domeProfile = ModelCurve();
domeProfile.addPoint(Point(0 m, 0 m, 5 m));     // Apex
domeProfile.addPoint(Point(1 m, 0 m, 4.6 m));
domeProfile.addPoint(Point(2 m, 0 m, 3.5 m));
domeProfile.addPoint(Point(3 m, 0 m, 0 m));     // Base rim
domeProfile.name = "DOME_PROFILE";

var rotationAxisDome = Axis(Point(0 m, 0 m, 0 m), Vector(0, 0, 1));
var dome = RevolveAProfileAroundAnAxis(domeProfile, rotationAxisDome, 360 deg);
dome.thickness = 8 mm;
dome.name = "DOME_SHELL";

// Sphere - simple spherical shell
var sphere = Sphere(Point(10 m, 5 m, 3 m), 2.5 m);
sphere.thickness = 10 mm;
sphere.material = S355;
sphere.name = "SPHERICAL_TANK";
```

## CreateHullBySkinningApproximation - Ship Hull Design

```javascript
// Hull defined by transverse section polygons (frames/ stations)
var sectionPolygons = [];

// Station 0 (Aft Perpendicular)
var sta0 = [];
sta0.push(Point(0 m, 0 m, 0 m));         // Keel
sta0.push(Point(0 m, 3 m, 2 m));         // Bilge turn
sta0.push(Point(0 m, 5 m, 6 m));         // Mid side
sta0.push(Point(0 m, 6 m, 10 m));        // Deck edge
sta0.push(Point(0 m, 0 m, 10 m));        // Centerline deck
sectionPolygons.push(sta0);

// Station 5 (Midship)
var sta5 = [];
sta5.push(Point(25 m, 0 m, 0 m));        // Keel
sta5.push(Point(28 m, 6 m, 2 m));        // Bilge turn
sta5.push(Point(30 m, 10 m, 8 m));       // Mid side
sta5.push(Point(30 m, 12 m, 14 m));      // Deck edge
sta5.push(Point(25 m, 0 m, 14 m));       // Centerline deck
sectionPolygons.push(sta5);

// Station 10 (Forward Perpendicular)
var sta10 = [];
sta10.push(Point(50 m, 0 m, 0 m));       // Keel
sta10.push(Point(51 m, 2 m, 1.5 m));     // Bilge
sta10.push(Point(52 m, 4 m, 6 m));       // Mid side
sta10.push(Point(52 m, 4.5 m, 12 m));    // Deck edge
sta10.push(Point(50 m, 0 m, 12 m));      // Centerline deck
sectionPolygons.push(sta10);

// CreateHullBySkinningApproximation - skin through station polygons
var hullShell = CreateHullBySkinningApproximation(sectionPolygons);
hullShell.numberOfStations = 21;     // Interpolate to 21 stations
hullShell.thickness = 22 mm;
hullShell.bottomThickness = 25 mm;   // Thicker bottom shell
hullShell.sideThickness = 20 mm;     // Thinner side shell
hullShell.material = DH36;
hullShell.name = "VESSEL_HULL";
```

## CreateStiffeners on Hull & ProfilePunchOrCut

```javascript
// CreateStiffeners - automated stiffener generation on hull surface
var hullSurface = hullShell;  // From previous example

// Define stiffener parameters
var stiffenerConfig = CreateStiffeners();
stiffenerConfig.surface = hullSurface;

// Longitudinal stiffeners
stiffenerConfig.longitudinalCount = 12;    // 12 longitudinals each side
stiffenerConfig.longitudinalType = "BulbFlat";
stiffenerConfig.longitudinalSize = "200x10";  // 200mm web, 10mm thick
stiffenerConfig.longitudinalSpacing = 0.75 m;

// Transverse stiffeners (frames)
stiffenerConfig.transverseCount = 21;     // 21 frames
stiffenerConfig.transverseType = "T-Bar";
stiffenerConfig.transverseSize = "400x12+150x18";  // Web 400x12, Flange 150x18
stiffenerConfig.transverseSpacing = 2.5 m;

stiffenerConfig.material = S355;
stiffenerConfig.create();

// ProfilePunchOrCut - create openings/cutouts through plates
var openingProfile = ModelCurve();
openingProfile.addPoint(Point(-0.4 m, -0.4 m, 0 m));
openingProfile.addPoint(Point(0.4 m, -0.4 m, 0 m));
openingProfile.addPoint(Point(0.4 m, 0.4 m, 0 m));
openingProfile.addPoint(Point(-0.4 m, 0.4 m, 0 m));
openingProfile.name = "MANHOLE_600x600";

// Punch rectangular opening through hull plate
var opening1 = ProfilePunchOrCut(hullShell, openingProfile);
opening1.center = Point(20 m, 4 m, 5 m);
opening1.throughAll = true;      // Cut through entire thickness
opening1.name = "MANHOLE_OPENING_01";

// Circular opening for pipe penetration
var circularOpening = ModelCurve();
circularOpening.addPoint(Point(0 m, 0.3 m, 0 m));
circularOpening.addPoint(Point(0.3 m, 0 m, 0 m));
circularOpening.name = "PIPE_PENETRATION";

var penetration = ProfilePunchOrCut(hullShell, circularOpening);
penetration.center = Point(30 m, 8 m, 10 m);
penetration.name = "PIPE_PENETRATION_300";

// GuidingCurvePunchOrCut - opening defined along a guiding curve
var slotGuide = GuideLine(Point(5 m, 2 m, 8 m), Point(5 m, 8 m, 12 m));
var slotProfile = ModelCurve();
slotProfile.addPoint(Point(-0.1 m, -0.02 m, 0 m));
slotProfile.addPoint(Point(0.1 m, -0.02 m, 0 m));
slotProfile.name = "SLOT_PROFILE";

var slottedOpening = GuidingCurvePunchOrCut(hullShell, slotGuide, slotProfile);
slottedOpening.name = "VENTILATION_SLOT";
```

## InclinedStiffeners & ConvertCurvedShellToFlatPlate

```javascript
// InclinedStiffenersFromFourPoints - stiffeners angled relative to shell
var p1 = Point(0 m, 0 m, 5 m);     // Start of stiffener on hull
var p2 = Point(1 m, 0 m, 5.5 m);   // End of stiffener on hull
var p3 = Point(0 m, -0.5 m, 5 m);   // Offset in plate plane at start
var p4 = Point(1 m, -0.5 m, 5.5 m); // Offset in plate plane at end

var inclinedStf = InclinedStiffenersFromFourPoints(p1, p2, p3, p4);
inclinedStf.angle = 45 deg;              // 45 degree inclination from shell normal
inclinedStf.height = 0.25 m;             // Web height
inclinedStf.flangeWidth = 0.12 m;        // Flange width
inclinedStf.thickness = 12 mm;
inclinedStf.material = S355;
inclinedStf.name = "INCLINED_STIFFENER_45DEG";

// Batch of inclined stiffeners along hull
var pBase1 = Point(2 m, 1 m, 5 m);
var pBase2 = Point(2 m, 3 m, 5 m);
var pBase3 = Point(8 m, 1 m, 6 m);
var pBase4 = Point(8 m, 3 m, 6 m);
var down1 = Point(2 m, 0.5 m, 5 m);
var down2 = Point(2 m, 2.5 m, 5 m);
var down3 = Point(8 m, 0.5 m, 6 m);
var down4 = Point(8 m, 2.5 m, 6 m);

var inclinedStf2 = InclinedStiffenersFromFourPoints(pBase1, pBase2, down1, down2);
inclinedStf2.angle = 60 deg;
inclinedStf2.height = 0.3 m;
inclinedStf2.name = "INCLINED_STF_60DEG";

// ConvertCurvedShellToFlatPlate - simplify curved shell to flat approximation
// Useful for tank tops or decks with slight camber
var curvedDeck = SkinCurves([curveBottom, curveTop]);  // From earlier example
curvedDeck.name = "CURVED_DECK_ORIGINAL";

var flatDeck = ConvertCurvedShellToFlatPlate(curvedDeck);
flatDeck.maxDeviation = 5 mm;        // Max 5mm deviation from original
flatDeck.preserveBoundary = true;    // Keep original boundary shape
flatDeck.name = "FLATTENED_DECK";

// Compare surface areas
print("Original curved deck area: " + curvedDeck.area() + " m^2");
print("Flattened deck area: " + flatDeck.area() + " m^2");
print("Deviation tolerance: " + flatDeck.maxDeviation);

// Multi-panel hull conversion
var allCurvedShells = [hullPanel, bowPanel, loftedHull, hullShell];
var flatApproximations = [];
for (var i = 0; i < allCurvedShells.length; i++) {
    var flat = ConvertCurvedShellToFlatPlate(allCurvedShells[i]);
    flat.maxDeviation = 8 mm;
    flat.name = allCurvedShells[i].name + "_FLAT_APPROX";
    flatApproximations.push(flat);
    print("Converted: " + allCurvedShells[i].name +
          " -> " + flat.name + " (max dev: " + flat.maxDeviation + ")");
}
```
