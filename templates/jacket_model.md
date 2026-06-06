# GeniE 导管架模型模板 (Jacket Model Template)

> **严格参照**: SESAM `Help\Tutorials\TutorialsBasicAndCodechecking\B8_GeniE_Piled_Jacket_Analysis\JS\` 和 `Help\GuidingDocuments\Parametric\JS\Frame.js`

```javascript
// ============================================================
// 阶段 0: GenieRules 兼容性与默认设置
// ============================================================
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 2 deg;
GenieRules.Geometry.autoDivideBeamsAtPlateIntersection = true;

// ============================================================
// 阶段 1: 材料定义
// ============================================================
S355 = MaterialLinear(355000000 Pa, 7850 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
S355.description = "S355 Structural Steel";

S275 = MaterialLinear(275000000 Pa, 7850 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
S275.description = "S275 Secondary Steel";

// 将 S355 设为默认材料
S355.setDefault(Material);

// ============================================================
// 阶段 2: 截面定义 (参照真实 Frame.js A2 命名规范)
// ============================================================
// 导管腿: 变截面
LEG_TOP = PipeSection(1.2 m, 0.05 m);
LEG_MID = PipeSection(1.4 m, 0.055 m);
LEG_BOT = PipeSection(1.6 m, 0.06 m);

// 水平撑
BRACE_H = PipeSection(0.762 m, 0.025 m);

// 斜撑 K型
BRACE_K = PipeSection(0.610 m, 0.020 m);

// 锥段: DynamicThickness=true 取大壁厚
CONE_LEG = ConeSection(true);

// ============================================================
// 阶段 3: Guiding Geometry
// ============================================================
function createGuidePlane(name, origin, normal) {
    var gp = GuidePlane(origin, normal);
    gp.name = name;
    return gp;
}

var zLevels = [0.0 m, 8.0 m, 18.0 m, 30.0 m, 45.0 m, 60.0 m];
var guidePlanes = [];
for (var i = 0; i < zLevels.length; i++) {
    guidePlanes.push(GuidePlane(Point(0, 0, zLevels[i]), Vector3d(0, 0, 1)));
}

// ============================================================
// 阶段 4: 四腿导管架建模
// ============================================================
var baseR = 16.0 m;
var topR = 12.0 m;

var legBottoms = [
    Point( baseR,  baseR, 0),
    Point(-baseR,  baseR, 0),
    Point( baseR, -baseR, 0),
    Point(-baseR, -baseR, 0)
];
var legTops = [
    Point( topR,  topR, 60.0 m),
    Point(-topR,  topR, 60.0 m),
    Point( topR, -topR, 60.0 m),
    Point(-topR, -topR, 60.0 m)
];

var legs = [];
for (var i = 0; i < 4; i++) {
    legs[i] = StraightBeam(legBottoms[i], legTops[i]);
    legs[i].name = "LEG_" + (i+1);
    legs[i].section = LEG_BOT;
    // 分段变截面: 底部 LEG_BOT, 中部 CONE_LEG, 顶部 LEG_TOP
    legs[i].divideSegmentAtRelative(1, 0.33);
    legs[i].divideSegmentAtRelative(2, 0.66);
    legs[i].setSegmentSection(1, LEG_BOT);
    legs[i].setSegmentSection(2, CONE_LEG);
    legs[i].setSegmentSection(3, LEG_TOP);
}

// ============================================================
// 阶段 5: 水平撑与斜撑
// ============================================================
for (var zIdx = 1; zIdx < guidePlanes.length; zIdx++) {
    var z = zLevels[zIdx];
    
    // 水平撑 (四条边)
    for (var j = 0; j < 4; j++) {
        var next = (j + 1) % 4;
        var ratio = (zLevels.length - 1 - zIdx) / (zLevels.length - 1);
        var r = baseR + (topR - baseR) * (1 - ratio);
        // 简化: 使用 GuidePlane 坐标生成
    }
    
    // 斜撑 (K型, 仅在中间层)
    if (zIdx > 1 && zIdx < guidePlanes.length - 1) {
        // 相邻层交叉斜撑...
    }
}

// ============================================================
// 阶段 6: 上部组块甲板 (简化)
// ============================================================
var mainDeckZ = 60.0 m;
var deckHalf = 14.0 m;

var p1 = Point(-deckHalf, -deckHalf, mainDeckZ);
var p2 = Point( deckHalf, -deckHalf, mainDeckZ);
var p3 = Point( deckHalf,  deckHalf, mainDeckZ);
var p4 = Point(-deckHalf,  deckHalf, mainDeckZ);

var MainDeck = Plate(p1, p2, p3, p4);
MainDeck.name = "MainDeck";

// 甲板梁
var deckBeams = [];
deckBeams[0] = StraightBeam(p1, p2);
deckBeams[1] = StraightBeam(p3, p4);
deckBeams[2] = StraightBeam(p1, p4);
deckBeams[3] = StraightBeam(p2, p3);

// 设置默认截面给甲板梁
var DECK_BEAM = ISection(0.4 m, 0.2 m, 0.012 m, 0.016 m, 0.027 m);
for (var k = 0; k < deckBeams.length; k++) {
    deckBeams[k].section = DECK_BEAM;
}

// ============================================================
// 阶段 7: 边界条件
// ============================================================
for (var i = 0; i < 4; i++) {
    var sp = SupportPoint(legBottoms[i]);
    sp.fixation = SuperFixation;
}

// ============================================================
// 阶段 8: 载荷
// ============================================================
var LCSelfWeight = LoadCase(0, 0, -9.81);
LCSelfWeight.name = "SelfWeight";

var LCOperation = LoadCase(0, 0, -9.81);
LCOperation.name = "Operation";

var LCExtreme = LoadCase(0, 0, -9.81);
LCExtreme.name = "Extreme";

// ============================================================
// 阶段 9: 分析与网格
// ============================================================
var Md_global = MeshDensity();
Md_global.elementLength = 0.5 m;

var Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());
Analysis1.add(LinearAnalysis());
Analysis1.setActive();
SimplifyTopology();
// Analysis1.execute();  // 取消注释以运行
```
