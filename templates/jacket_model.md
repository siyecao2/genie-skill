# GeniE 导管架模型模板 (Jacket Model Template)

本模板用于创建四腿导管架结构模型，适用于固定式海上平台基础结构。

```javascript
// ============================================================
// 阶段 1: 兼容性设置与项目初始化
// ============================================================
// 设置 SESAM 兼容版本，确保与后续模块的兼容性
gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);
gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);

// 创建新项目
var proj = Project.New();
proj.SetName("Jacket_4Leg_Platform");
proj.SetDescription("四腿导管架平台基础结构分析");

// 设置全局公差
var tol = ToleranceManager();
tol.SetDefaultTolerance(0.001);  // m

// ============================================================
// 阶段 2: 材料定义 (Materials)
// ============================================================
// 主管/主导管腿: S355 高强度结构钢
var matS355 = Material.Create();
matS355.name = "S355";
matS355.type = StructuralSteel;
matS355.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS355.SetYieldStress(355e6);
matS355.SetUltimateStress(490e6);

// 次要构件/撑杆: S275 普通结构钢
var matS275 = Material.Create();
matS275.name = "S275";
matS275.type = StructuralSteel;
matS275.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS275.SetYieldStress(275e6);
matS275.SetUltimateStress(410e6);

// ============================================================
// 阶段 3: 截面定义 (Sections)
// ============================================================
// 导管腿主管截面 - 变截面圆管 (顶部→底部直径增大)
var secLegTop = PipeSection.Create(1.200, 0.050);    // D=1200mm, t=50mm
secLegTop.name = "LEG_TOP";
secLegTop.material = matS355;

var secLegMid = PipeSection.Create(1.400, 0.055);    // D=1400mm, t=55mm
secLegMid.name = "LEG_MID";
secLegMid.material = matS355;

var secLegBot = PipeSection.Create(1.600, 0.060);    // D=1600mm, t=60mm
secLegBot.name = "LEG_BOT";
secLegBot.material = matS355;

// 水平撑杆截面
var secBraceH = PipeSection.Create(0.762, 0.025);    // D=762mm, t=25mm
secBraceH.name = "BRACE_H";
secBraceH.material = matS275;

// 斜撑/K型撑杆截面
var secBraceK = PipeSection.Create(0.610, 0.020);    // D=610mm, t=20mm
secBraceK.name = "BRACE_K";
secBraceK.material = matS275;

// 立管/套管截面
var secConductor = PipeSection.Create(0.508, 0.016);  // D=508mm, t=16mm
secConductor.name = "CONDUCTOR";
secConductor.material = matS275;

// ============================================================
// 阶段 4: 导引几何 - GuidePlane 工作平面
// ============================================================
// 创建底板工作平面 (高程 Z = 0.0m, 泥面)
var gpBase = GuidePlane.Create();
gpBase.name = "GP_BASE";
gpBase.SetOrigin(Point(0, 0, 0));
gpBase.SetNormal(Vector(0, 0, 1));
gpBase.snapmode = true;

// 复制工作平面到各水平层 (单位: m)
var zLevels = [0.0, 8.0, 18.0, 30.0, 45.0, 60.0];  // 泥面 + 各层高程
var guidePlanes = [];
for (var i = 0; i < zLevels.length; i++) {
    var gp = GuidePlane.Copy(gpBase, Point(0, 0, zLevels[i]));
    gp.name = "GP_EL" + zLevels[i].toFixed(1);
    gp.snapmode = true;
    guidePlanes.push(gp);
}

// ============================================================
// 阶段 5: 导管腿建模 (4 Legs)
// ============================================================
// 四腿矩形布置 (角点坐标)
var footPrintR = 12.0;  // 顶部分布半径 (m)
var baseR = 16.0;       // 底部分布半径 (m)
var legPos = [
    {name: "A1", topX:  footPrintR, topY:  footPrintR, botX:  baseR, botY:  baseR},
    {name: "A2", topX: -footPrintR, topY:  footPrintR, botX: -baseR, botY:  baseR},
    {name: "B1", topX:  footPrintR, topY: -footPrintR, botX:  baseR, botY: -baseR},
    {name: "B2", topX: -footPrintR, topY: -footPrintR, botX: -baseR, botY: -baseR}
];

// 创建各腿锥形梁
var legs = [];
for (var i = 0; i < legPos.length; i++) {
    var lp = legPos[i];
    // 底部点 (Z=0)
    var ptBot = Point(lp.botX, lp.botY, 0.0);
    // 顶部点 (Z=60)
    var ptTop = Point(lp.topX, lp.topY, 60.0);
    var legLine = Line(ptBot, ptTop);
    
    var leg = Beam.Create(legLine, secLegBot);
    leg.name = "LEG_" + lp.name;
    // 变截面设置 (多段锥形)
    leg.SetSection(secLegMid, 0.0, 20.0);   // 0-20m: D=1400
    leg.SetSection(secLegBot, 0.0, 8.0);    // 0-8m: D=1600 (覆盖底层)
    leg.SetSection(secLegTop, 45.0, 60.0);  // 45-60m: D=1200
    legs.push(leg);
}

// ============================================================
// 阶段 6: 水平撑杆 (Horizontal Braces)
// ============================================================
// 在各层之间布置水平撑杆
function createHorizontalBraces(levelZ, sec) {
    var braces = [];
    var r = footPrintR - (footPrintR - baseR) * (levelZ / 60.0);  // 线性插值半径
    var offsets = [
        [[ r,  r], [-r,  r]],
        [[-r,  r], [-r, -r]],
        [[-r, -r], [ r, -r]],
        [[ r, -r], [ r,  r]]
    ];
    for (var i = 0; i < offsets.length; i++) {
        var p1 = Point(offsets[i][0][0], offsets[i][0][1], levelZ);
        var p2 = Point(offsets[i][1][0], offsets[i][1][1], levelZ);
        var b = Beam.Create(Line(p1, p2), sec);
        b.name = "BRACE_H_EL" + levelZ.toFixed(1) + "_" + i;
        braces.push(b);
    }
    return braces;
}

// 在各水平层创建撑杆
for (var j = 1; j < zLevels.length; j++) {
    createHorizontalBraces(zLevels[j], secBraceH);
}

// ============================================================
// 阶段 7: 斜撑/K型撑杆 (Diagonal/K-Braces)
// ============================================================
// 在各层间布置K型斜撑
function createKBraces(zBase, zTop, sec) {
    // A侧面: A1-A2 之间的K撑
    var rBot = footPrintR - (footPrintR - baseR) * (zBase / 60.0);
    var rTop = footPrintR - (footPrintR - baseR) * (zTop / 60.0);
    
    // Frame A (x=+r 面)
    var b1 = Beam.Create(Line(Point( rBot,  rBot, zBase), Point( rTop, -rTop, zTop)), sec);
    b1.name = "BRACE_K_A1_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
    var b2 = Beam.Create(Line(Point( rBot, -rBot, zBase), Point( rTop,  rTop, zTop)), sec);
    b2.name = "BRACE_K_A2_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
    
    // Frame B (x=-r 面)
    var b3 = Beam.Create(Line(Point(-rBot,  rBot, zBase), Point(-rTop, -rTop, zTop)), sec);
    b3.name = "BRACE_K_B1_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
    var b4 = Beam.Create(Line(Point(-rBot, -rBot, zBase), Point(-rTop,  rTop, zTop)), sec);
    b4.name = "BRACE_K_B2_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
    
    // Frame 1 (y=+r 面)
    var b5 = Beam.Create(Line(Point( rBot,  rBot, zBase), Point(-rTop,  rTop, zTop)), sec);
    b5.name = "BRACE_K_C1_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
    var b6 = Beam.Create(Line(Point(-rBot,  rBot, zBase), Point( rTop,  rTop, zTop)), sec);
    b6.name = "BRACE_K_C2_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
    
    // Frame 2 (y=-r 面)
    var b7 = Beam.Create(Line(Point( rBot, -rBot, zBase), Point(-rTop, -rTop, zTop)), sec);
    b7.name = "BRACE_K_D1_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
    var b8 = Beam.Create(Line(Point(-rBot, -rBot, zBase), Point( rTop, -rTop, zTop)), sec);
    b8.name = "BRACE_K_D2_" + zBase.toFixed(0) + "_" + zTop.toFixed(0);
}

// 为每层间隔创建K撑
for (var k = 0; k < zLevels.length - 1; k++) {
    createKBraces(zLevels[k], zLevels[k + 1], secBraceK);
}

// ============================================================
// 阶段 8: 套管导向环 (Conductor Guides)
// ============================================================
// 在腿之间布置套管导向结构
var condGuidePositions = [
    [ 3.0,  3.0], [ 3.0, -3.0], [-3.0,  3.0], [-3.0, -3.0]
];

for (var l = 1; l < zLevels.length; l++) {
    for (var m = 0; m < condGuidePositions.length; m++) {
        var cx = condGuidePositions[m][0];
        var cy = condGuidePositions[m][1];
        var cond = Beam.Create(
            Line(Point(cx, cy, 0.0), Point(cx, cy, zLevels[l])),
            secConductor
        );
        cond.name = "COND_" + m + "_EL" + zLevels[l].toFixed(1);
    }
}

// ============================================================
// 阶段 9: 防沉板/底部支撑 (Mudmat / Bottom Support)
// ============================================================
// 底部泥垫板
var mmSize = 3.0;  // 防沉板尺寸
var mudmatCenters = legPos.map(function(lp) {
    return Point(lp.botX, lp.botY, -0.5);
});

for (var n = 0; n < mudmatCenters.length; n++) {
    var mc = mudmatCenters[n];
    var mudmat = Plate.CreateRectangular(mc, 0.0, mmSize, mmSize);
    mudmat.name = "MUDMAT_" + legPos[n].name;
    mudmat.thickness = 0.030;  // 30mm 板厚
    mudmat.material = matS275;
}

// ============================================================
// 阶段 10: 边界条件 (Boundary Conditions)
// ============================================================
// 桩基固接在泥面 (pinned at seabed Z=0)
var bcSeabedPinned = BC.Create();
bcSeabedPinned.name = "BC_Seabed_Pinned";
bcSeabedPinned.SetDisplacement(0, 0, 0);      // UX=UY=UZ=0
bcSeabedPinned.SetRotation(999, 999, 999);    // RX=RY=RZ=free (铰接)

// 在泥面处施加边界条件
for (var p = 0; p < legs.length; p++) {
    legs[p].SetBC(bcSeabedPinned, 0.0);  // 在梁的 0.0 参数位置
}

// ============================================================
// 阶段 11: 载荷定义 (Load Cases)
// ============================================================
// 载荷工况 1: 自重 (Gravity)
var lc1 = LoadCase.Create();
lc1.name = "Gravity";
lc1.type = Permanent;
lc1.number = 101;
lc1.ActivateSelfWeight(0, 0, -1, 1.0);  // Z轴向下, 1.0g

// 载荷工况 2: 环境载荷 (Environmental - 占位)
var lc2 = LoadCase.Create();
lc2.name = "Env_Wave_North";
lc2.type = Environmental;
lc2.number = 201;
// 实际波浪载荷将在后续通过 HydroD/Wadam 导入或手动施加
printed("环境载荷工况已创建，请通过外部模块或脚本施加波浪力。");

// 载荷工况 3: 风载荷 (Wind)
var lc3 = LoadCase.Create();
lc3.name = "Wind_North";
lc3.type = Environmental;
lc3.number = 301;
// 风载荷由专门的 Wind 模板设置
printed("风载荷工况已创建，请使用 wind_loads 模板施加风载荷。");

// ============================================================
// 阶段 12: 分析设置 (Analysis Setup)
// ============================================================
// 简化拓扑 (合并重合节点, 清理微小边)
SimplifyTopology();

// 创建活动列表
var act1 = Activity.Create();
act1.name = "Activity_Jacket_Analysis";

// 网格设置
var meshCtrl = MeshControl.Default();
meshCtrl.SetGlobalElementSize(0.5);  // 全局单元尺寸 0.5m
meshCtrl.SetLocalRefinementOnJoints(0.05, 0.30);  // 管节点局部细化

// 生成网格
var meshSet = MeshSet.Create();
meshSet.name = "Mesh_Jacket";
meshSet.AddAllBodies();
meshSet.SetMeshControl(meshCtrl);
meshSet.GenerateMesh();

// 求解器设置
var solver = LinearStaticSolver();
solver.name = "Solve_Jacket_Gravity";
solver.AddLoadCase(lc1);

// 执行分析
var analysis = Analysis.Create();
analysis.name = "Analysis_Jacket";
analysis.AddSolver(solver);
analysis.SetActivity(act1);

// 运行求解
// analysis.Solve();  // 取消注释以执行分析

// ============================================================
// 阶段 13: 结果输出
// ============================================================
print("===== 导管架模型创建完成 =====");
print("模型名称: " + proj.name);
print("导管腿数: 4");
print("水平层数: " + zLevels.length);
print("材料: S355 (主管), S275 (次构件)");
print("边界: 泥面铰接 (Z=0)");
print("载荷工况: " + analysis.GetSolver(0).GetLoadCaseCount());
print("===============================");
```

## 使用说明

1. 将脚本复制到 GeniE 的 JavaScript 编辑器中运行
2. 根据实际项目调整几何参数（高程、半径、截面尺寸）
3. 补充完整的环境载荷数据后取消 `analysis.Solve()` 注释
4. 运行后在 Results 视图查看位移和应力结果
