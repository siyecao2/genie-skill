# GeniE 标准建模工作流程 (Standard Modeling Workflow)

完整的 GeniE 结构建模 6 阶段工作流程，从项目设置到后处理与规范校核。

---

## 阶段 1: 项目设置 (Project Setup)

### 1.1 兼容性与工作空间

```javascript
// === 必须首先执行的兼容性设置 ===
gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);
gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);

// 创建项目
var proj = Project.New();
proj.SetName("Project_Name");
proj.SetDescription("项目简要描述");
proj.SetUnitSystem(SI);  // 使用国际单位制 (m, kg, N, Pa)

// 保存路径 (可选)
// proj.SaveAs("C:/Projects/Project_Name/Project_Name.js");
```

### 1.2 公差设置

```javascript
var tol = ToleranceManager();
tol.SetDefaultTolerance(0.001);    // 默认公差 1mm
tol.SetAngularTolerance(0.5);     // 角度公差 0.5度
tol.SetSnapTolerance(0.005);       // 捕捉公差 5mm

// 决策点: 
// - 大型结构 (>50m): 使用 0.005-0.010
// - 中型结构 (10-50m): 使用 0.001-0.005
// - 细部结构 (<10m): 使用 0.0005-0.001
```

---

## 阶段 2: 材料与截面定义 (Materials & Sections)

### 2.1 材料定义

```javascript
// 结构钢材料
var matS355 = Material.Create();
matS355.name = "S355";
matS355.type = StructuralSteel;
matS355.SetLinearIsotropicProperties(
    210e9,    // 弹性模量 E (Pa)
    0.3,      // 泊松比 ν
    7850      // 密度 ρ (kg/m³)
);
matS355.SetYieldStress(355e6);
matS355.SetUltimateStress(490e6);

// 决策点: 根据设计规范选择材料等级
// - DNV-OS-C101: S235, S275, S355, S420, S460
// - API RP 2A: A36, A572 Gr.50
// - NORSOK M-120: NV 级钢材
```

### 2.2 截面定义

```javascript
// 圆管截面
var secPipe = PipeSection.Create(0.610, 0.025);
secPipe.name = "Pipe_610x25";
secPipe.material = matS355;

// H型钢截面
var secISection = ISection.Create();
secISection.name = "HE400B";
secISection.material = matS355;
secISection.SetDimensions(0.400, 0.300, 0.0135, 0.0240);

// 箱型截面
var secBox = BoxSection.Create();
secBox.name = "Box_400x300";
secBox.SetDimensions(0.400, 0.300, 0.020, 0.020, 0.020);
```

---

## 阶段 3: 导引几何与构件创建 (Guiding Geometry + Beam/Plate)

### 3.1 创建工作平面 (GuidePlane)

```javascript
var gpBase = GuidePlane.Create();
gpBase.name = "GP_Base";
gpBase.SetOrigin(Point(0, 0, 0));
gpBase.SetNormal(Vector(0, 0, 1));
gpBase.snapmode = true;  // 关键: 必须启用捕捉

// 移动/复制工作平面
var gpLevel2 = GuidePlane.Copy(gpBase, Point(0, 0, 15.0));
gpLevel2.name = "GP_Level2";
gpLevel2.snapmode = true;
```

### 3.2 创建梁 (Beam)

```javascript
var beam = Beam.Create(
    Line(Point(0, 0, 0), Point(10, 0, 0)),
    secPipe
);
beam.name = "BEAM_1";
beam.offset = Centroid;  // 对齐方式: Centroid, FlushTop, FlushBottom
```

### 3.3 创建板 (Plate)

```javascript
var plate = Plate.CreateByPoints([
    Point(0, 0, 10),
    Point(5, 0, 10),
    Point(5, 5, 10),
    Point(0, 5, 10)
]);
plate.name = "PLT_Deck";
plate.thickness = 0.020;
plate.material = matS355;
```

### 3.4 创建蒙皮曲面 (SkinCurves)

```javascript
// 创建圆柱壁面
var bottomGuide = [Line(Point(0,0,0), Point(1,0,0)), Line(Point(1,0,0), Point(0,0,0))];
var topGuide = [Line(Point(0,0,5), Point(1,0,5)), Line(Point(1,0,5), Point(0,0,5))];
var skin = SkinCurves.Create(bottomGuide, topGuide);
skin.name = "SKIN_Cylinder";
skin.thickness = 0.020;
skin.material = matS355;
```

---

## 阶段 4: 载荷与组合 (Loads & Combinations)

### 4.1 载荷工况创建

```javascript
// 永久载荷 - 自重
var lcGravity = LoadCase.Create();
lcGravity.name = "Gravity";
lcGravity.type = Permanent;        // Permanent / Live / Environmental / Accidental
lcGravity.number = 101;
lcGravity.ActivateSelfWeight(0, 0, -1, 1.0);

// 活载荷 - 设备/操作
var lcLive = LoadCase.Create();
lcLive.name = "Equipment_Load";
lcLive.type = Live;
lcLive.number = 201;

// 环境载荷
var lcEnv = LoadCase.Create();
lcEnv.name = "Wave_100yr";
lcEnv.type = Environmental;
lcEnv.number = 301;
```

### 4.2 载荷组合

