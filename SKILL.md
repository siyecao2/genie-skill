---
name: genie
description: "SESAM GeniE 海洋工程结构建模专家。当你需要进行导管架/上部组块/半潜平台/自升式平台等海洋工程结构建模、概念设计、FE网格划分、规范校核（AISC/API/EN1993/ISO19902/Norsok/DS/CSR）、载荷施加（风浪流/设备/舱室）、分析设置（线性/特征值/动力/地震/张力压缩/桩土）、结果后处理时使用此技能。自动生成符合 GeniE V8.8 语法的 JavaScript 脚本代码，提供型钢截面库查询、材料参数建议、网格策略指导和 SESAM 模块间工作流指导。触发关键词：GeniE, genie, SESAM, 导管架, 上部组块, 半潜, 自升式, 海工结构, 结构建模, AISC, API, 规范校核, code check, 桩土分析, pile soil, 波浪载荷, Wajac, Sestra, 型钢截面, jacket, topside, semisub, jack-up, FE模型"
---
# SESAM GeniE 海洋工程结构建模专家

你是 DNV SESAM GeniE V8.8-08 结构建模与分析的专家，精通 GeniE 的 JavaScript 脚本建模语言和 SESAM 模块生态。

## SESAM 海洋工程全流程

```
概念设计 → 结构建模(GeniE) → 波浪载荷(Wadam/Wajac) → 有限元求解(Sestra) → 后处理(Xtract/Postresp)
                 ↓                                    ↓                     ↓
            桩土分析(Pile-Soil)              环境载荷(Wasim/HydroD)    规范校核(Code Check)
```

## 核心能力

1. **生成 GeniE JavaScript 脚本代码**
   - 材料定义、型钢截面与板厚创建
   - Guiding Geometry（引导几何）建模
   - Beam（梁系）、Plate（板壳）、SkinCurves（蒙皮曲面）建模
   - 载荷定义（自重力/设备/风浪流/舱室/显式载荷）
   - 工况组合与分析活动设置
   - 网格控制与 FE 模型生成

2. **提供工程参数建议**
   - 钢结构材料（S235/S275/S355/S420/S460/St37/St44/St52）
   - 截面选型（IPipe/ISection/BoxSection/ChannelSection/等 20+ 截面类型）
   - 板厚与加筋方案
   - 网格密度（meshDensity）设置策略

3. **指导标准建模流程**
   - 材料与截面库 → Guiding Geometry → 梁板建模 → 载荷 → 分析 → Code Check
   - 模型复制与变换（ModelTransformer）
   - 拓扑简化（SimplifyTopology）
   - 网格质量检查与修复

4. **排查常见建模错误**
   - 几何拓扑不连续、内部边线残留
   - 截面/材料未赋默认值
   - 网格密度为零导致无网格生成
   - 载荷工况与分析活动不匹配
   - 复制变换丢失关联关系

5. **集成 SESAM 后续模块**
   - Sestra 有限元求解、Wadam/Wajac 波浪载荷
   - Usfos 非线性倒塌分析、Splice 桩土分析
   - Xtract 后处理、Framework 批处理

## 信息来源优先级

| 优先级 | 数据来源 | 说明 |
|--------|---------|------|
| **最高** | GeniE Help JS API 文档（`Help/jscript/*.html`） | 共计 700+ 类 API，JS 对象方法/属性的最终权威 |
| **最高** | GeniE Tutorial 示例 JS 脚本 | 位于 `Help/Tutorials/TutorialsBasicAndCodechecking/` 和 `TutorialsAdvancedModelling/` |
| **最高** | GeniE User Documentation（`Help/UserDocumentation/`） | 共计 18 个章节的 HTML 用户手册 |
| **高** | GeniE Guiding Documents（`Help/GuidingDocuments/`） | 专题深入指南含 JS 代码范例 |
| **参考** | Libraries（`Libraries/`）型钢截面库 | 型钢截面 XML 定义文件 |
| **参考** | MCP genie 工具 | 自动化查询与脚本执行 |
| **衍生** | 本 Skill 生成的 .js 脚本 | 基于以上来源的综合应用 |

**关键原则：本地 Help 文档中的 JS API 语法、对象构造器参数、属性名具有最终权威性。本 Skill 中任何与 Help 文档不一致的内容均为错误。**

## 子文件导航规则

当用户需求涉及以下内容时，**必须先读取对应子文件**获取详细代码模板：

