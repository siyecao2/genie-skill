# GeniE 标准建模工作流程 (Standard Modeling Workflow)

> **严格参照**: SESAM 安装目录真实 JS 脚本 — Frame.js, B5_Topside_input.js, A2_SemisubPontoon_input.js, A7_SemisubPanel_and_FE_input.js, B8_PiledJacketAnalysis

---

## 阶段 1: 项目初始化 (GenieRules + Tolerances)

```javascript
// === 必须首先执行的兼容性设置 ===
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 2 deg;
GenieRules.BeamCreation.DefaultCurveOffset = ReparameterizedBeamCurveOffset();
GenieRules.Meshing.autoSimplifyTopology = true;
GenieRules.Transformation.DefaultConnectedCopy = false;
```

---

## 阶段 2: 材料与截面 (参照 Frame.js 385行 完整实例)

### 2.1 材料 — 直接构造器，不用 `.Create()`

```javascript
// 正确: MaterialLinear 直接构造
S355 = MaterialLinear(
    355000000 Pa,        // 屈服强度
    7850 kg/m^3,         // 密度
    2.1e+011 Pa,         // 杨氏模量
    0.3,                 // 泊松比
    1.2e-005 delC^-1,    // 热膨胀系数
    0.03 N*s/m           // 阻尼
);
S355.name = "S355";

// 设为默认材料
S355.setDefault(Material);

// 低密度补偿材料 (用于加筋板简化)
SuperMat = MaterialLinear(200000000 Pa, 1780 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03);
```

| 规范 | 材料等级 | Yield (MPa) |
|------|---------|:----------:|
| EN 10025 | S235 / S275 / S355 / S420 / S460 | 235-460 |
| API RP 2A | A36 / A572 Gr.50 | 250 / 345 |
| NORSOK M-120 | NV 级 | 见规范 |

### 2.2 截面 — 直接构造器 + 带单位

```javascript
// 正确的 GeniE JS 截面定义 (来自真实脚本)
PIPE_610x25 = PipeSection(0.610 m, 0.025 m);
PIPE_610x25.name = "Pipe_610x25";

HE400B = ISection(0.40 m, 0.30 m, 0.011 m, 0.019 m, 0.027 m);
HE400B.name = "HE400B";

BOX_400x300 = BoxSection(0.40 m, 0.30 m, 0.020 m, 0.020 m);
BOX_400x300.name = "Box_400x300";

// 锥段 (变截面)
CONE = ConeSection(true);  // true=取大壁厚

// T型钢 / 非对称I型
Tbar = UnsymISection(0.3 m, 0.01 m, 0.01 m, 0.006 m, 1e-6 m, 0.12 m, 0.06 m, 0.015 m);

// 扁钢
FB150x15 = BarSection(0.150 m, 0.015 m);
```

---

## 阶段 3: 几何建模

### 3.1 GuidePlane — 直接构造器

```javascript
// 正确: GuidePlane(originPoint, normalVector)
GPBase = GuidePlane(Point(0, 0, 0), Vector3d(0, 0, 1));
GPMain  = GuidePlane(Point(0, 0, 60 m), Vector3d(0, 0, 1));
```

### 3.2 梁 — 用 StraightBeam，不用 Beam.Create

```javascript
// 正确: StraightBeam(startPoint, endPoint)
var leg = StraightBeam(Point(0, 0, 0), Point(16, 16, 60 m));
leg.name = "LEG_A1";
leg.section = PIPE_610x25;

// 梁偏心 (平齐板顶)
leg.CurveOffset = AlignedCurveOffset(frFlushTop, 0 m);

// 变截面分段
leg.divideSegmentAtEccentric(1, 0.33);
leg.SetSegmentSection(1, LEG_BOT);
leg.SetSegmentSection(2, CONE);
leg.SetSegmentSection(3, LEG_TOP);
```

### 3.3 板 — Plate(4点)，不用 Plate.CreateByPoints

```javascript
// 正确: Plate(p1, p2, p3, p4)
MainDeck = Plate(
    Point(-15 m, -10 m, 60 m),
    Point( 15 m, -10 m, 60 m),
    Point( 15 m,  10 m, 60 m),
    Point(-15 m,  10 m, 60 m)
);
MainDeck.name = "MainDeck";
MainDeck.material = S355;
MainDeck.thickness = Thickness(0.025 m);
```

### 3.4 曲面 — SkinCurves + SweepCurve + CoverCurves

