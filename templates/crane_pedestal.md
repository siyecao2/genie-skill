# GeniE 起重机基座模型模板 (Crane Pedestal Model Template)

> **严格参照**: SESAM `Help\Tutorials\TutorialsAdvancedModelling\A1_GeniE_Crane_Pedestal\JS\GeniE_Crane_Pedestal_input.js` 和 `A7_GeniE_Semisub_Panel_and_FE\JS\GeniE_Semisub_Panel_and_FE_input.js` (Plate/Curve 建模)

```javascript
// ============================================================
// 阶段 0: GenieRules
// ============================================================
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 2 deg;

// ============================================================
// 阶段 1: 材料定义
// ============================================================
S355 = MaterialLinear(355000000 Pa, 7850 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
S355.setDefault(Material);

// ============================================================
// 阶段 2: 板厚
// ============================================================
Th30 = Thickness(0.030 m);
Th30.name = "30mm Pedestal Shell";
Th20 = Thickness(0.020 m);
Th20.name = "20mm Reinforcements";

// ============================================================
// 阶段 3: 截面定义
// ============================================================
// 基座加强环 (扁钢)
RING_STIFF = BarSection(0.25 m, 0.02 m);
RING_STIFF.name = "RingStiff_250x20";

// 加劲肋 T-bar
T_STIFF = UnsymISection(0.3 m, 0.01 m, 0.01 m, 0.006 m, 1e-6 m, 0.12 m, 0.06 m, 0.015 m);

// ============================================================
// 阶段 4: Guiding Geometry (圆筒 + 锥段)
// ============================================================
var baseZ = 30.0 m;
var baseR = 4.0 m;
var midZ  = 40.0 m;
var midR  = 2.5 m;
var topZ  = 50.0 m;
var topR  = 2.0 m;

// 底面圆
BaseCircle = GuideEllipse(Point(0, 0, baseZ), baseR, baseR, Vector3d(0, 0, 1));

// 中部圆
MidCircle = GuideEllipse(Point(0, 0, midZ), midR, midR, Vector3d(0, 0, 1));

// 顶面圆 (转台)
TopCircle = GuideEllipse(Point(0, 0, topZ), topR, topR, Vector3d(0, 0, 1));

// ============================================================
// 阶段 5: 圆筒曲面 (SkinCurves 蒙皮)
// ============================================================
// 下段: 锥筒 (底面→中部)
LowerCone = SkinCurves(Array(BaseCircle, MidCircle));
LowerCone.name = "LowerCone";
LowerCone.material = S355;
LowerCone.thickness = Th30;

// 上段: 圆柱 (中部→顶部)
UpperCylinder = SkinCurves(Array(MidCircle, TopCircle));
UpperCylinder.name = "UpperCylinder";
UpperCylinder.material = S355;
UpperCylinder.thickness = Th20;

// ============================================================
// 阶段 6: 环形水平板 (CoverCurves 封板)
// ============================================================
// 底部环形法兰
BottomFlange = CoverCurves(BaseCircle);
BottomFlange.name = "BottomFlange";
BottomFlange.thickness = Th30;

// 中部环形隔板
MidDiaphragm = CoverCurves(MidCircle);
MidDiaphragm.name = "MidDiaphragm";
MidDiaphragm.thickness = Th20;

// 顶部转台
TopTable = CoverCurves(TopCircle);
TopTable.name = "TopTable";
TopTable.thickness = Th30;

// ============================================================
// 阶段 7: GuideLine 加强环
// ============================================================
function createRingStiffener(circleCurve, offsetZ) {
    var stiffLine = circleCurve;  // 沿圆周线放置加劲肋
    return stiffLine;
}

// 沿高度分布环形加强肋
var ringLevels = [32 m, 35 m, 38 m, 42 m, 46 m];

// ============================================================
// 阶段 8: 边界条件 (底部固定)
// ============================================================
// 沿底面分布多点固支
var supportPoints = [
    Point(baseR, 0, baseZ),
    Point(-baseR, 0, baseZ),
    Point(0, baseR, baseZ),
    Point(0, -baseR, baseZ)
];

for (var i = 0; i < supportPoints.length; i++) {
    var sp = SupportPoint(supportPoints[i]);
    sp.fixation = SuperFixation;
}

// ============================================================
// 阶段 9: 载荷
// ============================================================
LCGrav = LoadCase(0, 0, -9.81);
LCGrav.name = "Gravity";

// 吊装载荷: 顶部点力 + 弯矩
LCLift = LoadCase(0, 0, 0);
LCLift.name = "LiftingLoad";

// 转台偏心点力 (模拟吊机载荷)
var liftForce = PointForce(Point(0, 0, topZ), Vector3d(0, 0, -500000));
LCLift.add(liftForce);

// 倾覆弯矩 (绕 X 轴)
var overturningMoment = 2.0e+006 N*m;

// ============================================================
// 阶段 10: 载荷组合
// ============================================================
CombULS = LoadCombination();
CombULS.addCase(LCGrav, 1.0);
CombULS.addCase(LCLift, 1.5);
CombULS.name = "ULS_MaxLift";

// ============================================================
// 阶段 11: 网格与分析
// ============================================================
Md = MeshDensity();
Md.elementLength = 0.2 m;  // 曲面板需较小网格

Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());
Analysis1.add(LinearAnalysis());
Analysis1.setActive();
SimplifyTopology();
// Analysis1.execute();
```
