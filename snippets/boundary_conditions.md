# Boundary Conditions - GeniE SESAM Snippets

## SupportPoint & BoundaryCondition (6 DOF)

```javascript
// Create a support point at a node location
var sp1 = SupportPoint(Point(0 m, 0 m, 0 m));
sp1.name = "FIXED_BASE_01";

// Boundary condition with full fixity (all 6 DOF fixed)
var bcFixed = BoundaryCondition();
bcFixed.Tx = Fixed;   // Translation X
bcFixed.Ty = Fixed;   // Translation Y
bcFixed.Tz = Fixed;   // Translation Z
bcFixed.Rx = Fixed;   // Rotation X
bcFixed.Ry = Fixed;   // Rotation Y
bcFixed.Rz = Fixed;   // Rotation Z
sp1.boundaryCondition = bcFixed;

// Pinned support (translations fixed, rotations free)
var sp2 = SupportPoint(Point(10 m, 0 m, 0 m));
sp2.name = "PINNED_BASE_02";
var bcPinned = BoundaryCondition();
bcPinned.Tx = Fixed;
bcPinned.Ty = Fixed;
bcPinned.Tz = Fixed;
bcPinned.Rx = Free;
bcPinned.Ry = Free;
bcPinned.Rz = Free;
sp2.boundaryCondition = bcPinned;

// Spring support with stiffness values
var sp3 = SupportPoint(Point(20 m, 0 m, 0 m));
sp3.name = "SPRING_BASE_03";
var bcSpring = BoundaryCondition();
bcSpring.Tx = Spring;
bcSpring.TxStiffness = 1e8 N/m;
bcSpring.Ty = Spring;
bcSpring.TyStiffness = 1e8 N/m;
bcSpring.Tz = Fixed;
bcSpring.Rx = Spring;
bcSpring.RxStiffness = 1e7 Nm/rad;
bcSpring.Ry = Spring;
bcSpring.RyStiffness = 1e7 Nm/rad;
bcSpring.Rz = Free;
sp3.boundaryCondition = bcSpring;
```

## SupportCurve with Boundary Stiffness Per Length

```javascript
// Support along a curve with distributed stiffness
var supCurve = SupportCurve();
supCurve.addPoint(Point(0 m, -3 m, 0 m));
supCurve.addPoint(Point(15 m, -3 m, 0 m));
supCurve.name = "SEABED_SUPPORT";

// Boundary stiffness per unit length of the curve
var bcDistributed = BoundaryCondition();
bcDistributed.Tz = Spring;
bcDistributed.TzStiffnessPerLength = 5e7 N/m/m;  // 50 MN/m per meter
bcDistributed.Tx = Spring;
bcDistributed.TxStiffnessPerLength = 1e7 N/m/m;
bcDistributed.Ty = Spring;
bcDistributed.TyStiffnessPerLength = 1e7 N/m/m;
supCurve.boundaryCondition = bcDistributed;
```

## SupportRigidLink

```javascript
// Rigid link connecting support point to structural nodes
var spCenter = SupportPoint(Point(5 m, 4 m, 0 m));
spCenter.name = "RIGID_MASTER";

// Create slave points that are rigidly linked to master
var rigidLink = SupportRigidLink();
rigidLink.masterPoint = spCenter;
rigidLink.addSlavePoint(Point(3 m, 2 m, 0 m));
rigidLink.addSlavePoint(Point(7 m, 2 m, 0 m));
rigidLink.addSlavePoint(Point(3 m, 6 m, 0 m));
rigidLink.addSlavePoint(Point(7 m, 6 m, 0 m));
rigidLink.name = "RIGID_FOOTPRINT";
```

## LocalSystem on Supports

```javascript
// ConstantLocalSystem - fixed orientation
var csGlobal = ConstantLocalSystem();
csGlobal.xAxis = Vector(1, 0, 0);
csGlobal.yAxis = Vector(0, 1, 0);
csGlobal.name = "GLOBAL_CS";

// GuideLocalSystem - orientation follows a guide
var glsSupport = GuideLocalSystem(
    Point(0 m, 0 m, 0 m),
    Vector(1, 0, 0),     // local X
    Vector(0, 1, 0)      // local Y (Z = X × Y)
);

// Apply local system to support
var spLocal = SupportPoint(Point(10 m, 5 m, -5 m));
spLocal.localSystem = glsSupport;
spLocal.name = "LOCAL_ORIENTED_SUPPORT";

var bcLocal = BoundaryCondition();
bcLocal.Tx = Fixed;       // Fixed in *local* X direction
bcLocal.Ty = Free;        // Free in *local* Y direction
bcLocal.Tz = Fixed;
bcLocal.Rx = Free;
bcLocal.Ry = Free;
bcLocal.Rz = Fixed;
spLocal.boundaryCondition = bcLocal;
```

