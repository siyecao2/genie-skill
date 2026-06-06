# GeniE JS 脚本示例集

> 所有脚本来自 SESAM 安装目录 `Help\Tutorials\` 和 `Help\GuidingDocuments\`，可直接通过 `File → Read Command File` 执行。

---

## 标准脚本模板

每个 Tutorial/GuidingDocument JS 脚本遵循统一结构：

```javascript
// 1. 版本兼容性
GenieRules.Compatibility.version = "V8.2-04";

// 2. 容差设定
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 2 deg;
GenieRules.Meshing.autoSimplifyTopology = true;
GenieRules.Meshing.eliminateInternalEdges = true;

// 3. 默认偏置与复制
GenieRules.BeamCreation.DefaultCurveOffset = ReparameterizedBeamCurveOffset();
GenieRules.Transformation.DefaultConnectedCopy = false;

// 4. 单位设定 (可选)
GenieRules.Units.setOutputUnits("m", "kN", "delC");
GenieRules.Units.setInputUnit(Length, "m");
GenieRules.Units.setInputUnit(Force, "kN");
```

---

## Tutorial JS 脚本清单 (21个)

| 教程 | JS 文件 | 行数 | 核心内容 |
|------|---------|:---:|------|
| B1 | B1_GeniE_Basics_input.js | ~30 | GuidePlane→Beam→Support→Load→Analysis |
| B2 | Small_Topside_input.js | ~200 | 甲板+Equipment+WeightList |
| B2 | SectAndMat.js | ~15 | 材料+截面单独定义模块 |
| B3 | Module_Frame_input.js | ~200 | GuidePlane→Cellar/MainDeck→Columns→CodeCheck |
| B4 | Arch_input.js | ~80 | 曲梁拱+风/雪载荷+重设计 |
| B5 | Topside_input.js | 301 | **上部组块完整**: ModelTransformer复制甲板、柱分段Cone、DeckRow偏移、Equipment+LoadInterface、LoadCombination |
| B5 | Detailed_Modelling_Joint_input.js | ~120 | 梁→壳节点细化 |
| B5 | Both_js_inputs.js | ~5 | 串联调用前两个脚本 |
| B6 | - | - | 导入B5模型做CodeCheck |
| B8 | PiledJacketAnalysis_input.js | ~400 | **导管架全流程**: GuidePlane→Legs/Bracings→Segments+Cone→Pile+Soil→Location→Wave→Hydro→Combos→Analysis |
| B11 | - | - | SACS导入 (无单独JS) |
| A1 | GeniE_Crane_Pedestal_input.js | ~400 | 曲板起重机基座 |
| A2 | GeniE_Semisub_Pontoon_input.js | 1238 | **半潜浮筒完整**: PolyCurve外板→SkinCurves/SweepCurve→explode分板→T-bar加强筋→copyMirror 1/4→1/2→完整模型 |
| A3 | Tubular_Joint_Beams_input.js | ~80 | 管节点梁模型 |
| A3 | Tubular_Joint_Shell_input.js | ~60 | 管节点壳转换 |
| A3 | Tubular_Joint_Shell_Refined_input.js | ~80 | 带精化区的壳模型 |
| A4 | GeniE_Parametric_Semisub_Panel_Modelling.js | ~80 | 参数化变量驱动的半潜面板 |
| A5 | Ship_Cargo_Rail_input.js | ~500 | 船舶货舱段 |
| A7 | GeniE_Semisub_Panel_and_FE_input.js | 915 | **半潜T1/T2/T3三模型**: SkinCurves+SweepCurve→WetSurface→DummyHydroPressure→explode→ModelTransformer→Compartments→Equipment→Support→3mMesh |
| A8 | Transportation_With_Contact_input.js | ~30 | PPC Gap/Contact分析 |
| A8 | Jacket.js | ~300 | 导管架模型（被Contact脚本引用） |
| A8 | Rotate_jacket_for_transportation.js | ~20 | 旋转导管架至运输姿态 |
| A15 | SoilCurves_all.js | ~100 | 大直径单桩PISA曲线(DM/BS/BM) |

---

## GuidingDocuments JS 脚本 (9个)

| 文档 | JS 文件 | 行数 | 核心内容 |
|------|---------|:---:|------|
| Parametric | Frame.js | 385 | **空间框架模板**: 21截面(AISC/NVS/INEXA)/3材料/95梁/3板/6支座→PrismEquipment→placeAtPoint→LoadCases→Analysis+Set |
| Parametric | Corrugated.js | 107 | **参数化波纹板**: do...while生成GuideLine→SweepCurve→divide(XPlane3d)→move3Point→Set |
| Parametric | NamePattern.js | 175 | **自动命名+空间框架生成**: GetName/GetObjectByName函数→MakeBeams(嵌套循环生成X/Y/Z/Diag梁) |
| Parametric | Report.js | 52 | **报表生成**: plotResult函数→ModelView+ResultPresentation→Graphics.saveImage→Figure→Report→saveAs(Word XML) |
| Parametric | Param_plate_in.js | ~80 | 参数化板输入 |
| Orthotropic | orthotropic_materials_for_corrugated_panel.js | ~60 | 波纹板正交材料 |
| Orthotropic | orthotropic_materials_for_non_continous_corrugated_panel.js | ~60 | 非连续波纹板正交材料 |
| Orthotropic | orthotropic_materials_for_stiffened_panel.js | ~50 | 加筋板正交各向异性材料 |
| Pile_Soil | Pile_Soil_Training_Example.js | 330 | **桩土分析完整示例**: Soil→SoilData→Scour→Location→PileSoilAnalysis→Splice |

---

## 关键技术模式

### 1. ModelTransformer 变换模式

```javascript
// 平移复制
MyModelTransformerMap = ObjectNameMap();
MyModelTransformerMap.Add(Bm2, "Bm15");   // 源对象→新对象名
ModelTransformer(MyModelTransformerMap).copyTranslate(Vector3d(0, 0, 6));