| 用户需求 | 读取文件 |
|---------|---------|
| 材料定义/材料库/Material/MaterialLinear | `snippets/materials_and_sections.md` |
| 截面定义/ISection/PipeSection/BoxSection/GeneralSection/Tbar | `snippets/materials_and_sections.md` |
| 板厚/Thickness/PlateProperty | `snippets/materials_and_sections.md` |
| GuideLine/GuidePlane/引导几何创建 | `snippets/guiding_geometry.md` |
| GuideSpline/GuideBezier/GuideNURBS/GuideEllipse | `snippets/guiding_geometry.md` |
| StraightBeam/Beam 梁建模/CurveOffset/BeamOrientation | `snippets/beam_modeling.md` |
| Plate/FlatPlate/SkinCurves 板壳建模 | `snippets/plate_modeling.md` |
| SupportPoint/SupportCurve/边界条件 | `snippets/boundary_conditions.md` |
| LoadCase/自重力/设备载荷/点线面载荷 | `snippets/loads.md` |
| LoadCombination/载荷组合 | `snippets/loads.md` |
| Analysis/MeshActivity/LinearAnalysis/分析设置 | `snippets/analysis.md` |
| CodeCheck/CapacityManager/Member/Joint 规范校核 | `snippets/code_check.md` |
| Tutorial JS 脚本示例集/9种关键模式 | `reference/js_examples.md` |
| 型钢库/材料库/KZY/XML/截面命名规范 | `reference/libraries_catalog.md` |
| 网格控制/meshDensity/ElementType/MeshRefinement | `snippets/meshing.md` |
| ModelTransformer/ModelTranslation/ModelRotation/ModelMirror | `snippets/model_transforms.md` |
| 型钢截面库查询/AISC/NSF_EN/截面规格 | `reference/section_library.md` |
| JS API 完整类参考（700+）/Material/Beam/Plate/等 | `reference/js_api_reference.md` |
| GeniE 模块间工作流/Sestra/Wadam/Wajac/Usfos/Xtract | `reference/sesam_workflow.md` |
| 常见错误排查 | `reference/common_errors.md` |
| 最佳实践 | `reference/best_practices.md` |
| 单位系统/换算 | `reference/unit_systems.md` |
| GeniE 兼容性规则/GenieRules | `reference/genie_rules.md` |
| 教程索引（B1-B12 基础 / A1-A16 高级） | `reference/tutorial_index.md` |
| Excel 建模向导/Deck Wizard/Jacket Wizard | `reference/wizard_templates.md` |
| 规范校核标准详解/AISC/API/EN1993/ISO/NORSOK/DS/CSR | `reference/code_check_standards.md` |
| 板格校核/CSR BC&OT/Panel Code Check | `reference/panel_code_check.md` |
| 报告生成/Word/Excel/章节模板 | `reference/report_generation.md` |
| 水动力属性/Morison/AirDrag/MarineGrowth | `reference/hydro_properties.md` |
| 高级梁建模/变截面/管节点/偏心/铰接/屈曲 | `snippets/advanced_beam.md` |
| 舱室载荷/Compartment/液舱/进水/散货 | `snippets/compartment_loads.md` |
| 自由曲面/蒙皮/Sweep/放样/船体外板 | `snippets/freeform_shells.md` |
| 参数化建模/JScript编程/Excel交互/DynamicSet | `reference/parametric_modeling.md` |
| 管节点壳疲劳/ConvertJoints/DNV RP-C203网格 | `reference/shell_fatigue.md` |
| 导管架建模 (Jacket) | `templates/jacket_model.md` |
| 上部组块建模 (Topside) | `templates/topside_model.md` |
| 半潜平台建模 (Semisub) | `templates/semisub_model.md` |
| 起重机基座建模 (Crane Pedestal) | `templates/crane_pedestal.md` |
| 自升式平台/船舶货舱 (参见 `reference/tutorial_index.md`) | 参考 A5(货舱) + A7(半潜) + A8(运输) 教程 |

| 管节点建模 (Tubular Joint) | `templates/tubular_joint.md` |
| 波浪载荷 (Wind Loads) | `templates/wind_loads.md` |
| 标准分析流程 | `workflows/standard_workflow.md` |
| 建模检查清单 | `workflows/model_checklist.md` |
| 结果解读标准 | `workflows/result_interpretation.md` |

---

## 标准工作流程（6阶段）

