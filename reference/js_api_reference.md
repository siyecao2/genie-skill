# GeniE JavaScript API Reference (V8.8-08)

## 1. API Documentation Location

The official API documentation consists of 700+ HTML files located at:

```
{SESAM_HOME}\Program Files\DNV\GeniE V8.8-08\Help\jscript\
```

### Key Entry Points
- **index.html** — top-level namespace/class listing
- **genindex.html** — alphabetical index of all classes and methods
- **toc.html** — table of contents (class hierarchy)
- Open `index.html` in a browser for interactive navigation.

### Quick Lookup
```javascript
// In GeniE scripting console, use:
help("Beam");          // prints summary of the Beam class
help("Section");       // prints Section class info
```

## 2. Class Category Overview

### Materials
| Class | Purpose |
|-------|---------|
| `Material` | Base material class, library lookup |
| `MaterialLinear` | Isotropic linear elastic material |
| `MaterialOrthotropic` | Orthotropic material for composites/plates |
| `MaterialNonLinear` | Nonlinear material behavior |
| `MaterialTemperatureDependent` | Material properties as function of temperature |

### Sections (Cross-sections)
| Class | Purpose |
|-------|---------|
| `Section` | Generic – queries library by profile name string |
| `PipeSection` | Circular hollow section by OD × thickness |
| `ISection` | Symmetric I-beam: web + top/bottom flanges |
| `UnsymISection` | Asymmetric I-beam |
| `BoxSection` | Rectangular hollow box |
| `ChannelSection` | C-channel section |
| `ChannelFilletSection` | C-channel with fillet radii |
| `BarSection` | Solid rectangular bar |
| `TBarSection` | T-shaped section |
| `AngleSection` | L-shaped angle |
| `GeneralSection` | Arbitrary polygon cross-section |
| `ConeSection` | Tapered/cone section |
| `DblWebPlateGirderSection` | Double-web plate girder |
| `BoxedPlateGirderSection` | Boxed plate girder |
| `CompositeSection` | Composite (steel+concrete) |

### GuidingGeometry
| Class | Purpose |
|-------|---------|
| `GuidePlane` | Infinite plane: `GuidePlane(Point, Vector3d)` or 3-Point |
| `GuideLine` | Infinite line: `GuideLine(Point, Vector3d)` or 2-Point |
| `GuideSpline` | Spline through control points |
| `GuideBezier` | Bézier curve |
| `GuideNURBS` | NURBS curve |
| `GuideEllipse` | Elliptical guide curve |
| `GuideGCC` | General conic curve |
| `GuideShapePreservingSpline` | Shape-preserving spline |
| `GuideCurve` | General curve reference |
| `GuideLocalSystem` | Local coordinate system for reference |

### Structure
| Class | Purpose |
|-------|---------|
| `Beam` | Generic beam (JS auto-selects StraightBeam or CurvedBeam) |
| `StraightBeam` | Linear beam between two points |
| `CurvedBeam` | Arced beam along guide curve |
| `Plate` | Generic plate |
| `FlatPlate` | Planar plate by polygonal boundary |
| `CurvedPlate` | Curved plate by surface boundary |
| `Equipment` | Point mass/stiffness equipment |
| `SupportPoint` | Boundary condition at a point |
| `SupportCurve` | Boundary condition along a curve/edge |
| `SupportLine` | Boundary condition along a line |
| `SupportSurface` | Boundary condition on a surface |
| `SupportRigidLink` | Rigid link constraint |
| `Joint` | Structural joint connection |
| `Concept` | Abstract model entity |
| `Compartment` | Enclosed volume for hydrostatic/buoyancy |
| `Set` | Group of structural entities |
| `LoadApplicationArea` | Area for load distribution |
| `WebFrame` | Web frame stiffener |
| `Stiffener` | Plate stiffener |

### Loads
| Class | Purpose |
|-------|---------|
| `LoadCase` | Container for a set of loads |
| `LoadCombination` | Combination of load cases with factors |
| `PointLoad` | Concentrated force at a point |
| `PointForceMoment` | Force + moment at a point |
| `LineLoad` | Distributed load along a line/beam |
| `SurfaceLoad` | Distributed load on a surface |
| `Pressure2dConstant` | Constant pressure on a 2D element |
| `Pressure2dLinear` | Linearly varying pressure |
| `Component1dLinear` | Linear varying 1D component load |
| `FootprintPoint` | Point footprint load |
| `FootprintLine` | Line footprint load |
| `FootprintPolygon` | Polygon area footprint load |
| `VolumeLoad` | Body force (gravity, acceleration) |
| `TemperatureLoad` | Thermal load case |
| `EquipmentLoad` | Load on equipment mass points |

### Analysis
| Class | Purpose |
|-------|---------|
| `Analysis` | Top-level analysis activity container |
| `LinearAnalysis` | Linear static analysis definition |
| `EigenValueAnalysis` | Natural frequency / buckling analysis |
| `DynamicAnalysis` | Time-history / spectral dynamic analysis |
| `MeshActivity` | Meshing activity definition |
| `LoadResultsActivity` | **Required** to map results back to analysis |
| `CodeCheckActivity` | Code checking activity |
| `PileSoilActivity` | Pile-soil interaction analysis |
| `FrequencyResponseAnalysis` | Frequency domain analysis |

### Code Check
| Class | Purpose |
|-------|---------|
| `CapacityManager` | Top-level code check manager |
| `CapacityRun` | A code check run configuration |
| `CapacityMember` | Member-level code check definition |
| `CapacityPanel` | Panel/plate buckling code check |
| `CapacityJoint` | Tubular joint strength check |
| `CapacityRule` | Design code (NORSOK, ISO, AISC, etc.) |

### Import / Export
| Class | Purpose |
|-------|---------|
| `ImportFEM` | Import Sestra FEM file |
| `ExportFEM` | Export analysis FEM model |
| `ImportGeniE` | Import another GeniE model |
| `ImportXML` | Import XML model data |
| `ImportSTEP` | Import STEP geometry |
| `ExportResults` | Export analysis results |
| `ExportWadam` | Export Wadam wave-load model |
| `ExportUsfos` | Export to Usfos nonlinear analysis |
| `ExportSesamXTF` | Export to Sesam XTF format |

## 3. Useful API Patterns

### Finding a Class
```javascript
// Open index.html in browser, Ctrl+F for class name
// Or use the genindex.html alphabetical listing

// In scripting console:
help("GuidePlane");   // summary
help("Beam");         // summary with methods
```

### Common Constructor Patterns
```javascript
// Points
var p1 = Point(0, 0, 0);
var p2 = Point(10, 0, 5);

// Guide planes
var gp1 = GuidePlane(p1, Vector3d(0, 0, 1));       // point + normal
var gp2 = GuidePlane(Point(0,0,0), Point(10,0,0), Point(0,10,0)); // 3 points

// Beams
var b1 = Beam(p1, p2);                               // straight
var b2 = Beam(p1, p2, guideCurve);                   // curved along guide

// Plates
var plate1 = FlatPlate(gp, [p1, p2, p3, p4]);       // on guide plane
var plate2 = FlatPlate([p1, p2, p3, p4]);           // 3+ coplanar points
```

## 4. Navigating the Documentation

1. **Start** with `index.html` — shows the full namespace tree.
2. **Search** `genindex.html` for a specific class/method name.
3. Each class page lists: constructors, properties, methods, and cross-references.
4. Pay attention to **inheritance** — many methods are defined on parent classes.
5. Property types are shown as `Quantity` with units or as other GeniE class types.
