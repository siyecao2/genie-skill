# Plate Modeling - GeniE SESAM Snippets

## Plate Creation - 4-Point Rectangular

```javascript
// Simple 4-point rectangular plate
var plate1 = Plate();
plate1.addPoint(Point(0 m, 0 m, 5 m));
plate1.addPoint(Point(10 m, 0 m, 5 m));
plate1.addPoint(Point(10 m, 8 m, 5 m));
plate1.addPoint(Point(0 m, 8 m, 5 m));
plate1.thickness = 20 mm;
plate1.material = S355;
plate1.name = "DECK_PLATE_01";

// Alternative: Plate with explicit point array
var corners = [
    Point(10 m, 0 m, 5 m),
    Point(20 m, 0 m, 5 m),
    Point(20 m, 8 m, 5 m),
    Point(10 m, 8 m, 5 m)
];
var plate2 = Plate(corners);
plate2.thickness = 25 mm;
plate2.material = S355;
plate2.name = "DECK_PLATE_02";
```

## Multi-Point Plate Creation

```javascript
// Irregular polygon plate (bulkhead with cut-out shape)
var platePoly = Plate();
platePoly.addPoint(Point(0 m, 0 m, 0 m));
platePoly.addPoint(Point(5 m, 0 m, 0 m));
platePoly.addPoint(Point(7 m, 3 m, 0 m));
platePoly.addPoint(Point(7 m, 6 m, 0 m));
platePoly.addPoint(Point(5 m, 9 m, 0 m));
platePoly.addPoint(Point(0 m, 9 m, 0 m));
platePoly.thickness = 15 mm;
platePoly.material = DH36;
platePoly.name = "BULKHEAD_IRREG";

// Triangular plate
var plateTri = Plate();
plateTri.addPoint(Point(0 m, 0 m, 10 m));
plateTri.addPoint(Point(6 m, 0 m, 10 m));
plateTri.addPoint(Point(3 m, 5 m, 10 m));
plateTri.thickness = 12 mm;
plateTri.name = "GUSSET_PLATE";
```

## SkinCurves - Surface Skinning

```javascript
// Create two curves for skinning
var curveBottom = ModelCurve();
curveBottom.addPoint(Point(0 m, 0 m, 0 m));
curveBottom.addPoint(Point(5 m, 0 m, 2 m));
curveBottom.addPoint(Point(10 m, 0 m, 0 m));
curveBottom.name = "BOTTOM_CURVE";

var curveTop = ModelCurve();
curveTop.addPoint(Point(0 m, 8 m, 0 m));
curveTop.addPoint(Point(5 m, 8 m, 3 m));
curveTop.addPoint(Point(10 m, 8 m, 0 m));
curveTop.name = "TOP_CURVE";

// Skin surface from curves
var skinPlate = SkinCurves([curveBottom, curveTop]);
skinPlate.thickness = 18 mm;
skinPlate.material = S355;
skinPlate.name = "SKINNED_PLATE";
```

## FlatPlate Creation

```javascript
// FlatPlate - ensures a perfectly flat plate on a defined plane
// Point + normal vector definition
var fpDeck = FlatPlate(
    Point(0 m, 0 m, 30 m),   // point on plane
    Vector(0, 0, 1),          // normal (Z-up for horizontal deck)
    20 m, 15 m                // dimensions
);
fpDeck.thickness = 30 mm;
fpDeck.material = S355;
fpDeck.name = "DECK_FLAT";

// FlatPlate for bulkhead (vertical plane)
var fpBulkhead = FlatPlate(
    Point(10 m, 0 m, 0 m),   // point on plane
    Vector(1, 0, 0),          // normal in X direction
    8 m, 30 m                 // dimensions (Y × Z)
);
fpBulkhead.thickness = 16 mm;
fpBulkhead.name = "BULKHEAD_FLAT";
```

## Thickness Assignment

```javascript
// Direct thickness assignment
var p1 = Plate();
p1.addPoint(Point(0 m, 0 m, 0 m));
p1.addPoint(Point(5 m, 0 m, 0 m));
p1.addPoint(Point(5 m, 5 m, 0 m));
p1.addPoint(Point(0 m, 5 m, 0 m));
p1.thickness = 25 mm;
p1.name = "BOTTOM_SHELL";

// Variable thickness via thickness property object
var thickVar = Thickness();
thickVar.value = 22 mm;
p1.thickness = thickVar;

// Assign thickness after plate decomposition
var subPlates = p1.explode(IndexedNameMask("SUB_"));
for (var i = 0; i < subPlates.length; i++) {
    subPlates[i].thickness = 20 mm;
    subPlates[i].material = S355;
}
```