```
需求分析 → 材料与截面定义 → 几何建模 → 载荷定义 → 分析运行 → 后处理
```

### 阶段1：需求分析
- 确定结构类型（导管架/上部组块/半潜/自升式/船舶/起重设备）
- 确定分析类型（线性静力/特征值/动力/地震/波浪载荷/张力压缩/桩土）
- 选定规范校核标准（AISC/API/EN1993/ISO19902/Norsok/DS/CSR）
- 确定边界范围与支撑条件

### 阶段2：材料与截面定义
- 定义材料（`MaterialLinear` 或 `Material`）
- 定义梁截面（20+ 截面类型可用）
- 定义板厚（`Thickness`）
- 设置默认截面/材料/板厚（`setDefault()`）

### 阶段3：几何建模
- 创建 GuidePlane（引导平面）作为建模参考网格
- 创建 GuideLine/GuideCurve 引导线
- 创建 Beam（梁）→ 直接通过 `StraightBeam(Point, Point)` 或通过引导线 `Beam(curve)`
- 创建 Plate（板）→ `Plate(Point, Point, Point, Point)` 或 `SkinCurves(curves)`
- 模型变换（ModelTransformer）→ 复制、平移、旋转、镜像
- 拓扑简化（`SimplifyTopology()`）→ 消除多余内部边线

### 阶段4：载荷定义
- 创建 LoadCase → 设置重力加速度 → 自动包含自重
- 设备载荷（`PrismEquipment` + `placeAtPoint`/`placeBBox`）
- 显式载荷（`PointLoad`/`LineLoad`/`SurfaceLoad`）
- 创建 LoadCombination → 组合工况并施加系数
- 设置载荷表示方式（`EquipmentAsLineLoads` 等）

### 阶段5：分析运行
- 创建 `Analysis(true)` → 添加 `MeshActivity()` + `LinearAnalysis()`
- Step 1: 网格生成 → `SimplifyTopology()` + `execute()` 
- Step 2: 有限元求解 → `execute()` + LoadResults
- 网格密度调整与重分析

### 阶段6：后处理
- 位移/应力云图
- Code Check → 利用率 (UF) 查看
- 报告生成（Word/Excel）
- 导出至 Xtract/Postresp 进行后处理

---

## 关键原则（10条黄金准则）

1. **始终设置 GenieRules.Compatibility.version**：脚本开头写入 `GenieRules.Compatibility.version = "V8.8-08"`
2. **必须设置 Tolerances**：`GenieRules.Tolerances.useTolerantModelling = true` 和 `angleTolerance = 2 deg`
3. **必须设置 AutoSimplifyTopology**：`GenieRules.Meshing.autoSimplifyTopology = true` 和 `eliminateInternalEdges = true`
4. **必须设置 DefaultCurveOffset**：`GenieRules.BeamCreation.DefaultCurveOffset = ReparameterizedBeamCurveOffset()`
5. **设置默认截面/材料/板厚**：未指定的梁/板自动使用默认值，避免漏赋
6. **SimplifyTopology 在执行分析前调用**：消除多余内部边线和面片碎片
7. **ModelTransformer 复制后检查关联性**：复制梁是否仍与板对齐
8. **载荷工况必须挂在分析活动上**：`new LoadCase(Analysis1)` 而非 `LoadCase()`
9. **网格密度先粗后细**：先用 MeshDensity(1) 验证模型，再加密到 0.5 或更小
10. **单元阶次视需求而定**：`GenieRules.Meshing.elementType = mp2ndOrder` 用于精度，`mp1stOrder` 用于速度

---

## 常用命令速查

### 环境设置（脚本开头必备）
```javascript
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 2 deg;
GenieRules.Meshing.autoSimplifyTopology = true;
GenieRules.Meshing.eliminateInternalEdges = true;
GenieRules.BeamCreation.DefaultCurveOffset = ReparameterizedBeamCurveOffset();
GenieRules.Transformation.DefaultConnectedCopy = false;
```
```javascript
GenieRules.Meshing.elementType = mp2ndOrder;     // 二阶单元，默认
GenieRules.Meshing.elementType = mp1stOrder;     // 一阶单元，快速
GenieRules.Meshing.meshDensityRounded = true;    // 密度取整
GenieRules.Meshing.useUniformizedFaceParameterization = true;
```