```javascript
// 极限状态 (ULS)
var lcULS = LoadCombination.Create();
lcULS.name = "ULS_1";
lcULS.type = ULS;
lcULS.number = 1001;
lcULS.AddLoadCaseFactor(lcGravity, 1.30);   // γ_G = 1.30
lcULS.AddLoadCaseFactor(lcLive, 1.50);       // γ_Q = 1.50

// 正常使用状态 (SLS)
var lcSLS = LoadCombination.Create();
lcSLS.name = "SLS_1";
lcSLS.type = SLS;
lcSLS.number = 2001;
lcSLS.AddLoadCaseFactor(lcGravity, 1.00);
lcSLS.AddLoadCaseFactor(lcLive, 1.00);

// 事故状态 (ALS)
var lcALS = LoadCombination.Create();
lcALS.name = "ALS_ShipImpact";
lcALS.type = ALS;
lcALS.number = 3001;
lcALS.AddLoadCaseFactor(lcGravity, 1.00);
lcALS.AddLoadCaseFactor(lcEnv, 1.00);
```

### 4.3 载荷组合因子参考

| 极限状态 | 永久载荷 γ_G | 活载荷 γ_Q | 环境载荷 γ_E |
|---------|-------------|-----------|-------------|
| ULS-a   | 1.30        | 1.50      | 0.70        |
| ULS-b   | 1.00        | 0.70      | 1.35        |
| SLS     | 1.00        | 1.00      | 1.00        |
| ALS     | 1.00        | 1.00      | 1.00        |

---

## 阶段 5: 分析执行 (Analysis Execution)

### 5.1 拓扑简化

```javascript
// 必须在网格划分前执行
SimplifyTopology();
// 此操作: 合并重合节点, 消除微小边, 清理悬挂点
```

### 5.2 网格划分

```javascript
var meshCtrl = MeshControl.Default();
meshCtrl.SetGlobalElementSize(0.5);          // 全局单元尺寸
meshCtrl.SetMinElementQuality(0.3);          // 最小单元质量 (0-1)
meshCtrl.SetMaxElementAspectRatio(5.0);      // 最大长宽比
meshCtrl.SetPlateQuadMesh(true);             // 四边形网格优先
meshCtrl.SetLocalRefinementOnJoints(0.05, 0.30);  // 节点局部细化

var meshSet = MeshSet.Create();
meshSet.name = "Mesh_Main";
meshSet.AddAllBodies();
meshSet.SetMeshControl(meshCtrl);
meshSet.GenerateMesh();

// 检查网格质量
var meshQuality = meshSet.CheckQuality();
if (meshQuality.failedElements > 0) {
    print("警告: " + meshQuality.failedElements + " 个单元不满足质量要求!");
}
```

### 5.3 求解

```javascript
// 线性静力求解器
var solver = LinearStaticSolver();
solver.name = "Solve_ULS";
solver.AddLoadCombination(lcULS);

// 模态求解器 (可选)
var modalSolver = ModalSolver();
modalSolver.name = "Solve_Modal";
modalSolver.SetNumModes(20);
modalSolver.SetFrequencyRange(0.1, 50.0);

// 活动与分析
var act = Activity.Create();
act.name = "Activity_1";

var analysis = Analysis.Create();
analysis.name = "Analysis_Main";
analysis.AddSolver(solver);
analysis.SetActivity(act);

// 执行
analysis.Solve();
```

### 5.4 决策点: 梁单元 vs 壳单元

| 场景 | 推荐单元 | 原因 |
|------|---------|------|
| 整体导管架分析 | 梁单元 | 计算效率高，结果满足整体设计 |
| 管节点精细分析 | 壳单元 | 需要热点应力 |
| 上部组块板梁 | 梁+板混合 | 板模拟甲板，梁模拟框架 |
| 起重机基座 | 壳单元 | 局部应力集中 |
| 疲劳分析 | 壳单元 | 热点应力精度要求 |

---

## 阶段 6: 后处理与规范校核 (Post-processing & Code Check)

### 6.1 结果提取

```javascript
// 位移结果
var maxDisp = Result.GetMaxDisplacement("Analysis_Main", "Solve_ULS");
print("最大位移: " + maxDisp.value.toFixed(4) + " m @ Node " + maxDisp.node);

// 应力结果
var maxStress = Result.GetMaxVonMisesStress("Analysis_Main", "Solve_ULS");
print("最大von Mises应力: " + (maxStress.value/1e6).toFixed(1) + " MPa");

// 支座反力
var reactions = Result.GetReactionForces("Analysis_Main", "Solve_ULS");
for (var r = 0; r < reactions.length; r++) {
    print(reactions[r].bc + ": Fx=" + (reactions[r].fx/1e3).toFixed(1) + "kN");
}
```

### 6.2 规范校核

```javascript
// 梁构件校核
var codeCheck = BeamCodeCheck.Create();
codeCheck.name = "CC_Beams";
codeCheck.SetDesignCode(DNV_OS_C101);     // 或 API_RP2A_LRFD, ISO_19902, NORSOK_N004
codeCheck.SetStrengthCheckType(ULS_Check);
codeCheck.AddActivity(act);
codeCheck.Run();

// 管节点校核
var jointCheck = JointCapacityCheck.Create();
jointCheck.name = "JC_Tubular";
jointCheck.SetDesignCode(API_RP2A_WSD);
jointCheck.AddActivity(act);
jointCheck.Run();
```

### 6.3 结果导出

```javascript
// 导出到 Xtract 进行高级后处理
Result.ExportToXtract("Analysis_Main", "C:/Results/filename.xtr");

// 导出报告
Report.GenerateAnalysisReport(analysis, "C:/Reports/Analysis_Report.pdf");
```

---

## 常用检查清单

- [ ] 兼容性规则已设置
- [ ] 所有构件已赋予材料和截面
- [ ] 所有 GuidePlane 的 snapmode = true
- [ ] SimplifyTopology() 已在网格前执行
- [ ] 边界条件足够约束刚体运动
- [ ] 网格质量满足要求
- [ ] 载荷工况编号不重复
- [ ] 载荷组合系数符合规范
