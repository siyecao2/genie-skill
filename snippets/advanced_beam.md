# Advanced Beam Modeling - GeniE SESAM Snippets

## Segmented Beam (Multi-Section)

```javascript
// Define pipe sections for different segments
var Pipe800 = PipeSection(0.812 m, 0.038 m);  // Main leg section
var Pipe600 = PipeSection(0.610 m, 0.025 m);  // Reduced section
var Pipe400 = PipeSection(0.406 m, 0.016 m);  // Brace connector

// Create a beam and divide at eccentricity points
var bLeg = StraightBeam(Point(0 m, 0 m, 0 m), Point(0 m, 0 m, 40 m));
bLeg.name = "JACKET_LEG_A1";
bLeg.material = S355;

// Divide the beam at specific Z elevations - returns array [seg0, seg1, seg2]
var segs = bLeg.divideAtEccentric(Point(0 m, 0 m, 10 m));
// segs[0]: 0~10m, keep as the working beam

var segs2 = segs[1].divideAtEccentric(Point(0 m, 0 m, 25 m));
// segs2[0]: 10~25m, segs2[1]: 25~40m

// Set different sections for each segment
bLeg.setSegmentSection(0, Pipe800);  // Bottom segment 0~10m
bLeg.setSegmentSection(1, Pipe600);  // Middle segment 10~25m
bLeg.setSegmentSection(2, Pipe400);  // Top segment 25~40m

// Alternative: divide using relative position (0.0 to 1.0)
var bLeg2 = StraightBeam(Point(20 m, 0 m, 0 m), Point(20 m, 0 m, 40 m));
bLeg2.divideAtEccentric(0.5);          // Split at midpoint
bLeg2.setSegmentSection(0, Pipe800);   // 0~50%
bLeg2.setSegmentSection(1, Pipe600);   // 50%~100%
bLeg2.name = "JACKET_LEG_B1";
```

## Tubular Joint Detailing (Cans & Stubs)

```javascript
// Define jacket legs and braces
var legA1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(0 m, 0 m, 60 m));
legA1.section = PipeSection(1.016 m, 0.045 m);
legA1.name = "LEG_A1";

var legB1 = StraightBeam(Point(25 m, 0 m, 0 m), Point(23 m, 0 m, 60 m));
legB1.section = PipeSection(1.016 m, 0.045 m);
legB1.name = "LEG_B1";

// Create K-brace using BeamAsBraceAdapter
var kbrace = BeamAsBraceAdapter();
kbrace.section = PipeSection(0.508 m, 0.019 m);
kbrace.material = S355;
kbrace.addPoint(Point(0 m, 0 m, 15 m));      // Brace end 1 at leg A1
kbrace.addPoint(Point(13 m, 0 m, 30 m));     // Mid gap
kbrace.addPoint(Point(0 m, 0 m, 45 m));      // Brace end 2 at leg A1
kbrace.name = "K_BRACE_A1";

var kbraceB = BeamAsBraceAdapter();
kbraceB.section = PipeSection(0.508 m, 0.019 m);
kbraceB.material = S355;
kbraceB.addPoint(Point(25 m, 0 m, 15 m));
kbraceB.addPoint(Point(12 m, 0 m, 30 m));
kbraceB.addPoint(Point(25 m, 0 m, 45 m));
kbraceB.name = "K_BRACE_B1";

// Assign joint cans - thickened chord sections at joint locations
var canSpec = JointCan();
canSpec.thickness = 0.060 m;     // 60mm thickened can
canSpec.length = 1.5 m;          // 1.5m can length (each side of brace centerline)
canSpec.chord = legA1;

// Assign a joint can at a specific joint location
AssignJointCan(Point(0 m, 0 m, 15 m), canSpec);
AssignJointCan(Point(0 m, 0 m, 45 m), canSpec);

// Assign cones (tapered transition between can and base pipe)
var coneSpec = Cone();
coneSpec.thickness = 0.038 m;
coneSpec.length = 0.8 m;
coneSpec.startDiameter = 1.016 m + 2 * 0.060 m;  // Can outer diameter
coneSpec.endDiameter = 1.016 m;                    // Base pipe outer diameter

AssignCones(Point(0 m, 0 m, 15 m), coneSpec);

// Batch assign cans and stubs to all joints
var canSpec2 = JointCan();
canSpec2.thickness = 0.055 m;
canSpec2.length = 1.2 m;

var stubSpec = Stub();
stubSpec.thickness = 0.025 m;
stubSpec.length = 0.6 m;

// Apply to all brace-to-chord connections at once
AssignCansAndStubs(KJointType, canSpec2, stubSpec);

// Assign all joints with detailed specification
AssignJoints(kbrace);
```

