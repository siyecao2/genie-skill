# Unit Systems in GeniE (V8.8-08)
> **SESAM 源**: `GeniE V8.8-08 Help\ReferenceDocuments\GeniE_SectProp_Units.pdf`

## 1. SI Base Units

GeniE uses **SI units internally** for all calculations. All inputs and outputs are in SI unless explicitly converted.

| Quantity | SI Unit | GeniE String |
|----------|---------|--------------|
| Length | meter | `m` |
| Mass | kilogram | `kg` |
| Force | Newton | `N` |
| Pressure / Stress | Pascal | `Pa` |
| Time | second | `s` |
| Angle | radian | `rad` |
| Temperature | Kelvin / °C | `K` / `delC` |

## 2. Unit Suffixes

GeniE accepts the following unit strings in `Quantity` constructors:

```javascript
// Length
Quantity(5, "m");       // 5 meters
Quantity(500, "mm");    // 500 millimeters → 0.5 m
Quantity(50, "cm");     // 50 centimeters → 0.5 m
Quantity(1, "km");      // 1 kilometer → 1000 m

// Force
Quantity(1000, "N");    // 1000 Newtons
Quantity(100, "kN");    // 100 kiloNewtons → 100,000 N
Quantity(1, "MN");      // 1 megaNewton → 1,000,000 N

// Mass
Quantity(5000, "kg");   // 5000 kilograms
Quantity(5, "tonne");   // 5 metric tonnes → 5000 kg
Quantity(5000, "g");    // 5000 grams → 5 kg

// Pressure / Stress
Quantity(355, "MPa");   // 355 MPa → 355,000,000 Pa
Quantity(2.1e5, "MPa"); // 210 GPa (Young's modulus for steel)
Quantity(1, "GPa");     // 1 GPa → 1,000,000,000 Pa

// Angle
Quantity(90, "deg");    // 90 degrees → π/2 rad
Quantity(3.14159, "rad"); // radians

// Temperature coefficient
Quantity(1.2e-5, "delC^-1"); // thermal expansion coefficient
```

## 3. Quantity Types

GeniE's `Quantity` class wraps a numeric value with a physical type:

```javascript
// Creating quantities explicitly
var force = Quantity(1000, "kN");
var length = Quantity(5.5, "m");
var pressure = Quantity(2.0e8, "Pa");

// Quantities can be used directly in property assignments
beam.section = pipeSec;
pipeSec.outerDiameter = Quantity(508, "mm");
pipeSec.thickness = Quantity(12.7, "mm");
mat.density = Quantity(7850, "kg/m3");
mat.youngsModulus = Quantity(2.1e11, "Pa");
```

### Supported Quantity Dimensions
`Force`, `Length`, `Mass`, `Pressure`, `Stress`, `Acceleration`, `Angle`, `Area`, `Volume`, `Density`, `ForcePerLength`, `Moment`, `Temperature`, `ThermalExpansion`, `Damping`, `SpringStiffness`, `Time`, `Frequency`, `Velocity`

## 4. Vector3d with Units

```javascript
// Unit-aware vectors
var v = Vector3d(0, 0, -9.81);                // m/s2 (SI)
var displacement = Vector3d(0, 0, Quantity(-10, "cm")); // explicit units

// Applied to:
// - GuidePlane normals
// - Load directions
// - Copy offsets in ModelTransformer
```

## 5. Common Offshore Engineering Conversions

### Length
| From | To | Multiply by |
|------|----|-------------|
| inch (in) | meter (m) | 0.0254 |
| foot (ft) | meter (m) | 0.3048 |
| yard (yd) | meter (m) | 0.9144 |

### Force / Weight
| From | To | Multiply by |
|------|----|-------------|
| kip (kip) | Newton (N) | 4448.22 |
| ton-force (short) | Newton (N) | 8896.44 |
| ton-force (metric) | Newton (N) | 9806.65 |
| pound-force (lbf) | Newton (N) | 4.44822 |

### Pressure / Stress
| From | To | Multiply by |
|------|----|-------------|
| psi | Pascal (Pa) | 6894.76 |
| ksi | Pascal (Pa) | 6,894,760 |
| MPa | Pascal (Pa) | 1,000,000 |
| bar | Pascal (Pa) | 100,000 |

### Density
| From | To | Multiply by |
|------|----|-------------|
| lb/ft3 | kg/m3 | 16.0185 |
| lb/in3 | kg/m3 | 27,679.9 |

### Common Steel Properties (SI)
```javascript
var steelDensity = 7850;        // kg/m3
var steelE = 2.1e11;            // Pa (= 210 GPa)
var steelNu = 0.3;              // Poisson's ratio
var steelYield355 = 355e6;      // Pa (= 355 MPa)
var g = 9.81;                   // m/s2 gravity
```

## 6. Imported Model Unit Handling

When importing models from other software:

```javascript
// ImportFEM may have different unit conventions
// Check: Sestra .fem files can use any consistent unit set
// GeniE always converts to SI internally

// If importing a kips-inch model:
// - Open the .fem, check node coordinates
// - If coordinates are in inches: model was likely in kip-in units
// - GeniE will try to detect, but verify:
//   1. Compare a known dimension
//   2. Scale if needed via ModelTransformer

// Manual scaling after import:
var mt = ModelTransformer();
mt.scaleFactor = 0.0254;  // inch → meter
mt.transform(importedSet);

// Verify: a 240-inch beam should be ~6.1 m after scaling
```

## 7. Best Practices

```javascript
// 1. Always use SI internally
var legLength = 50;  // 50 meters (not 164 ft)

// 2. Use Quantity() for clarity when reading external specs
var designPressure = Quantity(5000, "psi");  // clearly from API spec
var pipeOD = Quantity(20, "in");

// 3. Avoid mixing unit systems in one script
// BAD:
var a = 5;          // meters?
var b = Quantity(12, "in"); // inches?
// GOOD: convert all to SI at input boundary
var a = 5;          // meters
var b = 0.3048;      // 12 inches → meters

// 4. Verify output magnitudes
// Expect: jacket natural period ~1-3 seconds
// Expect: deck displacement ~0.01-0.1 m under operational loads
// Expect: brace axial stress ~50-250 MPa in service
```
