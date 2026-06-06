# Model Transforms - GeniE SESAM Snippets

## ObjectNameMap Creation Pattern

```javascript
// ObjectNameMap - maps source object names to target names during copy
// Useful for batch renaming after transforms

// Basic name mapping
var nameMap = ObjectNameMap();
nameMap.addMapping("LEG_A1", "LEG_A2");
nameMap.addMapping("BRACE_K1", "BRACE_K2");
nameMap.addMapping("PLATE_DECK_01", "PLATE_DECK_02");

// Indexed name mapping pattern
var indexedMap = ObjectNameMap();
for (var i = 1; i <= 4; i++) {
    indexedMap.addMapping("COLUMN_" + i, "COLUMN_" + (i + 4));
}

// Wildcard-style mapping (prefix replacement)
var prefixMap = ObjectNameMap();
prefixMap.setPrefixMapping("BAY1_", "BAY2_");
// Automatically maps BAY1_LEG → BAY2_LEG, BAY1_BRACE → BAY2_BRACE, etc.
```

## copyTranslate - Single Copy

```javascript
// Single copy translate - one copy at offset distance
var bay1Beams = getAllBeamsInBay(1);  // Collect all beams in Bay 1
var bay1Plates = getAllPlatesInBay(1);

// Copy translate entire bay by 20m in X direction
ModelTransformer.copyTranslate(
    bay1Beams.concat(bay1Plates),
    Vector(20 m, 0 m, 0 m),
    ObjectNameMap().setPrefixMapping("BAY1_", "BAY2_")
);

// Copy translate a single beam
var beam1 = StraightBeam(Point(0 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
beam1.name = "BEAM_ORIGINAL";

var copiedBeam = ModelTransformer.copyTranslate(
    beam1,
    Vector(0 m, 5 m, 0 m),
    ObjectNameMap().addMapping("BEAM_ORIGINAL", "BEAM_COPY")
);
```

## copyTranslate - Multi-Copy

```javascript
// Multi-copy translate - create N copies at regular intervals
var templateBay = getAllStructuralElementsInRange(0 m, 20 m);

// Create 5 copies at 20m intervals in X direction
var copies = ModelTransformer.copyTranslate(
    templateBay,
    Vector(20 m, 0 m, 0 m),
    5,  // Number of copies
    ObjectNameMap().setPrefixMapping("BAY1_", "BAY_COPY_")
);

// Generate 10 frames at 5m spacing
var frameTemplate = getSingleFrame(0 m);
ModelTransformer.copyTranslate(
    frameTemplate,
    Vector(0 m, 0 m, 5 m),
    10,
    ObjectNameMap().setPrefixMapping("FRAME_0_", "FRAME_")
);

// Jacket brace bay replication
var bracePanel = getBracePanel(0 m);  // Get brace pattern at Z=0
ModelTransformer.copyTranslate(
    bracePanel,
    Vector(0 m, 0 m, 15 m),   // 15m bay height
    4,                          // 4 bays
    ObjectNameMap().setPrefixMapping("BAY0", "BAY")
);
```

## copyRotate - Single & Multi-Step

```javascript
// Single rotation copy - one copy at specified angle
var leg = StraightBeam(Point(5 m, 0 m, 0 m), Point(5 m, 0 m, 30 m));
leg.name = "LEG_0DEG";
leg.section = PipeSection(0.8 m, 0.04 m);

// Rotate copy 90 degrees around Z-axis through origin
var leg90 = ModelTransformer.copyRotate(
    leg,
    Point(0 m, 0 m, 0 m),   // rotation center
    Vector(0, 0, 1),          // rotation axis (Z)
    90 deg,                    // rotation angle
    ObjectNameMap().addMapping("LEG_0DEG", "LEG_90DEG")
);

// Multi-step rotation - 4-fold symmetry (4 legs at 90° intervals)
var legTemplate = StraightBeam(Point(8 m, 0 m, 0 m), Point(8 m, 0 m, 40 m));
legTemplate.name = "LEG_TEMPLATE";

var legs = ModelTransformer.copyRotate(
    legTemplate,
    Point(0 m, 0 m, 0 m),   // center
    Vector(0, 0, 1),          // Z-axis
    90 deg,                    // step angle
    3                          // 3 copies (total 4 including original)
);

// Create circular array of braces
var braceProto = StraightBeam(Point(4 m, 0 m, 20 m), Point(6 m, 0 m, 22 m));
braceProto.name = "BRACE_0";
var braces = ModelTransformer.copyRotate(
    braceProto,
    Point(0 m, 0 m, 20 m),
    Vector(0, 0, 1),
    45 deg,
    7  // 7 copies at 45° = full circle with original
);
```

## copyMirror Operation