## Joint Creation & Design Rules

```javascript
// RulesForJointCreation - control how GeniE automatically detects joints
RulesForJointCreation({
    minBraceAngle: 20 deg,           // Minimum brace-to-chord angle
    maxEccentricity: 0.15 m,         // Max gap/overlap eccentricity
    jointTypeDetection: "automatic", // Auto-detect K, T, Y, X joints
    mergeTolerance: 0.05 m           // Snapping tolerance
});

// RulesForJointDesign - classification for joint strength checks
RulesForJointDesign({
    chordClassification: "CHS",      // Circular Hollow Section chord
    braceClassification: "CHS",
    gapJoints: true,                 // Allow gap-type joints
    overlapJoints: true,             // Allow overlap-type joints
    minGap: 0.05 m,
    maxOverlap: 0.8,                 // 80% max overlap ratio
    chordEndFixity: "fixed-fixed"    // Chord end condition
});

// Manual chord/brace designation for a specific joint
var jointK1 = Joint(Point(0 m, 0 m, 15 m));
jointK1.chordMember = legA1;
jointK1.addBraceMember(kbrace);     // Add brace to joint
jointK1.type = "K";                  // Explicit joint type
jointK1.name = "K_JOINT_EL_15M";
```

## Local Joint Flexibility

```javascript
// LocalJointFlexibility - model realistic joint stiffness
// Jacket joints are not perfectly rigid; LJF accounts for local deformations

var ljf = LocalJointFlexibility();
ljf.chordThickness = 0.045 m;
ljf.chordDiameter = 1.016 m;
ljf.braceThickness = 0.019 m;
ljf.braceDiameter = 0.508 m;
ljf.jointType = "K";
ljf.gapBetweenBraces = 0.08 m;
ljf.angleBetweenBraceAndChord = 45 deg;

// Apply LJF to all tubular joints in structure
ljf.applyToAllJoints();

// Apply to specific joint
var ljfJoint = LocalJointFlexibility();
ljfJoint.chordThickness = 0.060 m;
ljfJoint.chordDiameter = 1.016 m;
ljfJoint.braceThickness = 0.025 m;
ljfJoint.braceDiameter = 0.610 m;
ljfJoint.applyToJoint(jointK1);

print("Local joint flexibility applied to: " + jointK1.name);
```

## Pile Inside Leg (Grouted Case)

```javascript
// Outer leg (jacket leg)
var legOuter = StraightBeam(Point(0 m, 0 m, -30 m), Point(0 m, 0 m, 10 m));
legOuter.section = PipeSection(1.200 m, 0.050 m);
legOuter.name = "LEG_OUTER";
legOuter.material = S420;

// Inner pile - placed concentrically inside leg
var pileInner = StraightBeam(Point(0 m, 0 m, -50 m), Point(0 m, 0 m, 5 m));
pileInner.section = PipeSection(1.066 m, 0.040 m);
pileInner.name = "PILE_INNER";
pileInner.material = S355;

// Grout layer: define as material with grout properties
var Grout = Material();
Grout.E = 25e9 Pa;           // Grout elastic modulus
Grout.poisson = 0.2;
Grout.density = 2400 kg/m^3;

// Grouted pile-in-leg connection using SharedNodeConnector
var groutedConnection = SharedNodeConnector();
groutedConnection.master = legOuter;
groutedConnection.slave = pileInner;
groutedConnection.type = "Grouted";
groutedConnection.groutThickness = 0.017 m;    // Annular gap = (1200-1066-2*50-2*40)/2 ≈ 17mm
groutedConnection.groutLength = 8 m;            // Grout length from mudline
groutedConnection.groutMaterial = Grout;
groutedConnection.shearKeySpacing = 0.3 m;      // Shear key weld bead spacing
groutedConnection.shearKeyHeight = 0.006 m;     // Weld bead height
groutedConnection.name = "GROUTED_CONN_A1";
```

