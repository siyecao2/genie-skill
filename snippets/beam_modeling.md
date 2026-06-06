# Beam Modeling - GeniE SESAM Snippets

## StraightBeam Creation

```javascript
// Two-point method - simplest beam creation
var b1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
b1.name = "BEAM_01";
b1.section = pipeMain;
b1.material = S355;

// Multi-point beam (piecewise straight)
var b2 = StraightBeam();
b2.addPoint(Point(0 m, 0 m, 0 m));
b2.addPoint(Point(5 m, 2 m, 0 m));
b2.addPoint(Point(10 m, 0 m, 0 m));
b2.name = "BEAM_02";

// Beam array via loop
for (var i = 0; i < 5; i++) {
    var b = StraightBeam(Point(i * 5 m, 0 m, 0 m), Point(i * 5 m, 0 m, 10 m));
    b.name = "COLUMN_" + (i + 1);
    b.section = pipeMain;
    b.material = S355;
}
```

## Beam from Guide Curve

```javascript
// Create a guide curve first
var gc = GuideCurve(gpMain);
gc.curveType = "Spline";
gc.addPoint(Point(0 m, 0 m, 0 m));
gc.addPoint(Point(4 m, 1 m, 0 m));
gc.addPoint(Point(8 m, 0.5 m, 0 m));
gc.addPoint(Point(12 m, 0 m, 0 m));

// Beam from guide curve
var bSpline = StraightBeam(gc);
bSpline.name = "SPLINE_BEAM";
bSpline.section = pipeBrace;
bSpline.material = S355;
```

## Section & Material Assignment

```javascript
// Default behavior - inherits from setDefault()
setDefault("Material", S355);
setDefault("PipeSection", pipeMain);

var bDefault = StraightBeam(Point(0 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
bDefault.name = "USES_DEFAULTS";

// Explicit assignment overrides defaults
var bExplicit = StraightBeam(Point(0 m, 5 m, 0 m), Point(10 m, 5 m, 0 m));
bExplicit.section = pipeBrace;
bExplicit.material = S420;
bExplicit.name = "EXPLICIT_MAT_SECT";

// Change section after creation
bExplicit.section = PipeSection(0.406 m, 0.016 m);
```

## CurveOffset (AlignedCurveOffset)

```javascript
// AlignedCurveOffset for eccentric beam placement
// Options: frFlushTop, frFlushBottom, frCenter

// GuidePlane and guide curve for reference
var gpDeck = GuidePlane();
gpDeck.addPoint(Point(0 m, 0 m, 10 m));
gpDeck.addPoint(Point(20 m, 0 m, 10 m));
gpDeck.addPoint(Point(20 m, 8 m, 10 m));
gpDeck.addPoint(Point(0 m, 8 m, 10 m));

var gcRef = GuideLine(Point(0 m, 4 m, 10 m), Point(20 m, 4 m, 10 m));

// Create beam flush with top of reference plane
var bTop = StraightBeam(AlignedCurveOffset(gcRef, frFlushTop));
bTop.name = "BEAM_FLUSH_TOP";
bTop.section = ISection(0.5 m, 0.3 m, 0.3 m, 0.01 m, 0.016 m, 0.016 m);

// Create beam at center
var bCenter = StraightBeam(AlignedCurveOffset(gcRef, frCenter));
bCenter.name = "BEAM_CENTER";

// Create beam flush with bottom
var bBottom = StraightBeam(AlignedCurveOffset(gcRef, frFlushBottom));
bBottom.name = "BEAM_FLUSH_BOTTOM";
```

## Beam Orientation

```javascript
// rotateLocalX - rotate the local X-axis of the beam section
var b1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
b1.rotateLocalX(90 deg);  // Rotate strong axis orientation

// localSystemRule with GuideLocalSystem
var gls = GuideLocalSystem(Point(5 m, 0 m, 0 m), Vector(0, 1, 0), Vector(0, 0, 1));
b1.localSystemRule = gls;

// Manual orientation via local Z-axis vector
var b2 = StraightBeam(Point(0 m, 5 m, 0 m), Point(0 m, 5 m, 10 m));
b2.orientation = Vector(1, 0, 0);  // Flange oriented in global X direction
```

## Beam Modification Operations

