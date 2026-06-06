# Guiding Geometry - GeniE SESAM Snippets

## GuidePlane Creation

```javascript
// 4-point rectangular GuidePlane
var gpMain = GuidePlane();
gpMain.addPoint(Point(0 m, 0 m, 0 m));
gpMain.addPoint(Point(20 m, 0 m, 0 m));
gpMain.addPoint(Point(20 m, 10 m, 0 m));
gpMain.addPoint(Point(0 m, 10 m, 0 m));
gpMain.name = "MAIN_GUIDEPLANE";

// Triangular GuidePlane (3 points)
var gpTri = GuidePlane();
gpTri.addPoint(Point(0 m, 0 m, 5 m));
gpTri.addPoint(Point(8 m, 0 m, 5 m));
gpTri.addPoint(Point(4 m, 6 m, 5 m));
gpTri.name = "TRI_GUIDEPLANE";

// Trapezoidal GuidePlane
var gpTrap = GuidePlane();
gpTrap.addPoint(Point(0 m, 0 m, 0 m));
gpTrap.addPoint(Point(15 m, 0 m, 0 m));
gpTrap.addPoint(Point(12 m, 8 m, 0 m));
gpTrap.addPoint(Point(3 m, 8 m, 0 m));
gpTrap.name = "TRAP_GUIDEPLANE";
```

## GuidePlane SnapMode & Grid

```javascript
// Configure snap mode for precise modeling
var gpDeck = GuidePlane();
gpDeck.addPoint(Point(0 m, 0 m, 30 m));
gpDeck.addPoint(Point(60 m, 0 m, 30 m));
gpDeck.addPoint(Point(60 m, 40 m, 30 m));
gpDeck.addPoint(Point(0 m, 40 m, 30 m));

gpDeck.snapMode = smPoint;
gpDeck.snapX = 2 m;    // Snap interval in X direction
gpDeck.snapY = 2 m;    // Snap interval in Y direction
gpDeck.name = "DECK_SNAPPED";

// Grid-based modeling on GuidePlane
gpDeck.setGrid(2 m, 2 m);  // 2m x 2m grid spacing
```

## GuideLine / CreateLineTwoPoints

```javascript
// GuideLine from two points
var gl1 = GuideLine(Point(0 m, 0 m, 0 m), Point(50 m, 0 m, 0 m));
gl1.name = "BASE_LINE";

// CreateLineTwoPoints - alternative syntax
var gl2 = CreateLineTwoPoints(Point(0 m, 5 m, 0 m), Point(50 m, 5 m, 0 m));
gl2.name = "LINE2";

// GuideLine with intermediate points for segmented modeling
var glSeg = GuideLine();
glSeg.addPoint(Point(0 m, 0 m, 0 m));
glSeg.addPoint(Point(15 m, 3 m, 0 m));
glSeg.addPoint(Point(30 m, 0 m, 0 m));
glSeg.name = "SEGMENTED_LINE";
```

## GuideCurve Types

```javascript
// Arc curve - center-based definition
var gpCirc = GuidePlane();
// ... add points as needed
var gcArc = GuideCurve(gpCirc);
gcArc.curveType = "Arc";
gcArc.setArcByCenter(Point(10 m, 5 m, 0 m), 5 m, 0 deg, 180 deg);

// Ellipse curve
var gcEllipse = GuideCurve(gpCirc);
gcEllipse.curveType = "Ellipse";
gcEllipse.setEllipse(Point(10 m, 5 m, 0 m), 8 m, 4 m, 0 deg, 180 deg);

// Spline curve (cubic)
var gcSpline = GuideCurve(gpCirc);
gcSpline.curveType = "Spline";
gcSpline.addPoint(Point(0 m, 0 m, 0 m));
gcSpline.addPoint(Point(5 m, 2 m, 0 m));
gcSpline.addPoint(Point(10 m, 1 m, 0 m));
gcSpline.addPoint(Point(15 m, 3 m, 0 m));
gcSpline.addPoint(Point(20 m, 0 m, 0 m));

// Bezier curve
var gcBezier = GuideCurve(gpCirc);
gcBezier.curveType = "Bezier";
gcBezier.addControlPoint(Point(0 m, 0 m, 0 m));     // P0
gcBezier.addControlPoint(Point(3 m, 4 m, 0 m));     // P1
gcBezier.addControlPoint(Point(7 m, 4 m, 0 m));     // P2
gcBezier.addControlPoint(Point(10 m, 0 m, 0 m));    // P3

// NURBS curve with control points and weights
var gcNURBS = GuideCurve(gpCirc);
gcNURBS.curveType = "NURBS";
gcNURBS.setDegree(3);
gcNURBS.addControlPoint(Point(0 m, 0 m, 0 m), 1.0);
gcNURBS.addControlPoint(Point(3 m, 2 m, 0 m), 1.0);
gcNURBS.addControlPoint(Point(7 m, 2 m, 0 m), 0.8);
gcNURBS.addControlPoint(Point(10 m, 0 m, 0 m), 1.0);
gcNURBS.setKnotVector([0, 0, 0, 0, 1, 1, 1, 1]);
```