## Beam Eccentricities

```javascript
// Reference guide curve for deck beams
var gcDeck = GuideLine(Point(0 m, 0 m, 25 m), Point(30 m, 0 m, 25 m));

// AlignedCurveOffset - offset beam relative to a reference plane
// frFlushTop: beam top flange aligns with reference plane
var bFlushTop = StraightBeam(AlignedCurveOffset(gcDeck, frFlushTop, 0 m));
bFlushTop.section = ISection(0.6 m, 0.3 m, 0.3 m, 0.02 m, 0.025 m, 0.025 m);
bFlushTop.name = "BEAM_FLUSH_TOP";

// AlignedCurveOffset with lateral offset
var bOffset1 = StraightBeam(AlignedCurveOffset(gcDeck, frCenter, 0.5 m));
bOffset1.name = "BEAM_LATERAL_OFFSET";

// ConstantCurveOffset - fixed offset vector from reference curve
var bConstOff = StraightBeam(ConstantCurveOffset(gcDeck, Vector(0, 0, -0.5 m)));
bConstOff.name = "BEAM_CONST_OFFSET";

// EndOffset - different offsets at each end of beam
var bEndOff = StraightBeam(
    EndOffset(gcDeck, Point(0 m, 0 m, -0.3 m), Point(0 m, 0 m, -0.8 m))
);
bEndOff.name = "BEAM_END_OFFSET";

// ConstantCurveOffsetAtPoint - offset defined by point on curve
var bAtPoint = StraightBeam(
    ConstantCurveOffsetAtPoint(gcDeck, Point(10 m, 0 m, 25 m), Vector(0, -0.3 m, -0.5 m))
);
bAtPoint.name = "BEAM_AT_POINT";

// ReparameterizedBeamCurveOffset - default, maintains parametric mapping
var bReparam = StraightBeam(ReparameterizedBeamCurveOffset(gcDeck));
bReparam.name = "BEAM_REPARAM_DEFAULT";
```

## Beam End Sniping (Intersection Trimming)

```javascript
// Sniping - automatic trimming of intersecting beams
var chord1 = StraightBeam(Point(0 m, 0 m, 10 m), Point(20 m, 0 m, 10 m));
chord1.section = PipeSection(0.610 m, 0.025 m);
chord1.name = "CHORD_1";

var chord2 = StraightBeam(Point(0 m, 20 m, 10 m), Point(20 m, 20 m, 10 m));
chord2.section = PipeSection(0.610 m, 0.025 m);
chord2.name = "CHORD_2";

var diagonal = StraightBeam(Point(5 m, 0 m, 10 m), Point(15 m, 20 m, 10 m));
diagonal.section = PipeSection(0.406 m, 0.016 m);
diagonal.name = "DIAGONAL_BRACE";

// Enable snipe for automatic intersection trimming
diagonal.snipeEnds = true;           // Trim both ends at chord intersections
diagonal.snipeGap = 0.003 m;         // 3mm gap for welding access

// Selective sniping - snip only one end
var diag2 = StraightBeam(Point(0 m, 0 m, 10 m), Point(12 m, 18 m, 10 m));
diag2.section = PipeSection(0.356 m, 0.012 m);
diag2.snipeAtStart = true;           // Snip only at start end (at chord1)
diag2.snipeAtEnd = false;            // No sniping at end
diag2.name = "DIAGONAL_SNIPE_START";
```

## Beam Hinges