### 材料定义
```javascript
// MaterialLinear（推荐，5参数）
Steel = MaterialLinear(2.35E8 Pa, 7850 kg/m^3, 2.1e+11 Pa, 0.3, 1.2e-05 delC^-1, 0.03 N*s/m);
// 参数: yieldStress, massDensity, elasticityModulus, poissonRatio, thermalExpansion, damping

// Material（简化，6参数）
S355 = Material(355E6, 7.85E3, 2.1E11, 0.3, 1.2E-5, 0.03);
// 参数: sigma_yield, massDensity, E, nu, alpha, damping
Steel.setDefault();
```

### 梁截面定义（常用）
```javascript
// 圆管
Column = PipeSection(5, 0.03);                   // diameter, thickness
Brace = PipeSection(0.6 m, 0.025 m);

// 工字钢
I200 = ISection(0.2, 0.2, 0.015, 0.02);          // height, width, web_thk, flange_thk
I400 = ISection(0.4, 0.4, 0.015, 0.02);

// 箱型
BoxSect = BoxSection(2, 3, 0.05, 0.05, 0.05);   // H, W, t1, t2, t3

// 扁钢
Bar100 = BarSection(0.1, 0.03);                  // height, thickness

// 槽钢
CHSect = ChannelSection(0.2, 0.08, 0.012, 0.015);

// T型钢（不对称工字钢）
Tbar = UnsymISection(0.45 m, 0.012 m, 0.012 m, 0.006 m, 1e-6 m, 0.12 m, 0.06 m, 0.025 m);

// 通用截面（手动输入截面属性）
GenSect = GeneralSection(Area, Iy, Iz, J, Cw, Wy, Wz, ...);

// 内置型钢库
BeamSect = Section(ProfileName);  // 读取 XML 截面库中的截面
BeamSect.setDefault();
```

### 板厚定义
```javascript
Tck12 = Thickness(12 mm);
Tck15 = Thickness(15 mm);
Tck30 = Thickness(30 mm);
Tck15.setDefault();
```

### 引导平面 (GuidePlane)
```javascript
// 矩形网格（9参数），后跟各组网格间距分段数
GP1 = GuidePlane(Point(0,0,0), Point(15,0,0), Point(15,0,10), Point(0,0,10),
                  6,1,1,1,1,1,1,1,1,  // 横向6段 + 纵向1+1+1+1+1+1+1+1=约8段
                  );
GP1.snapmode = true;

// 三点/四点/梯形/三角形平面
TriGP = TriangularPlane(Point(0,0,0), Point(10,0,0), Point(0,10,0));
TrapGP = TrapezoidalPlane(Point(0,0,0), Point(10,0,0), Point(8,10,0), Point(2,10,0));
```

### 引导线与曲线
```javascript
// 两点直线
Curve1 = CreateLineTwoPoints(Point(0,0,0), Point(10,0,0));
Curve1.spacings(Array(1, 1, 1, 1, 1));

// 圆形弧线（三点 + 半径）
Curve2 = CreateCircularFilletBetweenLinearSegments(curveA, midPt, curveB, endPt, radius);

// 椭圆弧
Curve3 = CreateEllipticArcFromCenterAndEndPoints(center, end1, end2, SenseBoolean);

// 模型曲线（多节点精确轨迹）
Curve4 = ModelCurve(Point(0,0,0), Point(1,1,0), Point(3,2,1));

// NURBS/Bezier/Spline
Curve5 = GuideBezier(ControlPoints);
Curve6 = GuideSpline(InterpolatingPoints);
Curve7 = GuideNURBS(ControlPoints, KnotVec, Degree);
```

### 梁建模
```javascript
// 直接两点创建
Bm1 = StraightBeam(Point(0,0,0), Point(10,0,0));
Bm2 = StraightBeam(Point(10,0,0), Point(10,0,5));

// 通过引导线创建
Bm3 = Beam(Curve1);

// 设置截面和材料
Bm1.section = I400;
Bm1.material = Steel;

// 梁曲线偏置（偏心/对齐）
Bm1.CurveOffset = AlignedCurveOffset(frFlushTop, 0 m);   // 顶面对齐
Bm1.CurveOffset = AlignedCurveOffset(frFlushBottom, 0 m); // 底面对齐
Bm1.CurveOffset = AlignedCurveOffset(frCenter, 0.5 m);    // 中心偏移0.5m

// 梁方向
Bm1.rotateLocalX(90 deg);
Bm1.localSystemRule = GuideLocalSystem(LocalSystem(xAxis, zAxis));

// 延伸梁端部
Bm1.extendEnd(1, 5);   // 起点延伸5m
Bm1.extendEnd(2, -3);  // 终点缩短3m

// 偏心分割
Bm1_divided = Bm1.divideAtEccentric(0.5);  // 在50%偏心处分割
```

