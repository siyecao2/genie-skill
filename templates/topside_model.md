# GeniE 上部组块模型模板 (Topside Model Template)

> **严格参照**: SESAM `Help\Tutorials\TutorialsBasicAndCodechecking\B5_GeniE_Topside\JS\Topside_input.js` (301行)

```javascript
// ============================================================
// 阶段 0: GenieRules 兼容性
// ============================================================
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.BeamCreation.DefaultCurveOffset = ReparameterizedBeamCurveOffset();

// ============================================================
// 阶段 1: 材料定义 (参照 B5 Topside_input.js)
// ============================================================
S355 = MaterialLinear(355000000 Pa, 7850 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
S275 = MaterialLinear(275000000 Pa, 7850 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
S355.setDefault(Material);

// ============================================================
// 阶段 2: 板厚定义 (参照 B5 实际代码)
// ============================================================
Th25 = Thickness(0.025 m);  // 25mm 甲板
Th20 = Thickness(0.020 m);  // 20mm 下层甲板
Th12 = Thickness(0.012 m);  // 12mm 舱壁

// ============================================================
// 阶段 3: 截面定义
// ============================================================
// H 型钢 (ISection: height, flangeWidth, webThk, flangeThk, radius)
HE400A = ISection(0.39 m, 0.3 m, 0.011 m, 0.019 m, 0.027 m);
HE400A.name = "HE400A";

HE600B = ISection(0.6 m, 0.3 m, 0.011 m, 0.019 m, 0.027 m);
HE600B.name = "HE600B GIRDER";

// 管柱
COLUMN_610 = PipeSection(0.610 m, 0.025 m);
COLUMN_610.name = "COLUMN_610x25";

// T型加劲肋 (UnsymISection)
T_STIFF = UnsymISection(0.3 m, 0.01 m, 0.01 m, 0.006 m, 1e-6 m, 0.15 m, 0.06 m, 0.015 m);
T_STIFF.name = "T-Stiffener 300x150";

// 扁钢加劲
FB_STIFF = BarSection(0.15 m, 0.015 m);
FB_STIFF.name = "FB 150x15";

// ============================================================
// 阶段 4: GuidePlane 工作平面
// ============================================================
var zMainDeck  = 60.0 m;
var zLowerDeck = 52.0 m;
var zBottom    = 45.0 m;

GPMain  = GuidePlane(Point(0, 0, zMainDeck),  Vector3d(0, 0, 1));
GPLower = GuidePlane(Point(0, 0, zLowerDeck), Vector3d(0, 0, 1));
GPBottom = GuidePlane(Point(0, 0, zBottom),   Vector3d(0, 0, 1));

// ============================================================
// 阶段 5: 甲板板建模 (Plate: 4 个角点)
// ============================================================
var deckHalfL = 15.0 m;
var deckHalfW = 10.0 m;

var p1 = Point(-deckHalfL, -deckHalfW, zMainDeck);
var p2 = Point( deckHalfL, -deckHalfW, zMainDeck);
var p3 = Point( deckHalfL,  deckHalfW, zMainDeck);
var p4 = Point(-deckHalfL,  deckHalfW, zMainDeck);

MainDeck = Plate(p1, p2, p3, p4);
MainDeck.name = "MainDeck";
MainDeck.material = S355;
MainDeck.thickness = Th25;

var p1L = Point(-deckHalfL, -deckHalfW, zLowerDeck);
var p2L = Point( deckHalfL, -deckHalfW, zLowerDeck);
var p3L = Point( deckHalfL,  deckHalfW, zLowerDeck);
var p4L = Point(-deckHalfL,  deckHalfW, zLowerDeck);

LowerDeck = Plate(p1L, p2L, p3L, p4L);
LowerDeck.name = "LowerDeck";
LowerDeck.material = S355;
LowerDeck.thickness = Th20;

// ============================================================
// 阶段 6: 立柱 (参照 B5: 柱底部 Cone + 多段截面)
// ============================================================
var colPositions = [
    Point(-10 m, -6 m, zBottom),
    Point( 10 m, -6 m, zBottom),
    Point(-10 m,  6 m, zBottom),
    Point( 10 m,  6 m, zBottom)
];

var colTops = [
    Point(-10 m, -6 m, zMainDeck),
    Point( 10 m, -6 m, zMainDeck),
    Point(-10 m,  6 m, zMainDeck),
    Point( 10 m,  6 m, zMainDeck)
];

for (var i = 0; i < 4; i++) {
    var col = StraightBeam(colPositions[i], colTops[i]);
    col.name = "COL_" + (i+1);
    col.section = COLUMN_610;
}

// ============================================================
// 阶段 7: 甲板梁 (沿甲板边缘)
// ============================================================
// 主甲板边缘梁
var edge1 = StraightBeam(p1, p2);
edge1.name = "MainEdge_X1";
edge1.section = HE600B;
edge1.CurveOffset = AlignedCurveOffset(frFlushTop, 0 m);

var edge2 = StraightBeam(p2, p3);
edge2.name = "MainEdge_Y1";
edge2.section = HE600B;
edge2.CurveOffset = AlignedCurveOffset(frFlushTop, 0 m);

var edge3 = StraightBeam(p3, p4);
edge3.name = "MainEdge_X2";
edge3.section = HE600B;
edge3.CurveOffset = AlignedCurveOffset(frFlushTop, 0 m);

var edge4 = StraightBeam(p4, p1);
edge4.name = "MainEdge_Y2";
edge4.section = HE600B;
edge4.CurveOffset = AlignedCurveOffset(frFlushTop, 0 m);

// ============================================================
// 阶段 8: 边界条件
// ============================================================
for (var i = 0; i < 4; i++) {
    var sp = SupportPoint(colPositions[i]);
    sp.fixation = SuperFixation;
}

// ============================================================
// 阶段 9: 载荷
// ============================================================
LCGrav  = LoadCase(0, 0, -9.81);
LCGrav.name = "LCGrav";

LCOper  = LoadCase(0, 0, -9.81);
LCOper.name = "Operation";

// 设备载荷 (参照 B5 Equipment 模式)
Pump = PrismEquipment(4 m, 4 m, 5 m, 75000 kg);
Pump.name = "Pump_75t";

GenSet = PrismEquipment(10 m, 4 m, 5 m, 100000 kg);
GenSet.name = "Generator_100t";

// 放置设备至工况 (LCOper 设为当前工况前执行)
LCOper.placeAtPoint(Pump, Point(18 m, 0 m, zMainDeck + 0.5 m));
LCOper.placeAtPoint(GenSet, Point(14 m, 3.75 m, zLowerDeck + 0.5 m));

// 设备作为线载荷 (默认) 或偏心质量
// LCOper.equipmentAsMass();  // 如需设备作为质量

// ============================================================
// 阶段 10: 载荷组合
// ============================================================
Comb1 = LoadCombination();
Comb1.addCase(LCGrav, 1.0);
Comb1.addCase(LCOper, 1.1);
Comb1.name = "ULS_1";

// ============================================================
// 阶段 11: 分析与网格
// ============================================================
Md1 = MeshDensity();
Md1.elementLength = 0.3 m;

Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());
Analysis1.add(LinearAnalysis());
Analysis1.setActive();
SimplifyTopology();
// Analysis1.execute();
```