```javascript
// extendEnd - extend a beam from its end point
var b = StraightBeam(Point(0 m, 0 m, 0 m), Point(8 m, 0 m, 0 m));
b.extendEnd(2 m);  // Now 10m long (0 to 10m)
// b.extendEnd(-2 m);  // Shorten by 2m

// divideAtEccentric - split beam at eccentric connection
var bLong = StraightBeam(Point(0 m, 0 m, 0 m), Point(15 m, 0 m, 0 m));
var splitBeams = bLong.divideAtEccentric(Point(6 m, 0 m, 0 m));
var leftBeam = splitBeams[0];   // 0 to 6m
var rightBeam = splitBeams[1];  // 6m to 15m

// moveEnd - relocate one end of a beam
var b3 = StraightBeam(Point(0 m, 0 m, 5 m), Point(10 m, 0 m, 5 m));
b3.moveEnd(Point(12 m, 0 m, 5 m));  // Move end2 to new position

// divideSegmentAtEccentric and SetSegmentSection
var bVar = StraightBeam(Point(0 m, 0 m, 0 m), Point(20 m, 0 m, 0 m));
bVar.divideSegmentAtEccentric(Point(8 m, 0 m, 0 m));

// Set different section for each segment
bVar.setSegmentSection(0, PipeSection(0.610 m, 0.022 m));  // First segment
bVar.setSegmentSection(1, PipeSection(0.508 m, 0.016 m));  // Second segment
bVar.name = "VARIABLE_SECTION_BEAM";
```

## BeamCreationRule

```javascript
// Automated beam creation between existing members
var beamRule = BeamCreationRule();
beamRule.name = "BRACE_RULE";
beamRule.createBetween = cbAll;        // Create brace between all members
beamRule.beamType = InnerBeam;
beamRule.section = pipeBrace;
beamRule.material = S355;

// Apply rule
beamRule.create();
```

## BeamType & Specialty Beams

```javascript
// InnerBeam - standard structural beam
var bStruct = StraightBeam(Point(0 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
bStruct.beamType = InnerBeam;
bStruct.section = pipeMain;

// Nonstruct - non-structural (visual/reference only)
var bNonstruct = StraightBeam(Point(0 m, 2 m, 0 m), Point(10 m, 2 m, 0 m));
bNonstruct.beamType = Nonstruct;

// Shim - thin plate-like beam for alignment
var bShim = StraightBeam(Point(0 m, 4 m, 0 m), Point(10 m, 4 m, 0 m));
bShim.beamType = Shim;

// Truss - pinned end beam (axial only)
var bTruss = StraightBeam(Point(0 m, 6 m, 0 m), Point(10 m, 6 m, 0 m));
bTruss.beamType = Truss;
bTruss.section = BarSection(0.003 m^2);
```

## BeamAsBraceAdapter for Jacket Braces

```javascript
// Create jacket leg
var leg1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(0 m, 0 m, 60 m));
leg1.name = "LEG_A1";
leg1.section = PipeSection(1.0 m, 0.05 m);

var leg2 = StraightBeam(Point(20 m, 0 m, 0 m), Point(18 m, 0 m, 60 m));
leg2.name = "LEG_B1";
leg2.section = PipeSection(1.0 m, 0.05 m);

// Create brace adapter - automatically connects brace to legs
var braceAdapter = BeamAsBraceAdapter();
braceAdapter.section = pipeBrace;
braceAdapter.material = S355;
braceAdapter.beamType = InnerBeam;

// Define brace geometry (K-brace example)
braceAdapter.addPoint(Point(0 m, 0 m, 30 m));     // left leg connect
braceAdapter.addPoint(Point(10 m, 0 m, 15 m));    // mid intersection
braceAdapter.addPoint(Point(19 m, 0 m, 30 m));    // right leg connect
braceAdapter.name = "K_BRACE_01";
```

## CurvedBeam

```javascript
// Curved beam for arched or curved structural members
var cbArch = CurvedBeam();
cbArch.curveType = "Arc";
cbArch.center = Point(10 m, -5 m, 0 m);
cbArch.radius = 5 m;
cbArch.startAngle = 0 deg;
cbArch.endAngle = 180 deg;
cbArch.section = pipeMain;
cbArch.material = S355;
cbArch.name = "ARCH_BEAM";

// CurvedBeam from spline
var cbSpline = CurvedBeam();
cbSpline.curveType = "Spline";
cbSpline.addPoint(Point(0 m, 0 m, 0 m));
cbSpline.addPoint(Point(3 m, 1.5 m, 0 m));
cbSpline.addPoint(Point(7 m, 1.5 m, 0 m));
cbSpline.addPoint(Point(10 m, 0 m, 0 m));
cbSpline.section = PipeSection(0.508 m, 0.019 m);
cbSpline.name = "CURVED_BRACE";
```

## BeamJoiner

```javascript
// Create beams that need to be connected
var b1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(5 m, 0 m, 0 m));
var b2 = StraightBeam(Point(5 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
var b3 = StraightBeam(Point(5 m, 0 m, 0 m), Point(5 m, 5 m, 0 m));

// BeamJoiner connects multiple beams at a common point
var joiner = BeamJoiner();
joiner.add(b1);
joiner.add(b2);
joiner.add(b3);
joiner.connectAt(Point(5 m, 0 m, 0 m));  // Ensure all meet at this point
joiner.name = "JOINT_K1";
```