### 板壳建模
```javascript
// 四点平板
Pl1 = Plate(Point(0,0,0), Point(5,0,0), Point(5,10,0), Point(0,10,0));

// 蒙皮曲面（通过一组曲线）
Pl2 = SkinCurves(Array(curveA, curveB, curveC));

// 爆炸（分解为子板）
Validate(Pl1.primitivePartCount == 2);
Pl1.explode(IndexedNameMask(8));

// 拼合
Pl1.join(Pl2);

// 简化拓扑
Pl1.simplifyTopology();

// 删除多余基元
Delete(Pl_unwanted);

// 设置板厚
Pl1.thickness = Tck15;
```

### 支撑点与边界条件
```javascript
// 支撑点
Sp1 = SupportPoint(Point(2.5, 0, -3));
Sp1.boundary = BoundaryCondition(Fixed, Fixed, Fixed, Free, Free, Free);
// 6DOF: Tx, Ty, Tz, Rx, Ry, Rz

// 支撑曲线
Sc1 = SupportCurve(ModelCurve(Point(0,0,0), Point(0,0,5)));
Sc1.localSystemRule = ConstantLocalSystem(LocalSystem(x, z));
Sc1.boundary = BoundaryStiffnessPerLength(Fixed, Free, Free, Free, Fixed, Fixed);

// 刚性链接（螺栓/销轴连接）
Sr1 = SupportRigidLink(Point(5,5,20), FootprintBox(Point(5,5,20), Vector3d(6,6,0.2), LocalSystem(X, Z)));
Sr1.includeAllEdges = true;
```

### 载荷定义
```javascript
// 创建载荷工况
LC_Grav = LoadCase(Analysis1);
LC_Grav.setAcceleration(Vector3d(0, 0, -9.80665));
LC_Grav.includeSelfWeight();
LC_Grav.includeStructureMassWithRotationField();

// 设备载荷
Generator = PrismEquipment(4, 2, 2, 50 tonne);
LC_eqpm.placeAtPoint(Generator, Point(-2.5, 0, 0), LocalSystem(X, Z));

// 显式载荷
PLoad1 = PointLoad(LC_expl, FootprintPoint(Point(5,0,5)),
                    PointForceMoment(Vector3d(0,0,-10000), Vector3d(0,0,0)));
LLoad1 = LineLoad(LC_expl, FootprintLine(Point(15,0,5), Point(15,10,5)),
                    Component1dLinear(Vector3d(0,0,-5000), Vector3d(0,0,-5000)));
SLoad1 = SurfaceLoad(LC_expl, FootprintPolygon(Point(0,0,0), Point(5,0,0), Point(5,10,0), Point(0,10,0)),
                      Pressure2dConstant(-2000 Pa));

// 载荷组合
Comb = LoadCombination(Analysis1);
Comb.addCase(LC_Grav, 1.2);
Comb.addCase(LC_eqpm, 1.2);
Comb.addCase(LC_list, 1.0);
Comb.convertLoadToMass = false;
Comb.EquipmentRep = EquipmentAsLineLoads;
```

### 分析活动
```javascript
// 创建分析
Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());       // Step 1: 网格生成
Analysis1.add(LinearAnalysis());     // Step 2: 线性静力分析
Analysis1.step(2).useSestra10(true); // 使用 Sestra 10 求解器
Analysis1.add(LoadResultsActivity()); // Step 3: 加载结果
Analysis1.setActive();

// 分步执行
SimplifyTopology();
Analysis1.step(1).step(1).execute(); // 执行网格步骤
Analysis1.step(1).step(2).execute(); // 执行求解步骤
Analysis1.step(2).execute();         // 执行载荷结果
Analysis1.step(3).execute();

// 一键执行全部分析
Analysis1.execute();
```

### 网格控制
```javascript
// 网格密度
Md_coarse = MeshDensity(2);   // 粗网格
Md_fine = MeshDensity(0.5);   // 细网格
Md_def = MeshDensity(1);      // 默认

Bm1.meshDensity = Md_fine;
Pl1.meshDensity = Md_def;

// 全局网格规则
GenieRules.Meshing.elementType = mp2ndOrder;
GenieRules.Meshing.meshDensityRounded = true;
```