## Curve Spacings (Mesh Seed Control)

```javascript
// Set mesh seed along a curve for element density control
var gc = GuideCurve(gpMain);
gc.curveType = "Spline";
gc.addPoint(Point(0 m, 0 m, 0 m));
gc.addPoint(Point(20 m, 0 m, 0 m));

// Spacing options
gc.spacing = 0.5 m;       // Uniform 0.5m spacing
// gc.spacing = "variable"; // Variable spacing (use spacingList)
gc.spacingList = [0.3 m, 0.5 m, 0.3 m, 0.5 m, 0.3 m];  // Variable spacing array
```

## CopyTranslate of GuidePlane

```javascript
// Translate a GuidePlane to create parallel framing planes
var gpBase = GuidePlane();
gpBase.addPoint(Point(0 m, 0 m, 0 m));
gpBase.addPoint(Point(30 m, 0 m, 0 m));
gpBase.addPoint(Point(30 m, 5 m, 0 m));
gpBase.addPoint(Point(0 m, 5 m, 0 m));

// Copy translate to create frames at 5m intervals
var gpFrame1 = gpBase.copyTranslate(Vector(0 m, 0 m, 5 m));
gpFrame1.name = "FRAME_5";

var gpFrame2 = gpBase.copyTranslate(Vector(0 m, 0 m, 10 m));
gpFrame2.name = "FRAME_10";

var gpFrame3 = gpBase.copyTranslate(Vector(0 m, 0 m, 15 m));
gpFrame3.name = "FRAME_15";
```

## GuideLocalSystem & GuidePoint

```javascript
// GuideLocalSystem for defining local coordinate orientation
var gls1 = GuideLocalSystem(Point(0 m, 0 m, 0 m), Vector(1, 0, 0), Vector(0, 1, 0));
gls1.name = "GLOBAL_ALIGNED";

var glsLocal = GuideLocalSystem(Point(10 m, 5 m, 3 m), Vector(0.866, 0.5, 0), Vector(-0.5, 0.866, 0));
glsLocal.name = "ROTATED_30DEG";

// GuidePoint for reference markers
var gpTop = GuidePoint(Point(10 m, 0 m, 30 m));
gpTop.name = "TOP_CENTER";

var gpBottom = GuidePoint(Point(10 m, 0 m, -10 m));
gpBottom.name = "BOTTOM_CENTER";
```

## ModelCurve for Precise Trajectories

```javascript
// ModelCurve - creates a 3D curve in model space for scanning/skinning
var mc1 = ModelCurve();
mc1.addPoint(Point(0 m, 0 m, 0 m));
mc1.addPoint(Point(5 m, 2 m, 1 m));
mc1.addPoint(Point(10 m, 0 m, 3 m));
mc1.addPoint(Point(15 m, -2 m, 5 m));
mc1.name = "DECK_EDGE_CURVE";

// ModelCurve from existing GuideCurve
var mcFromGC = ModelCurve(gcSpline);
mcFromGC.name = "IMPORTED_CURVE";
```

## Curve Split/Join Operations

```javascript
// Split a curve at a point
var glong = GuideLine(Point(0 m, 0 m, 0 m), Point(20 m, 0 m, 0 m));
var splitPoint = Point(8 m, 0 m, 0 m);

var curves = glong.split(splitPoint);  // Returns array of two curves
var c1 = curves[0];  // 0 to 8m
var c2 = curves[1];  // 8m to 20m

// Join curves
var cJoined = c1.join(c2);
cJoined.name = "REJOINED";
```

## Point, PointSet, PointGrid Creation

```javascript
// Single points
var p1 = Point(0 m, 0 m, 0 m);
var p2 = Point(10 m, 5 m, 0 m);
var p3 = Point(20 m, 0 m, 0 m);

// PointSet - collection of discrete points
var psDeck = PointSet();
psDeck.addPoint(Point(0 m, 0 m, 5 m));
psDeck.addPoint(Point(5 m, 0 m, 5 m));
psDeck.addPoint(Point(10 m, 0 m, 5 m));
psDeck.addPoint(Point(15 m, 0 m, 5 m));
psDeck.addPoint(Point(20 m, 0 m, 5 m));
psDeck.name = "DECK_BEAM_POINTS";

// PointGrid - regular grid of points
var pg = PointGrid(Point(0 m, 0 m, 0 m),  // origin
                   Vector(1 m, 0 m, 0 m),   // X direction, 1m spacing
                   Vector(0 m, 1 m, 0 m),   // Y direction, 1m spacing
                   20, 10);                  // 20 points in X, 10 in Y
pg.name = "ANALYSIS_GRID";
```