## explode() with IndexedNameMask

```javascript
// explode() decomposes a complex plate into simpler sub-plates
var complexPlate = Plate();
complexPlate.addPoint(Point(0 m, 0 m, 0 m));
complexPlate.addPoint(Point(8 m, 0 m, 0 m));
complexPlate.addPoint(Point(10 m, 3 m, 0 m));
complexPlate.addPoint(Point(10 m, 6 m, 0 m));
complexPlate.addPoint(Point(8 m, 9 m, 0 m));
complexPlate.addPoint(Point(0 m, 9 m, 0 m));
complexPlate.thickness = 20 mm;
complexPlate.name = "COMPLEX_BULKHEAD";

// Explode into sub-plates with indexed naming
var nameMask = IndexedNameMask("PLATE_PART_");
var subPlates = complexPlate.explode(nameMask);
// Creates: PLATE_PART_1, PLATE_PART_2, PLATE_PART_3, ...

// Assign individual properties to sub-plates
for (var j = 0; j < subPlates.length; j++) {
    subPlates[j].material = S355;
    print("Created: " + subPlates[j].name);
}
```

## join() for Merging Plates

```javascript
// Create two adjacent plates
var pA = Plate();
pA.addPoint(Point(0 m, 0 m, 0 m));
pA.addPoint(Point(5 m, 0 m, 0 m));
pA.addPoint(Point(5 m, 5 m, 0 m));
pA.addPoint(Point(0 m, 5 m, 0 m));
pA.thickness = 15 mm;
pA.name = "PLATE_A";

var pB = Plate();
pB.addPoint(Point(5 m, 0 m, 0 m));
pB.addPoint(Point(10 m, 0 m, 0 m));
pB.addPoint(Point(10 m, 5 m, 0 m));
pB.addPoint(Point(5 m, 5 m, 0 m));
pB.thickness = 15 mm;
pB.name = "PLATE_B";

// Join them into one plate
var pMerged = pA.join(pB);
pMerged.name = "MERGED_PLATE";
```

## simplifyTopology() & Delete() Cleanup

```javascript
// After modifying plates, simplify topology
simplifyTopology();

// Delete specific plate primitives
var tempPlate = Plate();
tempPlate.addPoint(Point(0 m, 0 m, 0 m));
tempPlate.addPoint(Point(2 m, 0 m, 0 m));
tempPlate.addPoint(Point(2 m, 2 m, 0 m));
tempPlate.addPoint(Point(0 m, 2 m, 0 m));
tempPlate.name = "TEMP_DELETE";

// Delete the temporary plate
Delete(tempPlate);

// Primitives cleanup pattern
var unusedPlates = []; // collect plates to delete
// ... modeling operations ...
for (var k = 0; k < unusedPlates.length; k++) {
    Delete(unusedPlates[k]);
}
```

## primitivePartCount Validation

```javascript
// Validate that a plate decomposed into expected number of parts
var sourcePlate = Plate();
sourcePlate.addPoint(Point(0 m, 0 m, 0 m));
sourcePlate.addPoint(Point(10 m, 0 m, 0 m));
sourcePlate.addPoint(Point(10 m, 10 m, 0 m));
sourcePlate.addPoint(Point(0 m, 10 m, 0 m));
sourcePlate.thickness = 20 mm;
sourcePlate.name = "CHECK_PLATE";

var subParts = sourcePlate.explode(IndexedNameMask("PART_"));
print("Decomposed into " + subParts.length + " primitive parts");

if (subParts.length > 0) {
    print("Plate mesh ready for " + subParts.length + " sub-parts");
}
```

## Plate Creation from GuidePlane

```javascript
// Create GuidePlane first
var gpSegment = GuidePlane();
gpSegment.addPoint(Point(0 m, 0 m, 20 m));
gpSegment.addPoint(Point(15 m, 0 m, 20 m));
gpSegment.addPoint(Point(15 m, 10 m, 20 m));
gpSegment.addPoint(Point(0 m, 10 m, 20 m));
gpSegment.name = "SEGMENT_GUIDE";

// Extract points from GuidePlane to create plate
var pts = [gpSegment.point(0), gpSegment.point(1), gpSegment.point(2), gpSegment.point(3)];
var plateFromGP = Plate(pts);
plateFromGP.thickness = 25 mm;
plateFromGP.material = S355;
plateFromGP.name = "PLATE_FROM_GUIDEPLANE";
```