## Footprint Types

```javascript
// FootprintPoint - concentrated support
var fpPoint = FootprintPoint();
fpPoint.position = Point(5 m, 5 m, 0 m);
var spFP = SupportPoint(fpPoint);
spFP.name = "POINT_FOOTPRINT";

// FootprintBox - rectangular area support
var fpBox = FootprintBox();
fpBox.width = 2 m;
fpBox.length = 2 m;
fpBox.position = Point(10 m, 5 m, 0 m);
var spBox = SupportPoint(fpBox);
spBox.name = "BOX_FOOTPRINT";

// FootprintCylinder - circular area support
var fpCyl = FootprintCylinder();
fpCyl.radius = 1.5 m;
fpCyl.position = Point(15 m, 5 m, 0 m);
var spCyl = SupportPoint(fpCyl);
spCyl.name = "CYLINDER_FOOTPRINT";

// FootprintSphere - spherical contact support
var fpSph = FootprintSphere();
fpSph.radius = 1.0 m;
fpSph.position = Point(20 m, 5 m, 0 m);
var spSph = SupportPoint(fpSph);
spSph.name = "SPHERE_FOOTPRINT";

// FootprintLine - linear support
var fpLine = FootprintLine();
fpLine.startPoint = Point(0 m, 0 m, 0 m);
fpLine.endPoint = Point(8 m, 0 m, 0 m);
var spLine = SupportPoint(fpLine);
spLine.name = "LINE_FOOTPRINT";
```

## Symmetry Boundary Conditions

```javascript
// XY-plane symmetry (Z = 0): constrain Tz, Rx, Ry
var spSymXY = SupportPoint(Point(0 m, 0 m, 0 m));
var bcSymXY = BoundaryCondition();
bcSymXY.Tx = Free;
bcSymXY.Ty = Free;
bcSymXY.Tz = Fixed;    // Out-of-plane translation fixed
bcSymXY.Rx = Fixed;    // In-plane rotations fixed
bcSymXY.Ry = Fixed;
bcSymXY.Rz = Free;     // Out-of-plane rotation free
spSymXY.boundaryCondition = bcSymXY;

// XZ-plane symmetry (Y = 0): constrain Ty, Rx, Rz
var bcSymXZ = BoundaryCondition();
bcSymXZ.Tx = Free;
bcSymXZ.Ty = Fixed;
bcSymXZ.Tz = Free;
bcSymXZ.Rx = Fixed;
bcSymXZ.Ry = Free;
bcSymXZ.Rz = Fixed;

// YZ-plane symmetry (X = 0): constrain Tx, Ry, Rz
var bcSymYZ = BoundaryCondition();
bcSymYZ.Tx = Fixed;
bcSymYZ.Ty = Free;
bcSymYZ.Tz = Free;
bcSymYZ.Rx = Free;
bcSymYZ.Ry = Fixed;
bcSymYZ.Rz = Fixed;
```

## Boundary Type Comparison

```javascript
// Helper function to create common boundary types
function createSupport(type, position) {
    var sp = SupportPoint(position);
    var bc = BoundaryCondition();

    switch (type) {
        case "FULLY_FIXED":
            bc.Tx = Fixed; bc.Ty = Fixed; bc.Tz = Fixed;
            bc.Rx = Fixed; bc.Ry = Fixed; bc.Rz = Fixed;
            break;
        case "PINNED":
            bc.Tx = Fixed; bc.Ty = Fixed; bc.Tz = Fixed;
            bc.Rx = Free;  bc.Ry = Free;  bc.Rz = Free;
            break;
        case "ROLLER_X":
            bc.Tx = Free;  bc.Ty = Fixed; bc.Tz = Fixed;
            bc.Rx = Free;  bc.Ry = Free;  bc.Rz = Free;
            break;
        case "FREE":
            bc.Tx = Free; bc.Ty = Free; bc.Tz = Free;
            bc.Rx = Free; bc.Ry = Free; bc.Rz = Free;
            break;
    }

    sp.boundaryCondition = bc;
    sp.name = type + "_SUPPORT";
    return sp;
}

// Usage
var spFixed = createSupport("FULLY_FIXED", Point(0 m, 0 m, 0 m));
var spPinned = createSupport("PINNED", Point(10 m, 0 m, 0 m));
var spRoller = createSupport("ROLLER_X", Point(20 m, 0 m, 0 m));
```