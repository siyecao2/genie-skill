# GeniE 半潜式平台模型模板 (Semi-Submersible Model Template)

> **严格参照**: SESAM `Help\Tutorials\TutorialsAdvancedModelling\A2_GeniE_Semisub_Pontoon\JS\GeniE_Semisub_Pontoon_input.js` (1238行) 和 `A7_GeniE_Semisub_Panel_and_FE\JS\GeniE_Semisub_Panel_and_FE_input.js` (915行)

```javascript
// ============================================================
// 阶段 0: GenieRules
// ============================================================
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 2 deg;
GenieRules.Transformation.FlattenWhenMirroring = false;

// ============================================================
// 阶段 1: 材料定义
// ============================================================
Steel = MaterialLinear(200000000 Pa, 7850 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
Steel.name = "Steel";
SuperMaterial = MaterialLinear(200000000 Pa, 1780 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
SuperMaterial.name = "SuperMaterial (thick plate density compensation)";

Steel.setDefault(Material);

// ============================================================
// 阶段 2: 板厚
// ============================================================
Th35 = Thickness(0.035 m);
Th35.name = "Th_35mm_PontoonOuter";
Th_on = EndOn(Th35);  // 厚度方向

Th02 = Thickness(0.002 m);
Th02.name = "Th_02mm_Auxiliary";

// ============================================================
// 阶段 3: Guiding Geometry (参照 A2: PolyCurve 外板 + GuideLine 引导)
// ============================================================
// 浮筒外板轮廓 (PolyCurve)
var pontoonCurve = PolyCurve(Array(
    Point(-20.0 m, -20.0 m, 5.0 m),
    Point(-20.0 m,  -5.0 m, 5.0 m),
    Point(-15.0 m,  -5.0 m, 3.0 m),    // 渐变过渡
    Point(-15.0 m,   0.0 m, 3.0 m),
    Point(-18.0 m,   5.0 m, 5.0 m),
    Point(-20.0 m,  10.0 m, 5.0 m)
));

// 扫掠方向
var sweepDir = Vector3d(0, 40, 0);  // Y 方向 40m

// ============================================================
// 阶段 4: 曲面建模 (SkinCurves + SweepCurve)
// ============================================================
// 底面曲线
var bottomCurve = GuideLine(Point(-20, -20, 5), Point(-20, 10, 5));

// 顶面曲线 (偏移)
var topCurve = GuideLine(Point(-20, -20, 12), Point(-20, 10, 12));

// 侧面蒙皮
PontoonSide = SkinCurves(Array(bottomCurve, topCurve));
PontoonSide.name = "PontoonSide";

// 扫掠底面
PontoonBottom = SweepCurve(bottomCurve, Vector3d(40, 0, 0));
PontoonBottom.name = "PontoonBottom";

// ============================================================
// 阶段 5: 立柱 (参照 A7: Cylinder sector)
// ============================================================
var colCenter = Point(0, 0, 12);
var colRadius = 6.0 m;
var colHeight = 30.0 m;

// 立柱底面圆
var colBaseArc = GuideEllipse(ColCenter, colRadius, colRadius, Vector3d(0,0,1));
// 立柱顶面圆
var colTopArc = GuideEllipse(Point(0,0,42), colRadius, colRadius, Vector3d(0,0,1));

// 蒙皮创建立柱曲面
ColumnShell = SkinCurves(Array(colBaseArc, colTopArc));
ColumnShell.name = "ColumnShell";

// ============================================================
// 阶段 6: 舱内水平板 (CoverCurves)
// ============================================================
BottomPlate = CoverCurves(colBaseArc);
BottomPlate.name = "BottomPlate";

TopPlate = CoverCurves(colTopArc);
TopPlate.name = "TopPlate";

// ============================================================
// 阶段 7: 加劲肋 (T-bar on plates)
// ============================================================
var stiffSpacing = 0.8 m;  // 800mm 间距

// ============================================================
// 阶段 8: 1/4 → 1/2 → 完整模型 (ModelTransformer)
// ============================================================
// 通过镜像复制构建完整半潜平台
// ModelTransformer 模式: 复制 X 方向 → 复制 Y 方向

// ============================================================
// 阶段 9: 湿表面与舱室 (Panel Model)
// ============================================================
// 创建湿表面属性
WS1 = WetSurface();
WS1.name = "WetSurface_External";

// 分配至外板 (Front = 外侧)
PontoonSide.front.wetSurface = WS1;
ColumnShell.front.wetSurface = WS1;

// 虚拟水压载荷 (Wadam/HydroD 识别)
LC_wet = DummyHydroLoadCase(WS1);
LC_wet.name = "DummyHydroPressure";

// 识别封闭舱作为压载舱
cm = CompartmentManager();

// ============================================================
// 阶段 10: 边界条件 (模型对称, 3点约束刚体位移)
// ============================================================
var sp1 = SupportPoint(Point(-20, 0, 12));
sp1.boundary = BoundaryCondition(Fixed, Fixed, Free, Free, Free, Free);

var sp2 = SupportPoint(Point(20, 0, 12));
sp2.boundary = BoundaryCondition(Fixed, Fixed, Free, Free, Free, Free);

var sp3 = SupportPoint(Point(0, -20, 12));
sp3.boundary = BoundaryCondition(Free, Fixed, Free, Free, Free, Free);

// ============================================================
// 阶段 11: 载荷
// ============================================================
LCGrav = LoadCase(0, 0, -9.81);
LCGrav.name = "Gravity";

// 设备 (Eccentric-Mass 用于水动力分析)
Equip1 = PrismEquipment(5 m, 5 m, 5 m, 50000 kg);
LCGrav.placeAtPoint(Equip1, Point(20, 20, 33.5));

// ============================================================
// 阶段 12: 网格与分析
// ============================================================
var Md_panel = MeshDensity();
Md_panel.elementLength = 1.0 m;   // 面板网格 (Panel model)

var Md_fe = MeshDensity();
Md_fe.elementLength = 3.0 m;       // FE 结构网格

Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());
Analysis1.add(LinearAnalysis());
Analysis1.setActive();
SimplifyTopology();
// Analysis1.execute();
```