```javascript
// 蒙皮 (两条曲线放样)
ColumnShell = SkinCurves(Array(BaseCircle, TopCircle));

// 扫掠 (曲线沿方向)
PontoonBottom = SweepCurve(guideCurve, Vector3d(40, 0, 0));

// 封板 (封闭曲线填面)
BottomPlate = CoverCurves(BaseCircle);
```

### 3.5 模型变换 — ModelTransformer

```javascript
// 镜像复制 (参照 A2 1238行脚本)
// 使用 ObjectNameMap + ModelTransformer.copyMirror / copyTranslate / copyRotate
```

---

## 阶段 4: 载荷 (参照 B5 Topside_input.js)

### 4.1 载荷工况 — LoadCase(ax, ay, az)

```javascript
// 正确: 重力用加速度表示
LCGrav = LoadCase(0, 0, -9.81);
LCGrav.name = "Gravity";

LCOper = LoadCase(0, 0, -9.81);
LCOper.name = "Operation";

// 设备载荷
Pump = PrismEquipment(4 m, 4 m, 5 m, 75000 kg);
LCOper.placeAtPoint(Pump, Point(18 m, 0, 28 m));

// 设备作为偏心质量 (用于水动力分析)
// LCOper.equipmentAsMass();
```

### 4.2 载荷组合 — LoadCombination

```javascript
// 正确: LoadCombination(可选analysis), .addCase(lc, factor)
CombULS = LoadCombination();
CombULS.addCase(LCGrav, 1.0);
CombULS.addCase(LCOper, 1.5);
CombULS.name = "ULS_1";

// 风暴工况设置设计条件
CombStorm = LoadCombination();
CombStorm.addCase(LCGrav, 1.0);
CombStorm.addCase(LCExtreme, 1.6);
// CombStorm.designCondition = lcStorm;  // 代码校核需要
```

| 极限状态 | 永久γ_G | 活载γ_Q | 环境γ_E |
|---------|:------:|:------:|:------:|
| ULS-a   | 1.30   | 1.50   | 0.70   |
| ULS-b   | 1.00   | 0.70   | 1.35   |
| SLS     | 1.00   | 1.00   | 1.00   |
| ALS     | 1.00   | 1.00   | 1.00   |

---

## 阶段 5: 分析执行

### 5.1 拓扑简化

```javascript
// 必须在网格划分前执行
SimplifyTopology();
```

### 5.2 网格 — MeshDensity + Analysis

```javascript
// 全局网格密度
Md_global = MeshDensity();
Md_global.elementLength = 0.5 m;

// 局部网格密度 (管节点区域)
Md_refined = MeshDensity();
Md_refined.elementLength = 0.02 m;  // 20mm for fatigue
```

### 5.3 分析活动

```javascript
// 正确: Analysis(true), .add(activity), .setActive()
Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());
Analysis1.add(LinearAnalysis());
Analysis1.add(LoadResultsActivity());
Analysis1.setActive();

// 可选: 仅网格子集
// Analysis1.step(1).subset = NamedSet_Refined;

// 执行 (取消注释)
// Analysis1.execute();
```

### 5.4 梁单元 vs 壳单元 决策

| 场景 | 推荐单元 | 原因 |
|------|---------|------|
| 整体导管架分析 | 梁单元 | 计算效率高 |
| 管节点疲劳分析 | 壳单元 | 热点应力 (DNV RP-C203) |
| 上部组块板梁 | 梁+板混合 | 板模拟甲板 |
| 起重机基座 | 壳单元 | 局部应力集中 |

---

## 阶段 6: Code Check (规范校核)

```javascript
// 正确: CapacityManager + 标准字符串
var cc = CapacityManager();
cc.name = "CODE_CHECK";
cc.standard = "ISO19902";      // 海中结构
// 可选: "AISC_ASD" / "AISC_LRFD" / "API_WSD" / "EN1993_1_1" / "NORSOK_N004" / "DS449"
cc.setAnalysisResults(Analysis1);

// 构件校核
var memLegs = Member();
memLegs.fromStructure([lega1, lega2, legb1, legb2]);
cc.addMember(memLegs);

// 节点校核 (API 选项)
// cc.jointStandard = "API_WSD_2005";
// cc.jvrGeometricLimits = true;        // 几何限制范围
// cc.jvrModifiedGeometry = true;       // 修正几何范围 (uf435mod)

cc.execute();
```

---

## 常用检查清单

- [ ] GenieRules.Compatibility 已设置
- [ ] 所有构件已赋予截面 (section) 和材料 (material)
- [ ] SimplifyTopology() 已执行
- [ ] 边界条件足够约束刚体运动 (6 自由度)
- [ ] 网格密度已分配
- [ ] 载荷工况系数符合规范