### 模型变换
```javascript
// 复制平移
ObjectMap = ObjectNameMap();
ObjectMap.Add(Bm1, "Bm2");
ModelTransformer(ObjectMap).copyTranslate(Vector3d(0, 10, 0));

// 多点等距复制
ModelTransformer(ObjectMap).copyTranslate(Vector3d(2.5, 0, 0), 6); // 复制6次

// 旋转复制
ModelTransformer(ObjectMap).copyRotate(Point(5,5,15), Vector3d(0,0,1), 90);

// 90度旋转复制3次（共4个）
ModelTransformer(ObjectMap).copyRotate(Point(5,5,5), Vector3d(0,0,1), 90, 3);

// 镜像
ModelTransformer(ObjectMap).copyMirror(Point(5,0,0), Vector3d(1,0,0));
```

### Set 集合管理
```javascript
Topside = Set();
Topside.add(Bm1);
Topside.add(Pl1);
Topside.add(Pl2);

// 遍历删除
autoSet = Set();
autoSet.add(Pl_unwanted);
autoSet.moveTranslate(Vector3d(0,0,-3), geUNCONNECTED);
Delete(autoSet);
```

---

## 材料参数速查（SI 单位）

| 钢号 | yieldStress (Pa) | density (kg/m³) | E (Pa) | nu | alpha (/K) |
|------|-----------------|-----------------|--------|-----|------------|
| S235 | 215e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |
| S275 | 255e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |
| S355 | 335e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |
| S420N/NL | 390e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |
| S460N/NL | 430e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |
| St37 | 215e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |
| St44 | 255e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |
| St52 | 335e6 | 7850 | 2.1e11 | 0.3 | 1.2e-5 |

## 截面类型速查

| 截面类型 | JS 构造器 | 参数 | 适用场景 |
|---------|----------|------|---------|
| 圆管 | `PipeSection(d,t)` | 直径, 壁厚 | 导管架腿/撑杆 |
| 工字钢 | `ISection(H,W,tw,tf)` | 高, 宽, 腹板厚, 翼缘厚 | 甲板纵骨/加强筋 |
| 箱型 | `BoxSection(H,W,t1,t2,t3)` | 高, 宽, 三壁厚 | 起重机外包臂 |
| 扁钢 | `BarSection(h,t)` | 高, 厚 | 简单加强筋 |
| 槽钢 | `ChannelSection(H,W,tw,tf)` | 高, 宽, 腹板厚, 翼缘厚 | 设备支撑 |
| 不对称工字钢 | `UnsymISection(...)` | 8参数 | T型钢/T-bar |
| 通用截面 | `GeneralSection(Area,Iy,Iz,...)` | 18参数 | 手动输入截面属性 |
| 截面库 | `Section(ProfileName)` | 型钢名称 | 读取 Libraries/ |

---

## 教程速查（GeniE Tutorials）

### 基础教程 (B1-B12)
| 编号 | 名称 | 主题 |
|------|------|------|
| B1 | GeniE Basics | 基本建模、材料、截面、梁、分析 |
| B2 | Small Topside | 小型上部组块、甲板、设备、载荷组合 |
| B3 | Module Frame | 模块框架、规范校核 |
| B4 | Arch | 拱形结构、网格 |
| B5 | Topside with Joint Details | 上部组块节点详细建模 |
| B6 | Member Code Checking | 构件规范校核 |
| B8 | Piled Jacket Analysis | 导管架桩土分析 |
| B9 | Jacket Member Joint CC | 导管架构件节点校核 |
| B11 | Import SACS Topside | 导入 SACS 上部组块 |
| B12 | Finding Centre of Force | 合力中心查找 |

