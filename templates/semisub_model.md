# GeniE 半潜式平台模型模板 (Semi-Submersible Model Template)

本模板用于创建半潜式平台的浮筒-立柱-支撑结构模型，包括对称边界和曲面壳体建模。

```javascript
// ============================================================
// 阶段 1: 兼容性设置与项目初始化
// ============================================================
gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);
gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);

var proj = Project.New();
proj.SetName("SemiSub_Model");
proj.SetDescription("半潜式平台结构分析 - 1/4对称模型");

var tol = ToleranceManager();
tol.SetDefaultTolerance(0.001);

// ============================================================
// 阶段 2: 材料定义
// ============================================================
// 壳体/板材: S355
var matS355 = Material.Create();
matS355.name = "S355";
matS355.type = StructuralSteel;
matS355.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS355.SetYieldStress(355e6);

// 高强度区域: S420
var matS420 = Material.Create();
matS420.name = "S420";
matS420.type = StructuralSteel;
matS420.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS420.SetYieldStress(420e6);

// ============================================================
// 阶段 3: 截面定义
// ============================================================
// 立柱截面 - 圆柱
var secColumn = PipeSection.Create(15.0, 0.040);  // D=15m, t=40mm (大直径立柱)
secColumn.name = "COLUMN_MAIN";
secColumn.material = matS355;

// 横撑截面
var secBrace = PipeSection.Create(3.0, 0.025);    // D=3m, t=25mm
secBrace.name = "BRACE_CROSS";
secBrace.material = matS355;

// ============================================================
// 阶段 4: 几何参数
// ============================================================
var pontoonLength = 80.0;     // 浮筒长度 (m)
var pontoonWidth  = 15.0;     // 浮筒宽度 (m)
var pontoonHeight = 8.0;      // 浮筒高度 (m)
var pontoonZBot   = -30.0;    // 浮筒底部高程 (m)
var pontoonZTop   = pontoonZBot + pontoonHeight;

var columnDia    = 15.0;      // 立柱直径 (m)
var columnHeight = 25.0;      // 立柱高度 (m)
var columnZBot   = pontoonZTop;
var columnZTop   = columnZBot + columnHeight;

var deckBoxZ     = columnZTop;  // 甲板盒底部高程

// 四立柱布局 (半潜平台四个角)
var colSpacingX = 50.0;  // 立柱X方向间距 (中心到中心)
var colSpacingY = 40.0;  // 立柱Y方向间距

// 1/4 对称模型: 仅建模 +X, +Y 象限
var quarterColumnCenters = [
    {x: colSpacingX/2, y: colSpacingY/2, index: "NE"}
];

// ============================================================
// 阶段 5: GuidePlane 工作平面
// ============================================================
// 底部工作平面
var gpBot = GuidePlane.Create();
gpBot.name = "GP_Bottom";
gpBot.SetOrigin(Point(0, 0, pontoonZBot));
gpBot.SetNormal(Vector(0, 0, 1));
gpBot.snapmode = true;

// 中部工作平面 (浮筒顶/立柱底)
var gpMid = GuidePlane.Copy(gpBot, Point(0, 0, pontoonZTop));
gpMid.name = "GP_Mid";
gpMid.snapmode = true;

// 顶部工作平面
var gpTop = GuidePlane.Copy(gpBot, Point(0, 0, deckBoxZ));
gpTop.name = "GP_Top";
gpTop.snapmode = true;

// 对称面工作平面 X=0 (YZ面)
var gpSymX = GuidePlane.Create();
gpSymX.name = "GP_Symmetry_X";
gpSymX.SetOrigin(Point(0, 0, 0));
gpSymX.SetNormal(Vector(1, 0, 0));
gpSymX.snapmode = true;

// 对称面工作平面 Y=0 (XZ面)
var gpSymY = GuidePlane.Create();
gpSymY.name = "GP_Symmetry_Y";
gpSymY.SetOrigin(Point(0, 0, 0));
gpSymY.SetNormal(Vector(0, 1, 0));
gpSymY.snapmode = true;

// ============================================================
// 阶段 6: 浮筒建模 (Pontoon as Box Beam/Plate)
// ============================================================
// 浮筒截面点 (箱型截面轮廓)
var px = colSpacingX/2;
var py = colSpacingY/2;
var pw2 = pontoonWidth / 2;
var ph2 = pontoonHeight / 2;

// 浮筒中段: 从 X=0 到 X=colSpacingX/2 的箱型梁
var pontoonProfile = [
    Point(px, py - pw2, pontoonZBot),
    Point(px, py + pw2, pontoonZBot),
    Point(px, py + pw2, pontoonZTop),
    Point(px, py - pw2, pontoonZTop)
];

// 创建浮筒壳体曲面 (SkinCurves)
var pontoonCurveBot = Line(Point(0, py - pw2, pontoonZBot), Point(px, py - pw2, pontoonZBot));
var pontoonCurveTop = Line(Point(0, py + pw2, pontoonZBot), Point(px, py + pw2, pontoonZBot));

// 浮筒底板 - 板建模
var pontoonBottomPlate = Plate.CreateByPoints([
    Point(0, py - pw2, pontoonZBot),
    Point(px, py - pw2, pontoonZBot),
    Point(px, py + pw2, pontoonZBot),
    Point(0, py + pw2, pontoonZBot)
]);
pontoonBottomPlate.name = "PLT_PontoonBottom";
pontoonBottomPlate.thickness = 0.020;
pontoonBottomPlate.material = matS355;

// 浮筒侧板 (外板)
var pontoonSideOuter = Plate.CreateByPoints([
    Point(0, py + pw2, pontoonZBot),
    Point(px, py + pw2, pontoonZBot),
    Point(px, py + pw2, pontoonZTop),
    Point(0, py + pw2, pontoonZTop)
]);
pontoonSideOuter.name = "PLT_PontoonSideOuter";
pontoonSideOuter.thickness = 0.018;
pontoonSideOuter.material = matS355;

// 浮筒侧板 (内板)
var pontoonSideInner = Plate.CreateByPoints([
    Point(0, py - pw2, pontoonZBot),
    Point(px, py - pw2, pontoonZBot),
    Point(px, py - pw2, pontoonZTop),
    Point(0, py - pw2, pontoonZTop)
]);
pontoonSideInner.name = "PLT_PontoonSideInner";
pontoonSideInner.thickness = 0.018;
pontoonSideInner.material = matS355;

// 浮筒甲板 (顶部)
var pontoonDeckPlate = Plate.CreateByPoints([
    Point(0, py - pw2, pontoonZTop),
    Point(px, py - pw2, pontoonZTop),
    Point(px, py + pw2, pontoonZTop),
    Point(0, py + pw2, pontoonZTop)
]);
pontoonDeckPlate.name = "PLT_PontoonTop";
pontoonDeckPlate.thickness = 0.025;
pontoonDeckPlate.material = matS420;  // 高应力区

// ============================================================
// 阶段 7: 立柱建模 (Cylindrical Columns)
// ============================================================
// 使用 SkinCurves 创建圆柱曲面
var colRadius = columnDia / 2;
var colCenter = quarterColumnCenters[0];

// 立柱中心线
var colAxis = Line(
    Point(colCenter.x, colCenter.y, pontoonZTop),
    Point(colCenter.x, colCenter.y, deckBoxZ)
);

// 立柱梁 (中心线梁)
var colBeam = Beam.Create(colAxis, secColumn);
colBeam.name = "COLUMN_" + colCenter.index;

// 立柱壳板 (环形曲面, SkinCurves)
var colCircumference = [];
var nSegments = 24;  // 24 段近似圆
for (var i = 0; i < nSegments; i++) {
    var angle = (i / nSegments) * 2 * Math.PI;
    colCircumference.push({
        x: colCenter.x + colRadius * Math.cos(angle),
        y: colCenter.y + colRadius * Math.sin(angle)
    });
}

// 创建立柱圆柱壳体 (通过导引线)
var colGuideBottom = [];
var colGuideTop = [];
for (var j = 0; j < nSegments; j++) {
    var c = colCircumference[j];
    var cn = colCircumference[(j + 1) % nSegments];
    
    // 底部导引线段
    colGuideBottom.push(Line(
        Point(c.x, c.y, pontoonZTop),
        Point(cn.x, cn.y, pontoonZTop)
    ));
    // 顶部导引线段
    colGuideTop.push(Line(
        Point(c.x, c.y, deckBoxZ),
        Point(cn.x, cn.y, deckBoxZ)
    ));
}

// 通过导引曲线创建蒙皮曲面
var colSkin = SkinCurves.Create(colGuideBottom, colGuideTop);
colSkin.name = "SKIN_Column_" + colCenter.index;
colSkin.thickness = 0.035;
colSkin.material = matS355;

// ============================================================
// 阶段 8: 横撑 (Cross Bracing between Columns)
// ============================================================
// 在 NE 立柱与对称面边界之间设置横撑
var braceZLevels = [pontoonZTop + 5.0, pontoonZTop + 15.0];

for (var b = 0; b < braceZLevels.length; b++) {
    var bz = braceZLevels[b];
    
    // X方向撑杆 - 从X=0(对称面)到立柱中心
    var braceX = Beam.Create(
        Line(Point(0, colCenter.y, bz), Point(colCenter.x, colCenter.y, bz)),
        secBrace
    );
    braceX.name = "BRACE_X_Z" + bz.toFixed(1);
    
    // Y方向撑杆 - 从Y=0(对称面)到立柱中心
    var braceY = Beam.Create(
        Line(Point(colCenter.x, 0, bz), Point(colCenter.x, colCenter.y, bz)),
        secBrace
    );
    braceY.name = "BRACE_Y_Z" + bz.toFixed(1);
}

// ============================================================
// 阶段 9: 对称边界条件 (Symmetry Boundary Conditions)
// ============================================================
// X=0 平面 (YZ面) - 反对称: UX=0, RY=RZ=0
var bcSymX = BC.Create();
bcSymX.name = "BC_Symmetry_X";
bcSymX.SetDisplacement(0, 999, 999);    // UX=0, UY/UZ free
bcSymX.SetRotation(999, 0, 0);          // RY=RZ=0, RX free

// Y=0 平面 (XZ面) - 反对称: UY=0, RX=RZ=0
var bcSymY = BC.Create();
bcSymY.name = "BC_Symmetry_Y";
bcSymY.SetDisplacement(999, 0, 999);    // UY=0, UX/UZ free
bcSymY.SetRotation(0, 999, 0);          // RX=RZ=0, RY free

// 施加对称边界 - 选择在 X=0 和 Y=0 平面上的所有节点
// (GeniE 中通过选取对称面上的曲线/面来施加)
printed("对称边界条件已定义: X=0(YZ面) 和 Y=0(XZ面)");
printed("请手动选取对称面上的边线施加 BC_Symmetry_X 和 BC_Symmetry_Y");

// ============================================================
// 阶段 10: 载荷定义
// ============================================================
// LC 101: 自重
var lcGravity = LoadCase.Create();
lcGravity.name = "Gravity";
lcGravity.type = Permanent;
lcGravity.number = 101;
lcGravity.ActivateSelfWeight(0, 0, -1, 1.0);

// LC 201: 静水压力 (外部)
var lcHydro = LoadCase.Create();
lcHydro.name = "Hydrostatic_Pressure";
lcHydro.type = Environmental;
lcHydro.number = 201;
// 静水压力作用于壳体外部
var waterline = 0.0;  // 水线面 Z=0
lcHydro.ActivateHydrostaticPressure(waterline, 1025);  // 海水密度 1025 kg/m³

// LC 301: 甲板载荷
var lcDeck = LoadCase.Create();
lcDeck.name = "Deck_Load";
lcDeck.type = Live;
lcDeck.number = 301;

// ============================================================
// 阶段 11: 分析设置
// ============================================================
SimplifyTopology();

var act = Activity.Create();
act.name = "Activity_SemiSub";

var meshCtrl = MeshControl.Default();
meshCtrl.SetGlobalElementSize(0.8);
meshCtrl.SetPlateQuadMesh(true);

var meshSet = MeshSet.Create();
meshSet.name = "Mesh_SemiSub";
meshSet.AddAllBodies();
meshSet.SetMeshControl(meshCtrl);
meshSet.GenerateMesh();

var solver = LinearStaticSolver();
solver.name = "Solve_SemiSub";
solver.AddLoadCase(lcGravity);
solver.AddLoadCase(lcHydro);

var analysis = Analysis.Create();
analysis.name = "Analysis_SemiSub";
analysis.AddSolver(solver);
analysis.SetActivity(act);

// analysis.Solve();

print("===== 半潜式平台模型创建完成 =====");
print("模型类型: 1/4对称半潜平台");
print("浮筒: " + pontoonLength + "m x " + pontoonWidth + "m x " + pontoonHeight + "m");
print("立柱直径: " + columnDia.toFixed(1) + "m, 高度: " + columnHeight.toFixed(1) + "m");
print("对称边界: X=0 面 + Y=0 面");
print("载荷: 自重 + 静水压力 + 甲板载荷");
print("===============================");
```

## 使用说明

1. 本模板采用 1/4 对称模型以减少计算量
2. `SkinCurves` 用于创建圆柱壳体曲面，适合水动力分析
3. 对称边界条件需要根据对称类型选择正确的自由度约束
4. 静水压力作用于壳体外部，水深基准为 Z=0 (水线面)
5. 完整模型需通过镜像功能生成其余三个象限