```javascript
// Mirror across a plane defined by normal vector and point
var leftBay = getAllElementsInBay("LEFT");
var nameMap = ObjectNameMap();
nameMap.setPrefixMapping("LEFT_", "RIGHT_");

// Mirror across YZ-plane (X=0)
var rightBay = ModelTransformer.copyMirror(
    leftBay,
    Point(0 m, 0 m, 0 m),    // point on mirror plane
    Vector(1, 0, 0),           // normal of mirror plane (X-direction)
    nameMap
);

// Mirror half-model to create full model
var halfModel = getAllStructuralElements();
var nameMapFull = ObjectNameMap();
nameMapFull.setPrefixMapping("HALF_", "MIRROR_");

var mirroredHalf = ModelTransformer.copyMirror(
    halfModel,
    Point(0 m, 0 m, 0 m),
    Vector(0, 1, 0),  // Mirror across XZ-plane
    nameMapFull
);

// Mirror individual plate
var plateLeft = Plate();
plateLeft.addPoint(Point(0 m, -5 m, 10 m));
plateLeft.addPoint(Point(10 m, -5 m, 10 m));
plateLeft.addPoint(Point(10 m, 0 m, 10 m));
plateLeft.addPoint(Point(0 m, 0 m, 10 m));
plateLeft.name = "HULL_LEFT";
plateLeft.thickness = 18 mm;

var plateRight = ModelTransformer.copyMirror(
    plateLeft,
    Point(0 m, 0 m, 0 m),
    Vector(0, 1, 0),  // Mirror across XZ-plane
    ObjectNameMap().addMapping("HULL_LEFT", "HULL_RIGHT")
);
```

## Connected vs Unconnected Copy

```javascript
// DefaultConnectedCopy - elements at mirror plane share nodes
// (useful for symmetric half-models)
DefaultConnectedCopy = true;
var connectedBay = ModelTransformer.copyTranslate(
    bayElements,
    Vector(10 m, 0 m, 0 m),
    ObjectNameMap().setPrefixMapping("BAY1_", "BAY2_")
);
// Nodes on the common boundary are merged

// Disconnected copy - completely independent copy
DefaultConnectedCopy = false;
var independentBay = ModelTransformer.copyTranslate(
    bayElements,
    Vector(30 m, 0 m, 0 m),
    ObjectNameMap().setPrefixMapping("BAY1_", "BAY3_")
);
// No node sharing - completely separate structure

// Reset to default after operations
DefaultConnectedCopy = true;
```

## GuidePlane Copying Alongside Structural Elements

```javascript
// When copying structural elements, also copy GuidePlanes for consistency
var gpBase = GuidePlane();
gpBase.addPoint(Point(0 m, 0 m, 0 m));
gpBase.addPoint(Point(15 m, 0 m, 0 m));
gpBase.addPoint(Point(15 m, 10 m, 0 m));
gpBase.addPoint(Point(0 m, 10 m, 0 m));
gpBase.name = "GP_BAY1";

var structuralElements = getElementsInGuidePlane(gpBase);

// Copy GuidePlane and elements together
var gpCopied = gpBase.copyTranslate(Vector(0 m, 0 m, 15 m));
gpCopied.name = "GP_BAY2";

var nameMap = ObjectNameMap();
nameMap.setPrefixMapping("BAY1_", "BAY2_");
ModelTransformer.copyTranslate(
    structuralElements,
    Vector(0 m, 0 m, 15 m),
    nameMap
);
```

## moveTranslate for Relocating Objects

```javascript
// Move objects without creating copies
var wrongPositionBeam = StraightBeam(Point(5 m, 0 m, 3 m), Point(15 m, 0 m, 3 m));
wrongPositionBeam.name = "MISPLACED_BEAM";
wrongPositionBeam.section = pipeMain;

// Move to correct position
ModelTransformer.moveTranslate(
    wrongPositionBeam,
    Vector(0 m, 0 m, 7 m)  // Move up by 7m
);

// Move a group of elements
var deckElements = getDeckElementsAtLevel(25 m);
var correction = Vector(0 m, 2 m, 5 m);  // Adjust Y and Z
ModelTransformer.moveTranslate(deckElements, correction);

// Move after creation
var tempBeam = StraightBeam(Point(0 m, 0 m, 0 m), Point(10 m, 0 m, 0 m));
tempBeam.section = pipeBrace;
ModelTransformer.moveTranslate(tempBeam, Vector(3 m, 4 m, 12 m));
tempBeam.name = "REPOSITIONED_BEAM";
```

## Set-Based Operations

```javascript
// Use Set to manage groups of objects for batch operations
var baySet = Set();
baySet.name = "BAY_2_ELEMENTS";

// Add elements to the set
var beamsB2 = getBeamsByPattern("BAY2_*");
var platesB2 = getPlatesByPattern("BAY2_*");

for (var i = 0; i < beamsB2.length; i++) {
    baySet.add(beamsB2[i]);
}
for (var j = 0; j < platesB2.length; j++) {
    baySet.add(platesB2[j]);
}

// Batch move translate on the entire set
ModelTransformer.moveTranslate(baySet, Vector(0 m, 0 m, 5 m));

// Batch copy rotate on the set
var rotatedCopy = ModelTransformer.copyRotate(
    baySet,
    Point(0 m, 0 m, 0 m),
    Vector(0, 0, 1),
    180 deg,
    ObjectNameMap().setPrefixMapping("BAY2_", "BAY2_ROT_")
);

// Verify set contents
print("Set contains " + baySet.size() + " elements");
var elements = baySet.elements();
for (var k = 0; k < elements.length; k++) {
    print("  " + elements[k].name);
}
```