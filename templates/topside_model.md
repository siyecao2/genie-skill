# GeniE 上部组块模型模板 (Topside Model Template)

本模板用于创建海上平台上部组块结构模型，包含甲板、支撑框架、设备载荷和板梁系统。

```javascript
// ============================================================
// 阶段 1: 兼容性设置与项目初始化
// ============================================================
gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);
gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);

var proj = Project.New();
proj.SetName("Topside_MainDeck_Model");
proj.SetDescription("上部组块结构分析 - 主甲板 & 下层甲板");

var tol = ToleranceManager();
tol.SetDefaultTolerance(0.001);

// ============================================================
// 阶段 2: 材料定义
// ============================================================
// 甲板及主要结构: S355
var matS355 = Material.Create();
matS355.name = "S355";
matS355.type = StructuralSteel;
matS355.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS355.SetYieldStress(355e6);
matS355.SetUltimateStress(490e6);

// 次要构件: S275
var matS275 = Material.Create();
matS275.name = "S275";
matS275.type = StructuralSteel;
matS275.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS275.SetYieldStress(275e6);
matS275.SetUltimateStress(410e6);

// ============================================================
// 阶段 3: 截面定义
// ============================================================
// 甲板梁截面 - H型钢
var secDeckBeam = ISection.Create();
secDeckBeam.name = "DECK_BEAM_H400";
secDeckBeam.material = matS355;
secDeckBeam.SetDimensions(0.400, 0.200, 0.012, 0.016);  // H400x200x12x16

var secDeckGirder = ISection.Create();
secDeckGirder.name = "DECK_GIRDER_H600";
secDeckGirder.material = matS355;
secDeckGirder.SetDimensions(0.600, 0.250, 0.016, 0.025);  // H600x250x16x25

// 立柱截面 - 圆管
var secColumn = PipeSection.Create(0.610, 0.025);
secColumn.name = "COLUMN_P610x25";
secColumn.material = matS355;

// 板加劲肋截面 - T型
var secStiffener = TSection.Create();
secStiffener.name = "STIFFENER_T150x150";
secStiffener.material = matS275;
secStiffener.SetDimensions(0.150, 0.150, 0.010, 0.010);

// ============================================================
// 阶段 4: 几何参数定义
// ============================================================
var deckLength = 30.0;   // 甲板长度 X方向 (m)
var deckWidth  = 20.0;   // 甲板宽度 Y方向 (m)
var zMainDeck  = 60.0;   // 主甲板高程 (m)
var zLowerDeck = 52.0;   // 下层甲板高程 (m)
var zBottom     = 45.0;  // 底部支座高程 (m)

// 甲板角点坐标 (以中心为原点)
var corners = [
    {x: -deckLength/2, y: -deckWidth/2},
    {x:  deckLength/2, y: -deckWidth/2},
    {x:  deckLength/2, y:  deckWidth/2},
    {x: -deckLength/2, y:  deckWidth/2}
];

// ============================================================
// 阶段 5: GuidePlane 工作平面
// ============================================================
// 主甲板工作平面
var gpMainDeck = GuidePlane.Create();
gpMainDeck.name = "GP_MainDeck";
gpMainDeck.SetOrigin(Point(0, 0, zMainDeck));
gpMainDeck.SetNormal(Vector(0, 0, 1));
gpMainDeck.snapmode = true;

// 下层甲板工作平面
var gpLowerDeck = GuidePlane.Copy(gpMainDeck, Point(0, 0, zLowerDeck));
gpLowerDeck.name = "GP_LowerDeck";
gpLowerDeck.snapmode = true;

// 底部工作平面
var gpBottom = GuidePlane.Copy(gpMainDeck, Point(0, 0, zBottom));
gpBottom.name = "GP_Bottom";
gpBottom.snapmode = true;

// ============================================================
// 阶段 6: 甲板板建模 (Deck Plates)
// ============================================================
// 主甲板板 (四边板)
var mainDeckPlate = Plate.CreateByPoints([
    Point(corners[0].x, corners[0].y, zMainDeck),
    Point(corners[1].x, corners[1].y, zMainDeck),
    Point(corners[2].x, corners[2].y, zMainDeck),
    Point(corners[3].x, corners[3].y, zMainDeck)
]);
mainDeckPlate.name = "PLT_MainDeck";
mainDeckPlate.thickness = 0.020;  // 20mm 板厚
mainDeckPlate.material = matS355;

// 下层甲板板
var lowerDeckPlate = Plate.CreateByPoints([
    Point(corners[0].x, corners[0].y, zLowerDeck),
    Point(corners[1].x, corners[1].y, zLowerDeck),
    Point(corners[2].x, corners[2].y, zLowerDeck),
    Point(corners[3].x, corners[3].y, zLowerDeck)
]);
lowerDeckPlate.name = "PLT_LowerDeck";
lowerDeckPlate.thickness = 0.016;  // 16mm
lowerDeckPlate.material = matS355;

// ============================================================
// 阶段 7: 支撑框架 (Transverse & Longitudinal Frames)
// ============================================================
// 横向框架 (沿X方向扫描, Y方向跨度)
var frameSpacing = 3.0;  // 框架间距 (m)
var nFramesTrans = Math.floor(deckLength / frameSpacing) + 1;

for (var i = 0; i < nFramesTrans; i++) {
    var xPos = -deckLength/2 + i * frameSpacing;
    
    // 主甲板下横向梁
    var beamT = Beam.Create(
        Line(Point(xPos, -deckWidth/2, zMainDeck), Point(xPos, deckWidth/2, zMainDeck)),
        secDeckBeam
    );
    beamT.name = "FRAME_TRANS_X" + i;
    beamT.offset = FlushTop;  // 顶面与甲板齐平
    
    // 下层甲板下横向梁
    var beamTL = Beam.Create(
        Line(Point(xPos, -deckWidth/2, zLowerDeck), Point(xPos, deckWidth/2, zLowerDeck)),
        secDeckBeam
    );
    beamTL.name = "FRAME_TRANS_L_X" + i;
    beamTL.offset = FlushTop;
}

// 纵向框架 (沿Y方向扫描, X方向跨度)
var nFramesLong = Math.floor(deckWidth / frameSpacing) + 1;

for (var j = 0; j < nFramesLong; j++) {
    var yPos = -deckWidth/2 + j * frameSpacing;
    
    // 主甲板纵向梁
    var beamL = Beam.Create(
        Line(Point(-deckLength/2, yPos, zMainDeck), Point(deckLength/2, yPos, zMainDeck)),
        secDeckGirder
    );
    beamL.name = "FRAME_LONG_Y" + j;
    beamL.offset = FlushTop;
    
    // 下层甲板纵向梁
    var beamLL = Beam.Create(
        Line(Point(-deckLength/2, yPos, zLowerDeck), Point(deckLength/2, yPos, zLowerDeck)),
        secDeckGirder
    );
    beamLL.name = "FRAME_LONG_L_Y" + j;
    beamLL.offset = FlushTop;
}

// ============================================================
// 阶段 8: 立柱建模 (Columns between Decks)
// ============================================================
// 四角立柱 + 中间立柱
var columnPositions = [];
// 四角
for (var cx = 0; cx < 2; cx++) {
    for (var cy = 0; cy < 2; cy++) {
        columnPositions.push({
            x: (cx - 0.5) * deckLength * 0.9,
            y: (cy - 0.5) * deckWidth  * 0.9
        });
    }
}
// 边中点
columnPositions.push({x: 0.0, y: -deckWidth/2 * 0.9});
columnPositions.push({x: 0.0, y:  deckWidth/2 * 0.9});
columnPositions.push({x: -deckLength/2 * 0.9, y: 0.0});
columnPositions.push({x:  deckLength/2 * 0.9, y: 0.0});

var columns = [];
for (var k = 0; k < columnPositions.length; k++) {
    var cp = columnPositions[k];
    var col = Beam.Create(
        Line(Point(cp.x, cp.y, zBottom), Point(cp.x, cp.y, zMainDeck)),
        secColumn
    );
    col.name = "COL_" + k;
    columns.push(col);
}

// ============================================================
// 阶段 9: 设备质点和集中载荷 (Equipment Loads)
// ============================================================
// 设备1: 压缩机 (主甲板中心)
var eq1Mass = PointMass.Create();
eq1Mass.name = "EQ_Compressor";
eq1Mass.position = Point(3.0, 2.0, zMainDeck);
eq1Mass.mass = 25000;       // 25吨
eq1Mass.massMoments = [0, 0, 0];  // 质量惯性矩 (无偏心)

// 设备2: 发电机 (下层甲板)
var eq2Mass = PointMass.Create();
eq2Mass.name = "EQ_Generator";
eq2Mass.position = Point(-5.0, -3.0, zLowerDeck);
eq2Mass.mass = 18000;       // 18吨

// 设备3: 分离器
var eq3Mass = PointMass.Create();
eq3Mass.name = "EQ_Separator";
eq3Mass.position = Point(-4.0, 5.0, zMainDeck);
eq3Mass.mass = 12000;       // 12吨

// ============================================================
// 阶段 10: Set 定义 (分组)
// ============================================================
// 主甲板结构集合
var setMainDeck = Set.Create();
setMainDeck.name = "SET_MainDeck";
setMainDeck.AddObject(mainDeckPlate);
printed("主甲板集合已创建: " + setMainDeck.name);

// 下层甲板结构集合
var setLowerDeck = Set.Create();
setLowerDeck.name = "SET_LowerDeck";
setLowerDeck.AddObject(lowerDeckPlate);
printed("下层甲板集合已创建: " + setLowerDeck.name);

// 立柱集合
var setColumns = Set.Create();
setColumns.name = "SET_Columns";
for (var c = 0; c < columns.length; c++) {
    setColumns.AddObject(columns[c]);
}

// ============================================================
// 阶段 11: 载荷工况定义
// ============================================================
// LC 101: 自重
var lcGravity = LoadCase.Create();
lcGravity.name = "Gravity";
lcGravity.type = Permanent;
lcGravity.number = 101;
lcGravity.ActivateSelfWeight(0, 0, -1, 1.0);

// LC 201: 设备载荷 (活载)
var lcEquipment = LoadCase.Create();
lcEquipment.name = "Equipment";
lcEquipment.type = Live;
lcEquipment.number = 201;
lcEquipment.AddLoad(eq1Mass);
lcEquipment.AddLoad(eq2Mass);
lcEquipment.AddLoad(eq3Mass);

// LC 301: 环境载荷 - 风 (占位)
var lcWind = LoadCase.Create();
lcWind.name = "Wind_South";
lcWind.type = Environmental;
lcWind.number = 301;
printed("风载荷工况已创建 (LC301), 请使用 wind_loads 模板施加风载。");

// ============================================================
// 阶段 12: 载荷组合 (Load Combinations)
// ============================================================
// 极限状态组合: 1.3*自重 + 1.5*设备
var lcULS = LoadCombination.Create();
lcULS.name = "ULS_Comb1";
lcULS.type = ULS;
lcULS.number = 1001;
lcULS.AddLoadCaseFactor(lcGravity,  1.30);
lcULS.AddLoadCaseFactor(lcEquipment, 1.50);

// 正常使用状态: 1.0*自重 + 1.0*设备
var lcSLS = LoadCombination.Create();
lcSLS.name = "SLS_Comb1";
lcSLS.type = SLS;
lcSLS.number = 2001;
lcSLS.AddLoadCaseFactor(lcGravity,  1.00);
lcSLS.AddLoadCaseFactor(lcEquipment, 1.00);

// ============================================================
// 阶段 13: 分析设置
// ============================================================
SimplifyTopology();

var act = Activity.Create();
act.name = "Activity_Topside";

// 网格设置
var meshCtrl = MeshControl.Default();
meshCtrl.SetGlobalElementSize(0.4);   // 全局网格 0.4m
meshCtrl.SetPlateQuadMesh(true);      // 板使用四边形网格
meshCtrl.SetMinElementQuality(0.3);

var meshSet = MeshSet.Create();
meshSet.name = "Mesh_Topside";
meshSet.AddAllBodies();
meshSet.SetMeshControl(meshCtrl);
meshSet.GenerateMesh();

// 求解器
var solver1 = LinearStaticSolver();
solver1.name = "Solve_Topside_ULS";
solver1.AddLoadCombination(lcULS);

var solver2 = LinearStaticSolver();
solver2.name = "Solve_Topside_SLS";
solver2.AddLoadCombination(lcSLS);

// 执行分析
var analysis = Analysis.Create();
analysis.name = "Analysis_Topside";
analysis.AddSolver(solver1);
analysis.AddSolver(solver2);
analysis.SetActivity(act);

// analysis.Solve();  // 取消注释以执行

// ============================================================
// 阶段 14: 结果输出
// ============================================================
print("===== 上部组块模型创建完成 =====");
print("模型: " + proj.name);
print("甲板尺寸: " + deckLength + " x " + deckWidth + " m");
print("主甲板高程: +" + zMainDeck.toFixed(1) + " m");
print("下层甲板高程: +" + zLowerDeck.toFixed(1) + " m");
print("立柱数: " + columns.length);
print("设备质点数: 3");
print("载荷组合: ULS (LC1001) + SLS (LC2001)");
print("===============================");
```

## 使用说明

1. 根据实际甲板尺寸修改 `deckLength` 和 `deckWidth`
2. 根据实际设备重量和位置调整 `PointMass` 定义
3. 立柱布局可改为非均匀网格以适应不规则甲板
4. 运行前确认载荷组合系数符合设计规范要求