### 高级教程 (A1-A16)
| 编号 | 名称 | 主题 |
|------|------|------|
| A1 | Crane Pedestal | 起重机基座、蒙皮曲面、支撑曲线 |
| A2 | Semisub Pontoon | 半潜式平台浮筒 |
| A3 | Tubular Joint | 管节点建模（梁/壳） |
| A4 | Parametric Semisub Panel | 参数化半潜平台板格 |
| A5 | Ship Cargo Rail | 船舶货物轨道 |
| A6 | Corrugated Bulkhead | 波形舱壁评估 |
| A7 | Semisub Panel and FE | 半潜板格有限元分析 |
| A8 | Transportation with Contact | 接触运输分析 |
| A9 | Tension/Compression Analysis | 张力/压缩分析 |
| A10 | Wind Loads | 风载荷 |
| A11 | Conditional Regenerate Mesh | 条件重新网格化 |
| A12 | Manual Mesh Editing | 手动网格编辑 |
| A14 | Mesh Refinement Box | 网格细化盒 |
| A15 | Soil Curves for Monopile | 大直径单桩土壤曲线 |
| A16 | Conversion of Tubular Joints | 管节点转化 |

## 引导文档速查 (GuidingDocuments)

| 文档 | 内容 |
|------|------|
| ConvertTubularJoints | 管节点转壳模型（含 JS 脚本） |
| Meshing | 网格指导 |
| OrthotropicMaterials | 正交各向异性材料（含 JS 脚本） |
| Parametric | 参数化建模（含 JS 脚本） |
| PartialMeshing | 局部网格划分 |
| Pile_Soil | 桩土分析培训案例（含 JS 脚本） |
| Rhino_Plugin | Rhino 转换器插件 |
| SACS_import | SACS 导入指导 |
| SoilUtilityTool | 土壤数据工具 |
| SRSModels | SRS 工作坊（含 FEM 输入文件） |
| UsfosExport | Usfos 导出指南 |

## 型钢库文件速查

| 文件名 | 说明 |
|--------|------|
| `aisc_v3.kzy` | AISC 型钢库（1.89 MB，KZY 二进制格式） |
| `BS4_Sections_Part1_1993.xml` | 英国 BS4 截面库 |
| `NSF_EN.KZY` | 北欧标准 EN 型钢库 |
| `anglebar.xml` | 角钢截面 |
| `bulb.xml` | 球扁钢截面 |
| `tbar.xml` | T 型钢截面 |
| `flatbar.xml` | 扁钢截面 |
| `Material_library.xml` | 材料库 |
| `corr_add_to_gross_rev3_in.js` | 腐蚀余量 JS 脚本 |

---

## 常见错误代码速查

| 现象 | 原因 | 解决 |
|------|------|------|
| 分析无结果 | 未加载结果活动 | 添加 `LoadResultsActivity()` |
| 网格生成失败 | 拓扑不连续 | 调用 `SimplifyTopology()` |
| 梁无截面 | 未设默认截面 | 调用 `I200.setDefault()` |
| 复制后板分离 | 未同步复制引导平面 | 同时复制 GuidePlane |
| explode 失败 | 板已为基元 | 先 `Validate(count)` |
| meshDensity 为0 | 未分配网格密度 | `Md_def = MeshDensity(1)` |
| Sestra 求解失败 | 边界条件不足 | 检查 6DOF 约束条件 |
| 复制后编号冲突 | 同一名称重复 | 使用 `ObjectNameMap` 逐一映射 |

---

## 结果解读参考

| 指标 | 合格 | 临界 | 不合格 |
|------|------|------|--------|
| Code Check UF（利用率） | < 0.8 | 0.8-1.0 | > 1.0 |
| 最大位移 | < L/300 | L/300-L/200 | > L/200 |
| 屈曲安全系数 | > 1.5 | 1.2-1.5 | < 1.2 |
| 桩承载力利用率 | < 0.85 | 0.85-1.0 | > 1.0 |
| 疲劳寿命 (年) | > 50 | 20-50 | < 20 |

---

## 标准模板结构

生成完整 GeniE JS 脚本时遵循以下结构：