// 镜像复制
ModelTransformer(MyModelTransformerMap).copyMirror(Point(0, 0, 0), Vector3d(1, 0, 0));

// 旋转复制（最后一个参数=复制份数）
ModelTransformer(MyModelTransformerMap).copyRotate(Point(27.36 m,27.36 m,0 m), Vector3d(0,0,1), 90, 3);
```

### 2. 板壳创建模式

```javascript
// SkinCurves: 曲线放样
Pl1 = SkinCurves(Array(Curve1, Curve5));

// SweepCurve: 沿方向/曲线扫描
Pl2 = SweepCurve(Curve5, Vector3d(-40, 0, 0));
Pl3 = SweepCurve(Curve18, Curve5);  // 沿曲线

// CoverCurves: 封闭边界填面
Pl4 = CoverCurves(Curve2, Curve3, Curve4);
```

### 3. 梁偏心偏移模式

```javascript
// 平齐板顶
Bm1.CurveOffset = AlignedCurveOffset(frFlushTop, 0 m);

// 线性变化偏移
Bm32.CurveOffset = ReparameterizedBeamCurveOffset(
    LinearVaryingCurveOffset(
        ConstantCurveOffsetAtPoint(Vector3d(0,0,-0.195)),
        ConstantCurveOffsetAtPoint(Vector3d(0,0,0.295)), false));
```

### 4. 柱分段+变截面模式

```javascript
// 先延伸梁端
Bm28.extendEnd(1, 4);
// 分段
Bm28.divideSegmentAtEccentric(1, 0.05);
Bm28.divideSegmentAtEccentric(2, 0.1053);
// 锥段 (DynamicThickness=1 取大壁厚)
Cone = ConeSection(1, true);
// 分配截面
Bm28.SetSegmentSection(1, P_leg_large_35);
Bm28.SetSegmentSection(2, Cone);
Bm28.SetSegmentSection(3, P_leg_norm_35);
```

### 5. 复杂板爆炸分解

```javascript
// 复杂板验证+分解为简单部分
Validate(Pl10.primitivePartCount == 3);
Pl10.explode(IndexedNameMask(11));
Validate(Pl11); Validate(Pl12); Validate(Pl13);
Delete(Pl11); Delete(Pl12);  // 删除不需要的子板
```

### 6. 分析运行模式

```javascript
Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());
Analysis1.add(LinearAnalysis());
Analysis1.add(LoadResultsActivity());
Analysis1.setActive();
Analysis1.step(1).subset = Wet_Surfaces;  // 子集网格
SimplifyTopology();
Analysis1.execute();
```

### 7. 湿表面+舱室面板模型

```javascript
WS1 = WetSurface();
Pl1.front.wetSurface = WS1;           // 逐个表面分配
LC1 = DummyHydroLoadCase(WS1);       // 虚拟水压载荷
cm = CompartmentManager();            // 识别封闭舱
LC2.compartment(Point(...)).globalIntensity = DummyHydroPressure();
```

### 8. 设备放置模式

```javascript
Tank = PrismEquipment(3, 5, 4, 15000);  // 长宽高+质量kg
LC5.placeAtPoint(Tank, Point(20,20,33.5), 
    LocalSystem(Vector3d(1,0,0), Vector3d(0,0,1)));
LC5.equipmentAsMass();                   // 设备作为质量
```

### 9. 网格特征边控制

```javascript
FEdge1 = FeatureEdge(ModelCurve(Point(...), Point(...)));
Num_el = NumberOfElements(2);
FEdge1.numberOfElements = Num_el;
```

---

## 报表生成 (Report.js 模式)

```javascript
function plotResult(loadcasein, setin) {
    ModelView_temp = ModelView();
    ModelView_temp.addElement(DisplayConfiguration("Results - with Mesh", moPaper));
    ModelView_temp.addElement(ResultPresentation());
    ModelView_temp.resultPresentation.resultComponent = rsStress;
    ModelView_temp.resultPresentation.calculationType = rsVonMises;
    // ... 设置可见模型和当前载荷
    var Plotfile = loadcasein.name() + setin.name() + ".jpg";
    Graphics.saveImage(Plotfile);
    var NewPlot = Figure(Plottitle, Plotfile);
    return NewPlot;
}
// 生成Word报表
Case_1 = Report("Case_1");
Case_1.add(ChapterStructure());
Case_1.element(1).add(TablePlateCoordinate());
Case_1.element(1).add(TablePlateProperty());
Case_1.add(plotResult(Operation, UpperDeck));
Case_1.saveAs("Case_1.doc", mrWordXML);
```
