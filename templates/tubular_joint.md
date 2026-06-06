# GeniE 管节点模型模板 (Tubular Joint Model Template)

本模板用于创建管节点（弦杆-撑杆连接）的精细化有限元分析模型，支持梁单元和壳单元两种建模方式。

```javascript
// ============================================================
// 阶段 1: 兼容性设置与项目初始化
// ============================================================
gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);
gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);

var proj = Project.New();
proj.SetName("Tubular_Joint_Analysis");
proj.SetDescription("管节点承载力分析 - Y/T/K型节点");

var tol = ToleranceManager();
tol.SetDefaultTolerance(0.0001);  // 极高精度

// ============================================================
// 阶段 2: 材料定义
// ============================================================
// 主管材料: S355
var matS355 = Material.Create();
matS355.name = "S355";
matS355.type = StructuralSteel;
matS355.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS355.SetYieldStress(355e6);
matS355.SetUltimateStress(490e6);

// 撑杆材料: S355 (可独立定义)
var matBrace = Material.Create();
matBrace.name = "S355_Brace";
matBrace.type = StructuralSteel;
matBrace.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matBrace.SetYieldStress(355e6);
matBrace.SetUltimateStress(490e6);

// ============================================================
// 阶段 3: 截面定义
// ============================================================
// 弦杆 (Chord) - 主管
var secChord = PipeSection.Create(1.200, 0.050);   // D=1200mm, t=50mm
secChord.name = "CHORD_SEC";
secChord.material = matS355;

// 撑杆1 (Brace 1) - 垂直撑杆
var secBrace1 = PipeSection.Create(0.762, 0.030);  // D=762mm, t=30mm
secBrace1.name = "BRACE1_SEC";
secBrace1.material = matBrace;

// 撑杆2 (Brace 2) - 斜撑杆
var secBrace2 = PipeSection.Create(0.610, 0.025);  // D=610mm, t=25mm
secBrace2.name = "BRACE2_SEC";
secBrace2.material = matBrace;

// ============================================================
// 阶段 4: 几何参数
// ============================================================
// 弦杆: 水平管, 长度 8.0m
var chordLength = 8.0;
var chordRadius = 0.600;       // D/2

// 撑杆与弦杆的直径比和壁厚比
var beta1  = 0.762 / 1.200;    // d/D = 0.635
var beta2  = 0.610 / 1.200;    // d/D = 0.508
var tau1   = 0.030 / 0.050;    // t/T = 0.60
var tau2   = 0.025 / 0.050;    // t/T = 0.50

// 撑杆角度
var brace1Angle = 90.0 * Math.PI / 180;  // 垂直 (90°)
var brace2Angle = 45.0 * Math.PI / 180;  // 45° 斜撑

// 交点位置 (弦杆中点)
var jointX = 0.0;
var jointY = 0.0;
var jointZ = 0.0;

// ============================================================
// 阶段 5: GuidePlane 工作平面
// ============================================================
// 水平工作平面 (弦杆平面)
var gpHoriz = GuidePlane.Create();
gpHoriz.name = "GP_Horizontal";
gpHoriz.SetOrigin(Point(0, 0, 0));
gpHoriz.SetNormal(Vector(0, 0, 1));
gpHoriz.snapmode = true;

// 垂直工作平面 (XZ面, 用于撑杆1)
var gpVert = GuidePlane.Create();
gpVert.name = "GP_Vertical_XZ";
gpVert.SetOrigin(Point(0, 0, 0));
gpVert.SetNormal(Vector(0, 1, 0));
gpVert.snapmode = true;

// ============================================================
// 阶段 6: 弦杆建模 (Chord Beam)
// ============================================================
// 弦杆: 沿X轴
var chordLine = Line(
    Point(-chordLength/2, 0, 0),
    Point( chordLength/2, 0, 0)
);

var chordBeam = Beam.Create(chordLine, secChord);
chordBeam.name = "CHORD";

// 弦杆边界条件: 两端简支
var bcChordEnd = BC.Create();
bcChordEnd.name = "BC_Chord_Pinned";
bcChordEnd.SetDisplacement(0, 0, 0);      // 平移全约束
bcChordEnd.SetRotation(999, 999, 999);    // 转动自由

chordBeam.SetBC(bcChordEnd, 0.0);   // 左端
chordBeam.SetBC(bcChordEnd, 1.0);   // 右端

// ============================================================
// 阶段 7: 撑杆建模 (Brace Beams)
// ============================================================
// 撑杆1: 垂直 (沿 Z 轴正向)
var brace1Length = 4.0;  // 取弦杆直径的 ~4倍
var brace1Line = Line(
    Point(0, 0, jointZ),
    Point(0, 0, jointZ + brace1Length)
);
var brace1Beam = Beam.Create(brace1Line, secBrace1);
brace1Beam.name = "BRACE_1_Vertical";

// 撑杆2: 45° 斜撑 (在 XZ 平面)
var brace2Length = 5.0;
var brace2dx = brace2Length * Math.sin(brace2Angle);
var brace2dz = brace2Length * Math.cos(brace2Angle);
var brace2Line = Line(
    Point(jointX, jointY, jointZ),
    Point(jointX + brace2dx, jointY, jointZ + brace2dz)
);
var brace2Beam = Beam.Create(brace2Line, secBrace2);
brace2Beam.name = "BRACE_2_45deg";

// ============================================================
// 阶段 8: 节点连接处理 (Joint Connection)
// ============================================================
// 设置梁单元之间的节点连接
// 撑杆端部连接到弦杆 (GeniE 自动在交点创建节点)
SimplifyTopology();  // 合并重合节点, 确保撑杆端与弦杆交于同一点

// 撑杆端部荷载施加参考点
var refPointBrace1 = Point(0, 0, jointZ + brace1Length);
var refPointBrace2 = Point(jointX + brace2dx, jointY, jointZ + brace2dz);

// ============================================================
// 阶段 9: 壳单元转换选项 (Shell Conversion)
// ============================================================
// 将管节点区域转为壳单元进行精细分析
// 方法: 先创建梁模型, 再通过 ShellFromBeam 转换

var useShellModel = true;  // 开关: true=壳单元, false=梁单元

if (useShellModel) {
    // 定义壳转换区域参数
    var shellRange = 1.5 * chordRadius;  // 壳转换范围: 1.5倍弦杆半径
    
    // 将弦杆管节点区域转换为壳
    var chordShellRegion = [jointX - shellRange, jointX + shellRange];
    var brace1ShellRegion = [jointZ, jointZ + shellRange];
    var brace2ShellRegion = [0.0, shellRange / Math.sin(brace2Angle)];
    
    // 使用 ShellFromBeam 转换 (GeniE 支持此功能)
    // 创建壳结构
    var chordShell = ShellFromBeam.Create(chordBeam, chordShellRegion[0], chordShellRegion[1]);
    chordShell.name = "SHELL_Chord";
    chordShell.thickness = 0.050;
    chordShell.material = matS355;
    
    var brace1Shell = ShellFromBeam.Create(brace1Beam, brace1ShellRegion[0], brace1ShellRegion[1]);
    brace1Shell.name = "SHELL_Brace1";
    brace1Shell.thickness = 0.030;
    brace1Shell.material = matBrace;
    
    var brace2Shell = ShellFromBeam.Create(brace2Beam, brace2ShellRegion[0], brace2ShellRegion[1]);
    brace2Shell.name = "SHELL_Brace2";
    brace2Shell.thickness = 0.025;
    brace2Shell.material = matBrace;
    
    // 壳单元交集处理 (管节点相贯线)
    // GeniE 自动处理壳体相交, 如需手动控制可使用以下方式:
    // IntersectionManager.ResolveAllIntersections();
    
    printed("壳单元转换完成: 弦杆 + 2撑杆已转为壳模型");
}

// ============================================================
// 阶段 10: 局部网格细化 (Local Joint Mesh Refinement)
// ============================================================
var meshCtrl = MeshControl.Default();

// 全局网格
meshCtrl.SetGlobalElementSize(0.15);  // 0.15m (梁单元足够)

if (useShellModel) {
    // 壳模型全局网格
    meshCtrl.SetGlobalElementSize(0.05);  // 0.05m (壳单元需更细)
    meshCtrl.SetPlateQuadMesh(true);      // 四边形网格
    
    // 管节点相交线附近局部细化
    meshCtrl.SetLocalRefinementOnCurve(
        [jointX - shellRange, jointX + shellRange],  // 弦杆相贯线区域
        0.015,  // 局部单元尺寸 15mm
        0.10    // 过渡区域半径
    );
    
    // 梁-壳过渡区域处理
    // BeamToShellTransition.Create(chordBeam, chordShell, 0.4);
    
    printed("壳模型网格: 全局 50mm, 相贯线区域 15mm");
} else {
    // 梁模型: 仅需单元尺寸
    meshCtrl.SetGlobalElementSize(0.08);  // 梁模型 80mm 即可
}

// ============================================================
// 阶段 11: 载荷定义 (Joint Analysis Load Cases)
// ============================================================
// LC 501: 撑杆1轴向受拉
var lcBrace1Tension = LoadCase.Create();
lcBrace1Tension.name = "Joint_Brace1_Tension";
lcBrace1Tension.type = Live;
lcBrace1Tension.number = 501;

// 撑杆1端部施加轴向拉力 1000 kN (沿杆轴, Z方向)
lcBrace1Tension.AddNodalForce(
    brace1Beam, 1.0,  // 梁参数位置 1.0 = 端部
    0, 0, 1000e3,     // Fx, Fy, Fz (局部坐标)
    0, 0, 0           // Mx, My, Mz
);

// LC 502: 撑杆2轴向受压
var lcBrace2Comp = LoadCase.Create();
lcBrace2Comp.name = "Joint_Brace2_Compression";
lcBrace2Comp.type = Live;
lcBrace2Comp.number = 502;

lcBrace2Comp.AddNodalForce(
    brace2Beam, 1.0,
    0, 0, -800e3,     // 轴向压力 800 kN
    0, 0, 0
);

// LC 503: 撑杆1弯矩 (面内弯曲)
var lcBrace1IPB = LoadCase.Create();
lcBrace1IPB.name = "Joint_Brace1_IPB";
lcBrace1IPB.type = Live;
lcBrace1IPB.number = 503;

lcBrace1IPB.AddNodalMoment(
    brace1Beam, 1.0,
    0, 150e3, 0,      // My = 150 kN·m (面内弯曲)
    0, 0, 0
);

// LC 505: 组合载荷
var lcCombined = LoadCase.Create();
lcCombined.name = "Joint_Combined";
lcCombined.type = Live;
lcCombined.number = 505;

lcCombined.AddNodalForce(brace1Beam, 1.0, 0, 50e3, 600e3, 0, 100e3, 0);   // Fz + My
lcCombined.AddNodalForce(brace2Beam, 1.0, 30e3, 0, -400e3, 0, 0, 80e3);   // Fx + Fz + Mz

// ============================================================
// 阶段 12: 节点承载力校核设置 (Joint Capacity Check)
// ============================================================
// 当使用梁单元模型时, 可进行管节点冲剪校核
if (!useShellModel) {
    // 激活管节点校核
    var jointCheck = JointCapacityCheck.Create();
    jointCheck.name = "JC_Tubular_Node1";
    jointCheck.type = TubularJoint;
    
    // 指定弦杆与撑杆
    jointCheck.SetChord(chordBeam);
    jointCheck.AddBrace(brace1Beam);
    jointCheck.AddBrace(brace2Beam);
    
    // 校核规范选择
    jointCheck.SetDesignCode(API_RP2A_WSD);   // 或 ISO19902, NORSOK N-004
    
    // 校核类型
    jointCheck.SetCheckType(PunchingShear);    // 冲剪校核
    jointCheck.SetCheckType(ChordWallBearing); // 弦杆壁承载
    
    // 执行校核 (在 Solve 后)
    // jointCheck.Run();
    
    printed("管节点校核已设置: 规范=API RP2A WSD, 类型=冲剪+弦杆壁承载");
}

// ============================================================
// 阶段 13: 分析执行
// ============================================================
var act = Activity.Create();
act.name = "Activity_JointAnalysis";

var meshSet = MeshSet.Create();
meshSet.name = "Mesh_TubularJoint";
meshSet.AddAllBodies();
meshSet.SetMeshControl(meshCtrl);
meshSet.GenerateMesh();

// 为每个载荷工况创建求解器
var solvers = [];
var solverNames = ["Brace1Tension", "Brace2Comp", "Brace1IPB", "Combined"];
var loadCases  = [lcBrace1Tension, lcBrace2Comp, lcBrace1IPB, lcCombined];

for (var s = 0; s < solvers.length; s++) {
    var solv = LinearStaticSolver();
    solv.name = "Solve_" + solverNames[s];
    solv.AddLoadCase(loadCases[s]);
    solvers.push(solv);
}

var analysis = Analysis.Create();
analysis.name = "Analysis_TubularJoint";
for (var a = 0; a < solvers.length; a++) {
    analysis.AddSolver(solvers[a]);
}
analysis.SetActivity(act);

// analysis.Solve();

// ============================================================
// 阶段 14: 结果输出
// ============================================================
print("===== 管节点模型创建完成 =====");
print("节点类型: Y/T型管节点 (弦杆+2撑杆)");
print("弦杆: D=" + (chordRadius*2).toFixed(3) + "m, t=" + secChord.wallThickness.toFixed(3) + "m");
print("撑杆1: D=" + secBrace1.od.toFixed(3) + "m, t=" + secBrace1.wallThickness.toFixed(3) + "m (垂直)");
print("撑杆2: D=" + secBrace2.od.toFixed(3) + "m, t=" + secBrace2.wallThickness.toFixed(3) + "m (45°)");
print("β1=" + beta1.toFixed(3) + ", β2=" + beta2.toFixed(3));
print("τ1=" + tau1.toFixed(3)  + ", τ2=" + tau2.toFixed(3));
print("模型类型: " + (useShellModel ? "壳单元" : "梁单元"));
print("载荷工况: 4 (轴拉/轴压/面内弯/组合)");
print("===============================");
```

## 使用说明

1. `useShellModel` 开关控制使用壳单元还是梁单元分析管节点
2. 壳单元模型提供更准确的局部应力分布，但计算量大
3. 梁单元模型可使用内置的管节点冲剪校核功能
4. 相贯线区域网格尺寸 15mm 可精确捕捉热点应力
5. 撑杆长度应 ≥ 4×弦杆直径以消除端部效应影响
6. 实际项目中应验证直径比 β 和壁厚比 τ 在校核规范的适用范围内