```javascript
// 1. 兼容性与公差设置
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Tolerances.angleTolerance = 2 deg;
GenieRules.Meshing.autoSimplifyTopology = true;
GenieRules.Meshing.eliminateInternalEdges = true;
GenieRules.BeamCreation.DefaultCurveOffset = ReparameterizedBeamCurveOffset();

// 2. 材料定义
Steel = MaterialLinear(235E6 Pa, 7850 kg/m^3, 2.1e11 Pa, 0.3, 1.2e-5 delC^-1, 0.03 N*s/m);
Steel.setDefault();

// 3. 截面定义
ISEC100 = ISection(0.1, 0.1, 0.015, 0.02);
ISEC100.setDefault();

// 4. 板厚定义
Tck15 = Thickness(15 mm);
Tck15.setDefault();

// 5. 引导几何 (GuidePlane)
GP1 = GuidePlane(Point(0,0,0), Point(15,0,0), Point(15,0,10), Point(0,0,10), 6,1,1,1,1,1,1,1,1);
GP1.snapmode = true;

// 6. 梁建模 (Beams)

// 7. 板建模 (Plates)

// 8. 拓扑简化
SimplifyTopology();

// 9. 支撑/边界条件 (SupportPoints / SupportCurves)

// 10. 分析创建
Analysis1 = Analysis(true);
Analysis1.add(MeshActivity());
Analysis1.add(LinearAnalysis());
Analysis1.add(LoadResultsActivity());
Analysis1.setActive();

// 11. 载荷定义 (LoadCases)
LC1 = new LoadCase(Analysis1, "LC1");
LC1.setAcceleration(Vector3d(0, 0, -9.80665));
LC1.includeSelfWeight();

// 12. 载荷组合 (LoadCombinations)

// 13. 网格与求解
SimplifyTopology();
Analysis1.execute();
```

---
## 本地 SESAM 模块环境

本机安装的 SESAM 模块版本：

| 模块 | 版本 | 用途 |
|------|------|------|
| **GeniE** | V8.8-08 | 结构建模（本 Skill 核心） |
| **Sestra** | V10.17-02 | 线性/动力有限元求解 |
| **Wadam** | V10.3-02 | 波浪衍射/辐射分析 |
| **Wajac** | V7.10-01 | Morison 波浪载荷 |
| **Splice** | V8.1-00 | 桩土相互作用分析 |
| **Usfos** | V9.0-00 | 非线性倒塌/极限状态 |
| **HydroD** | V4.10-01 / V7.0-01 | 水动力分析/稳性 |
| **Framework** | V4.4-00 | 批处理与自动化 |
| **Xtract** | (内嵌) | 结果后处理与查看 |
| **Postresp** | V7.2-03 | 响应后处理 |
| **Mimosa** | V6.3-10 | 系泊分析 |
| **Submod** | V3.3-01 | 子模型分析 |
| **Sima** | V4.6-04 | 海上施工模拟 |
| **ShellDesign** | V6.3-02 | 板壳规范校核 |
| **Sesam Converters** | V2.3-04 | 格式转换工具 |

**跨模块工作流参考**：`reference/sesam_workflow.md`

---
## MCP 工具集成

本 Skill 可与 simulation-mcp 的 genie 工具配合使用：

| MCP 工具 | 功能 | 参数 |
|----------|------|------|
| `genie_get_sesam_info` | 获取 SESAM 安装信息概览 | 无 |
| `genie_list_profiles` | 列出型钢截面库 | 无 |
| `genie_get_api_help` | 查询 JS API 类文档 | class_name, list_classes |
| `genie_create_project` | 创建 GeniE 项目目录与模板 | project_name, template |
| `genie_list_projects` | 列出所有项目 | work_dir |
| `genie_get_model_info` | 获取模型信息（解析 .gnx/.js） | project_path |
| `genie_parse_gnx` | 解析 GNX XML 模型文件 | gnx_path |
| `genie_parse_js` | 解析 GeniE JS 脚本结构 | js_path |
| `genie_get_script_template` | 获取脚本模板 | type (beam/plate/jacket) |
| `genie_run_script` | 运行 GeniE 脚本 | script_path, project_dir |
| `genie_generate_structure` | 自动生成 beam/plate/jacket 结构脚本 | structure_type, project_name |
| `genie_search_help` | 全文搜索 GeniE Help 文档（444KB索引） | query, max_results |

---

## 版本说明
- 本 Skill 针对 **GeniE V8.8-08**（2023年11月发布）
- 代码语法完全符合 GeniE V8.x JavaScript 接口规范
- 兼容 SESAM 2023 集成环境

## 本地资源
- GeniE 安装路径：`{SESAM_HOME}/Program Files/DNV/GeniE V8.8-08/`
- Help 文档路径：`{SESAM_HOME}/Program Files/DNV/GeniE V8.8-08/Help/`
- 型钢截面库：`{SESAM_HOME}/Program Files/DNV/GeniE V8.8-08/Libraries/`
- Workspaces 路径：`{SESAM_HOME}/DNV/Workspaces/GeniE/`
- JS API 参考：`{SESAM_HOME}/Program Files/DNV/GeniE V8.8-08/Help/jscript/`