```javascript
// Create a beam with a hinge at a specified location
var bLong = StraightBeam(Point(0 m, 0 m, 0 m), Point(15 m, 0 m, 0 m));
bLong.section = ISection(0.5 m, 0.25 m, 0.25 m, 0.012 m, 0.018 m, 0.018 m);
bLong.material = S355;
bLong.name = "BEAM_WITH_HINGE";

// Hinge at 6m from start - moment release for My and Mz
bLong.Hinge(6 m, {My: true, Mz: true, Mx: false});

// Hinge at beam end (simple pin connection)
var bHingeEnd = StraightBeam(Point(0 m, 3 m, 0 m), Point(10 m, 3 m, 0 m));
bHingeEnd.section = PipeSection(0.406 m, 0.016 m);
bHingeEnd.Hinge(10 m, {My: true, Mz: true, Mx: true});  // Full moment release at end
bHingeEnd.name = "PIN_ENDED_BEAM";

// Multiple hinges in one beam
var bMulti = StraightBeam(Point(0 m, 6 m, 0 m), Point(18 m, 6 m, 0 m));
bMulti.section = ISection(0.4 m, 0.2 m, 0.2 m, 0.01 m, 0.015 m, 0.015 m);
bMulti.Hinge(4 m, {My: true, Mz: true});   // Hinge at 4m
bMulti.Hinge(12 m, {My: true, Mz: true});  // Hinge at 12m -> two-span Gerber beam
bMulti.name = "GERBER_BEAM";
```

## Beam Length Adjustment

```javascript
// extendEnd - extend or shorten a beam from specified end
var b1 = StraightBeam(Point(0 m, 0 m, 5 m), Point(8 m, 0 m, 5 m));
b1.section = PipeSection(0.508 m, 0.019 m);
b1.name = "ORIGINAL_BEAM";

// Extend end 1 by +2m (0m → -2m for end1)
b1.extendEnd(1, 2 m);
// Now: Point(-2, 0, 5) to Point(8, 0, 5), total 10m

// Shorten end 2 by -3m
b1.extendEnd(2, -3 m);
// Now: Point(-2, 0, 5) to Point(5, 0, 5), total 7m

// Practical use: extend brace to penetrate chord wall
var brace = StraightBeam(Point(0 m, 0 m, 10 m), Point(8 m, 8 m, 18 m));
var chord = StraightBeam(Point(0 m, 0 m, 10 m), Point(0 m, 0 m, 30 m));
chord.section = PipeSection(0.762 m, 0.032 m);

// Extend brace so it fully intersects the chord
brace.extendEnd(1, 0.4 m);  // Penetrate 0.4m into chord centerline

// insertSplitPoint - add a node at a specific location along beam
var bLongSpan = StraightBeam(Point(0 m, 0 m, 0 m), Point(20 m, 0 m, 0 m));
bLongSpan.insertSplitPoint(Point(7 m, 0 m, 0 m));   // Add node at 7m
bLongSpan.insertSplitPoint(Point(14 m, 0 m, 0 m));  // Add node at 14m
// Beam now has 3 segments with intermediate nodes for load application
bLongSpan.name = "BEAM_WITH_SPLIT_POINTS";
```

## Overlapping Beams & BeamJoiner

```javascript
// Two beams that nominally meet but may not share exact topology
var bLeft = StraightBeam(Point(0 m, 0 m, 0 m), Point(8.01 m, 0.005 m, 0.002 m));
bLeft.section = PipeSection(0.508 m, 0.019 m);
bLeft.name = "BEAM_LEFT";

var bRight = StraightBeam(Point(7.98 m, -0.004 m, -0.001 m), Point(16 m, 0 m, 0 m));
bRight.section = PipeSection(0.508 m, 0.019 m);
bRight.name = "BEAM_RIGHT";

var bCross = StraightBeam(Point(8 m, 0 m, 0 m), Point(8 m, 6 m, 0 m));
bCross.section = PipeSection(0.356 m, 0.012 m);
bCross.name = "BEAM_CROSS";

// BeamJoiner - merge overlapping/touching beam ends into a single node
var joiner = BeamJoiner();
joiner.add(bLeft);
joiner.add(bRight);
joiner.add(bCross);
joiner.tolerance = 0.05 m;              // 50mm snap tolerance
joiner.connectAt(Point(8 m, 0 m, 0 m)); // Target connection point
joiner.name = "JOINER_K2";

// Overlapping beam detection and resolution
var bOverlap1 = StraightBeam(Point(0 m, 5 m, 0 m), Point(12 m, 5 m, 0 m));
var bOverlap2 = StraightBeam(Point(8 m, 5 m, 0 m), Point(20 m, 5 m, 0 m));
// Beams overlap from 8m to 12m

var overlapJoiner = BeamJoiner();
overlapJoiner.add(bOverlap1);
overlapJoiner.add(bOverlap2);
overlapJoiner.tolerance = 0.1 m;
overlapJoiner.resolveOverlaps();  // Trim or merge overlapping segments
overlapJoiner.name = "OVERLAP_RESOLVER";
```

## Beam Buckling Analysis

```javascript
// BucklingLength - define effective buckling length for Euler buckling
var column = StraightBeam(Point(0 m, 0 m, 0 m), Point(0 m, 0 m, 12 m));
column.section = ISection(0.5 m, 0.3 m, 0.3 m, 0.012 m, 0.020 m, 0.020 m);
column.material = S355;
column.name = "COLUMN_C1";

// Set buckling length (distance between inflexion points)
column.BucklingLength("major_axis", 0.7 * 12 m);   // K=0.7 for fixed-pinned
column.BucklingLength("minor_axis", 1.0 * 12 m);   // K=1.0 for pinned-pinned
column.BucklingLength("torsional", 1.0 * 12 m);

// BucklingFactor - multiplier on member length for effective length
var brace1 = StraightBeam(Point(10 m, 0 m, 5 m), Point(20 m, 0 m, 15 m));
brace1.section = PipeSection(0.356 m, 0.012 m);
brace1.material = S355;
brace1.name = "BRACE_BF";

// Effective length = BucklingFactor * actual length
brace1.BucklingFactor("in_plane", 0.85);    // In-plane buckling factor
brace1.BucklingFactor("out_of_plane", 0.85); // Out-of-plane buckling factor

// Batch assign buckling factors
var braces = [brace1, kbrace, diagonal];
for (var i = 0; i < braces.length; i++) {
    braces[i].BucklingFactor("in_plane", 0.90);
    braces[i].BucklingFactor("out_of_plane", 0.90);
    print("Buckling factors set for: " + braces[i].name);
}
```

## Snap Planes for Beam Alignment

```javascript
// Create a snap plane at deck level
var snapPlaneDeck = SnapPlane();
snapPlaneDeck.planePoint = Point(0 m, 0 m, 30 m);
snapPlaneDeck.planeNormal = Vector(0, 0, 1);  // Horizontal plane Z=30
snapPlaneDeck.name = "SNAP_DECK_Z30";

// Create beams that snap to the plane
setDefault("SnapPlane", snapPlaneDeck);

var bSnap1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(0 m, 0 m, 35 m));
// End 2 will snap to Z=30m automatically
bSnap1.name = "SNAPPED_COLUMN";

var bSnap2 = StraightBeam(Point(10 m, 5 m, 0 m), Point(10 m, 5 m, 35 m));
bSnap2.name = "SNAPPED_COLUMN_2";

// Vertical snap plane for frame alignment
var snapPlaneFrame = SnapPlane();
snapPlaneFrame.planePoint = Point(0 m, 10 m, 0 m);
snapPlaneFrame.planeNormal = Vector(0, 1, 0);  // Vertical plane Y=10

// Diagonal brace snapped to frame plane
var bDiag = StraightBeam(
    Point(0 m, 0 m, 5 m),
    Point(20 m, 20 m, 25 m)   // Intersection will snap to Y=10 plane
);
bDiag.localSystem = "SnapToPlane";
bDiag.section = PipeSection(0.457 m, 0.019 m);
bDiag.name = "DIAG_BRACE_SNAPPED";

// Remove snap plane default to prevent unintended snapping
setDefault("SnapPlane", null);
```
