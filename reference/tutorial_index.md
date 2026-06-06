# GeniE Tutorial Index (教程索引)

> 所有教程 PDF 位于 `Help\Tutorials\`，基于 GeniE V8.2-V8.8，有效期至 2023.11。
> 每个教程都附带对应的 JS 输入文件，可直接 File → Read Command File 完整建模。

---

## Basic Tutorials (基础教程)

### B1: Learning the Basics and Getting Started
- **页数**: 25 | **版本**: V8.2
- **内容**: 建立简单框架模型、定义材料和截面、创建GuidePlane引导面、Beam建模、SupportPoint边界、点载荷、静力分析、结果展示、视图选项
- **特点**: 零基础入门，不需要任何 GeniE 使用经验
- **输入文件**: `B1_GeniE_Basics_input.js`

### B2: Modelling a Topside with Equipment
- **页数**: 49 | **版本**: V8.2
- **内容**: 创建Main Support Frame → 加板 → 板梁平齐(Flush Stiffener) → 扩展甲板 → 建Set → 上层甲板 → 边界条件 → 创建Equipment设备载荷 → 导入Weight List → 显式载荷 → 载荷组合 → 分析运行
- **特点**: 含设备建模和重量清单导入

### B3: Modelling and Code Checking a Module Frame
- **页数**: 24 | **版本**: V8.2
- **内容**: GuidePlane → Cellar Deck → Main Deck → Columns + Bracings → Offsets(Eccentricities格式偏心) → Supports → Loads → Static Analysis → Code Check
- **特点**: 工字钢/箱型/管截面的空间框架 + 代码校核完整流程

### B4: Modelling an Arch
- **页数**: 22 | **版本**: V8.2
- **内容**: Guiding Geometry 引导几何 → Upper/Lower Chord(曲梁) → Column Frame → 完整模型 → 雪+风载荷 → 结构分析 → 结果 → Redesign and Reanalysis(重设计+重分析)
- **特点**: 曲梁建模、重设计循环示范

### B5: Topside with Detailed Modelling of Joint ✅ **完整读取** (60页全)
- **页数**: 60 | **版本**: V8.2
- **截面**: NSF_EN.KZY库导入HE400A/HE600A + 4个Pipe截面(P_leg_norm_15/25, P_leg_large_35, P_brace_20)
- **材料**: Mat1(2E8 yield) / PlateMat(同Mat1但厚度0.015m)
- **步骤**: Meshing Rules(Round off Mesh Density) → Guide Plane(XY平面At 4, 5x5格) → Cellar Deck(HE600A) → Copy 6m→Main Deck(改HE400A) → Columns: 分段(0.5m+1m+3.5m+5m)/Pipe截面+P_leg_large_35+Cone(D1=1m D2=0.5m DynamicThickness=1) → 镜像X+复制Y生成4根柱 → Deck Rows 1-6(XZ面, 管+工字梁+对角撑含FB50X10) → Deck Rows A-E(YZ面类似) → Equipment(3×30t+2×50t点质量+线载荷) → 显式载荷 → 组合(LC_total=Gravity 1.0+Equip 1.0) → 边界(4根柱底全固) → Analysis → 结果 → **Joint转换**: beam→shell(Convert joint to shells/Smart divide/非结构再连) → 板厚赋值+网格密度0.1m → FE Mesh → Combined Beam+Shell分析(同载荷)

### B6: Member Code Checking
- **页数**: 29 | **版本**: V8.2
- **内容**: Import B5模型 → Capacity Manager + Capacity Members → Add Run(API WSD) → 修改Buckling数据 → Generate Code Check Loads → 执行校核 → 理解forces/moments → Redesign失败构件 → Rerun分析+校核 → Create Code Check Report
- **特点**: 构件校核全流程、重设计循环、报告生成

### B8: Piled Jacket Analysis ✅ **完整读取** (52页全)
- **页数**: 52 | **版本**: V8.2
- **内容**: 22步完整导管架建模分析流程
- **截面**: Pipe06(0.6x0.01)/Pipe12(1.2x0.03)/Pipe16(1.6x0.03)/Pipe21(2.1x0.08)/Pipe22(2.2x0.08)/Pipe32(3.2x0.09)
- **板厚**: Th2(0.2m)/Th4(0.4m)/Th6(0.6m)
- **材料**: Steel(3.56E5 kPa yield) / PlateMaterial(0.785 t/m³ 低密度补偿厚板)
- **步骤**: 2个Guide Plane(138m+0m) → Legs(Pipe32)→ 7个高程Bracings(Snap Plane F11交替/水平5/40/75/105/135/138/143m) → 弦侧边斜撑 → Leg分段: Divide at 80m→上部Pipe22+3m锥段(Dynamic Thickness=1取大壁厚) → 镜像复制其余腿 → 顶部增设5m Stub → Conductor支撑架(短横梁Y=3m) + 3根NonStructural Conductor(Pipe06,下部固定) → Simplified Topside(3层甲板+墙+12个120t点质量) → 4个Set(Jacket/Conductors/Legs/Topside) → 2m网格密度 → Pile入土75m(Pipe21,堆尖无限长边界) → **6层Soil**(Sand1/Sand1/Sand2/Clay3/Clay4/Sand5含1/1/1/3/15/30子层) → API1987 p-y / API1993 t-z / API1993 q-z → Scour(总冲刷0.5m+局部1m+坡度20m) → Location(水深124m/水密度1.025/静止水位线+124m) → 电流剖面(与波同向) → 规则波集(振幅15m/周期10s/起始角-60°) → Wave Load Condition(Stokes 5th+CalmSea) → Hydro(Morison Cd=0.7 Cm=2; Flooding=1全进水; 海洋生物0.05m到海底) → 5个LoadCase(Gravity+WindN/E/S/W 5/7/6/9 kPa) → Wave Load Analysis(24步/5°/最大剪力+倾覆力矩) → **8个LoadCombination**(4方向×2极值, WLC 1.6+Buoyancy 1.0+Gravity 1.2+Wind 1.6) → 后台运行Wajac→Sestra→Gensod→Splice → 结果展示
- **特点**: 需Wajac+Splice+Sestra license; 非线性桩土分析不支持Smart Load Combinations

### B9: Jacket Member and Joint Code Checking
- **页数**: 23 | **版本**: V8.2
- **内容**: Import B8模型 → Create Joint Concepts → Capacity Manager(填入Members+Joints) → Add Code Check Run(API WSD) → Generate Loads+Execute → Redesign Member → Redesign Joint → Rerun Analysis+Code Check
- **特点**: 构件+节点双校核 + 重设计

### B11: Import SACS Topside Model
- **页数**: 24 | **版本**: V8.8
- **内容**: Open Workspace → Import SACS Topside(含beams/shells/joints/supports/loads) → Verify Model → Check Beam Properties → Check Wishbones(Leg-Pile连接) → Connect Top of Legs with Piles → Check Supports → Imported Sets → Load Cases/Combinations → Mesh Density → Static Analysis → Results → Code Checking
- **特点**: V8.8新增SACS导入功能，含载荷和组合自动导入

### B12: Finding Centre of Force and Redesigning by Editing Section Properties
- **页数**: 17 | **版本**: V8.2
- **内容**: Import Model → Run Analysis → Find Centre of Force(含自重+设备+点/线/面载荷) → Initial Code Checking → Redesign by Editing Beam Cross Section Properties
- **特点**: CoF计算 + 手动修改截面属性重设计

---

## Advanced Tutorials (高级教程)

### A1: Curved Structure Modelling – Crane Pedestal
- **页数**: 58 | **版本**: V8.2
- **内容**: 17步: Units/Sections → Guiding Geometry → Outer Hull → Deck Plates → Vertical Stiffener Plates → Stiffener Beams → Column → Web Frames → Column Stiffeners → Crane → Boundary Conditions → Analysis + Loads → Verify Model → Coarse Mesh → Finer Mesh → Sets + Even Finer Mesh
- **特点**: 曲板壳结构、多级网格细化

### A2: Curved Structure Modelling – Semisubmersible Pontoons
- **页数**: 59 | **版本**: V8.2
- **内容**: 16步: 材料/截面/板厚 → Outer Hull(Prismatic+Transition+Fore) → Bulkheads+Web Frames → Column → Pontoon Stiffeners → Bulkhead Stiffeners → Longitudinal Bulkhead → Sets → FE Mesh → Half Pontoon → Complete Pontoon → Complete Semisub → Final FE Mesh
- **特点**: 完整半潜平台浮筒建模，含曲板+加强筋+舱壁

### A3: Tubular Joint Modelling
- **页数**: 35 | **版本**: V8.2
- **内容**: Beam Model → Boundary+Loads → Analysis → Results → New Workspace+Import → **Convert Beam to Shell** → FE Mesh → 分析对比 → Convert with **Refinement Zones** → 再分析 → 对比应力确定 SCF
- **特点**: 梁→壳自动转换、粗/精网格对比、SCF计算

### A4: Parametric Semisub Panel Modelling
- **页数**: 10 | **版本**: V8.2
- **内容**: 参数化输入变量定义(Draft/Pon_Width/Pon_Height/Col_Diameter等) → Wet Surface + Dummy Hydro Pressure → 修改变量生成不同尺寸模型 → 输出 Semisub_Panel_T1.FEM 供 HydroD 使用
- **特点**: 纯参数化，输入文件: `Parametric_Semisub_Panel_Model_Input.js`

### A5: Ship Cargo Rail
- **页数**: 50 | **版本**: V8.3
- **内容**: 14步: Guiding Geometry → Plate+Stiffeners(X=-10) → Web Frame+Stiffeners(X=-5) → Copy Web Frame → Hull → Modify Thicknesses → Decks+Stiffeners → Cargo Rail → Copy Model → Boundary → Loads → Analysis
- **特点**: 船舶货舱段截面、Web Frame 复制、加筋板建模

### A6: Evaluation of Corrugated Bulkhead
- **页数**: ~30 | **版本**: V8.3
- **内容**: 波纹舱壁建模与评估
- **特点**: 正交各向异性材料方法评估波纹舱壁

### A7: Semisubmersible Panel and Structural (FE) Modelling ✅ **完整读取** (42页全)
- **页数**: 42 | **版本**: V8.2
- **材料**: Steel(2E8 yield, 7850kg/m³) / SuperMaterial(1780kg/m³密度补偿厚板, 用于简化加筋板建模)
- **步骤**: Guiding Geometry → Pontoon → Column(圆锥过渡) → **T1.FEM Panel模型**: 创建Wet Surface WS1 → Dummy Hydro Pressure载荷(标识水动力面) → 面板网格 → 导出 → **完整模型**: 上下甲板+竖壁 → Guide Curves + Cover圆形板 → Girders(Y向) + 9 Stiffeners(X向间距2.736m) → Flush at top → 1/4镜像→1/2 → **T2.FEM Morison模型**: 2水平柱间支撑(Pipe1) → SE=2 → 导出 → **Derrick**: 4腿+斜撑+水平杆+对角撑(Pipe2/Pipe3)+4×2E5kg点质量 → **Compartments**: Compartment Manager自动识别封闭舱 → 每个液舱单独LC → Dummy Hydro Pressure → **Equipments**: 4×Prismatic Equipment@(±20,±20,33.5) → 设为Eccentric-Mass → **Support Points**: 3点约束浮体刚体位移(旋转自由) → **T3.FEM**: 3m全局网格/SE=3 → 导出

### A8: Transportation With Contact
- **页数**: 17 | **版本**: V8.3
- **内容**: Create Model → Point-Point Connections(PPC初始间隙) → Gap/Contact Analysis(非线性) → View Results
- **特点**: 导管架运输分析、非线性间隙/接触、PPC用于关闭的初始间隙

### A9: Tension/Compression Analysis
- **页数**: 11 | **版本**: V8.2
- **内容**: 创建10x10x10空间框架 → 选定梁赋予 Truss(Pipe1=compression only, Pipe2=tension only) → 非线性拉压分析 → 结果展示
- **特点**: Truss单元应用、Compression-only(脱开)和Tension-only(拉索)

### A10: Wind Loads ✅ **完整读取** (33页全)
- **页数**: 33 | **版本**: V8.4
- **内容**: Import Jacket+Topside(SE30_Jacket.xml) → Wind Profiles: Wind_OPR(25m/s Extreme API 21) / Wind_Storm(50m/s) 方向与波同向 → Wajac风力计算: Air Drag Cdy=Cdz(常数系数) → 6波浪×风剖面组合(3波+Wind_Storm / 3波+Wind_OPR) → Wajac 36步@10° → **GeniE设备风力**: Generator(100t, 10×4×5m)/Pump1(75t, 4×20×5m)/Pump2(75t)/Dummy1(48×1×6m)/Dummy2(25×1×5m, 旋转90°) → Equipment Side Loads(Wind Pressure Constant + Drag coefficient + Suction factor) → Pump1被Pump2遮挡(Drag=0.1) → Dummy设备代表非结构板 → 6风载荷工况 × 3方向 × 2工况 = WindOPR/STM 00/37/90 → **6 LoadCombinations**(WLC 1.6+Buoyancy+Wind 1.6) → Storm工况设Design Condition(代码校核需要) → 风在Plates上(WindOnPlate/Constant Wind Pressure) → 风作为Joints点载荷(Dummy3 + LoadInterface + 6个Joints)

### A11: Conditional Regenerate Mesh
- **页数**: 23 | **版本**: V8.2
- **内容**: 条件重新网格：仅对修改部分重新划分，已锁定的网格保持不变
- **特点**: 大模型中快速局部网格更新

### A12: Manual Mesh Editing
- **页数**: 31 | **版本**: V8.2
- **内容**: Import bracket.xml → Assign Mesh Properties → Subset Meshing(先画fine再画coarse) → Edit: Remove Element/Split Edge → Imprint Node/Manipulate Triangle → Refine → 加Hole概念 + Hole Mesh
- **特点**: 手动控制到节点/单元级、Snapping辅助、子集网格法

### A14: Mesh Refinement Box
- **页数**: 15 | **版本**: V8.2
- **内容**: Import简化船段 → Create Hole+Mesh Option for Hole(孔周细化) → Refinement Within Defined Area(Refine Box特征)
- **特点**: 孔周网格细化 + 矩形区域细化框

### A15: Soil Curves for Large Diameter Monopile
- **页数**: 14 | **版本**: V8.4
- **内容**: PISA JIP研究成果 → 大直径单桩4种土弹簧: P-Y(侧向), DM(分布弯矩), BS(基底水平力), BM(基底弯矩) → 手动输入P-Y/DM/BS/BM曲线 → T-Z/Q-Z按API2014自动计算 → 40层土 → Split V8.0
- **特点**: V8.4新增大直径单桩曲线(超越传统P-Y/T-Z/Q-Z)

### A16: Conversion of Tubular Joints ✅ **完整读取** (24页全)
- **页数**: 24 | **版本**: V8.8
- **内容**: 基于3个预定义节点(Jt1/Jt2/Jt3@-15m)的完整导管架
- **Jt1疲劳筛选**: Select connected beams → Convert joint → **7参数设置**: A.NonStructural / B.200mm全局 / C.Patch Surf Quad Mesher(推荐, AFront将被移除) / D.Brace Split=3.9m / E.Chord Split=7.2m(3×直径) / F.Set名=Jt1_Shells / G.无精化过渡 → LV_200(Linear Edge Growth 1.05) → PSQ网格 → ALT+M生成
- **Jt2精化疲劳**: 同上参数 + Create Refinement Zones / Transition Zones → 2层×20mm(Total Zone Width=-40mm) → 自动创建Jt2RefinementZones/Jt2TransitionZones → 80mm过渡网格 → View_model Set(排除非结构) → 结果: 位移0.532m(Jt1一致)
- **Jt3重叠支杆**: 11根支杆含3组重叠 → Brace Split=2.4m(因Bm588截面变化无锥段、Bm2219长约1.826m) → 识别Through Braces(Bm700/Bm808/Bm583) → 重叠处自动无精化层 → 1层20mm(可能处) → Conditional Regenerate Mesh → 8节点全转换(Tubular_joint_all_completed.gnx) → 结果在GeniE(Xtract)
- **配套文件**: `Tubular_joint_initial.gnx` → `Tubular_joint_completed.gnx` → `Tubular_joint_all_completed.gnx`
- **技术要点**: DNV RP-C203 Ch4.2网格密度=0.1√(r·t), 至少1层径向方形; 弦杆/支杆须截面连续; Overlap自动识别Through Brace; Patch Surf Quad > Advanced Front Quad(2024年将移除)

---

## 学习路径建议

```
入门: B1 → B2 → B3
框架结构: B4(拱) → B5(上组+节点细化) → B6(构件校核)
导管架: B8(桩土+分析) → B9(构件+节点校核)
导入工作流: B11(SACS导入) → B12(合力中心)
曲面建模: A1(起重机基座) → A2(半潜浮筒)
管节点: A3(梁→壳+SCF) → A16(重叠支杆+疲劳网格)
参数化: A4(半潜面板) → A5(货舱段)
完整半潜: A7(3种分析模型)
高级分析: A8(运输接触) → A9(拉压分析) → A10(风载荷)
网格精通: A11(条件重网格) → A12(手动编辑) → A14(细化箱)
桩基: A15(大直径单桩PISA曲线)
```

---

## 教程中的 JS 输入文件

每个教程目录的 `JS\` 子文件夹下均包含对应的 `.js` 输入文件，可通过以下方式使用：

```javascript
// 方式1: 直接读取
File > Read Command File > 选择 Input.js

// 方式2: 代码中引用
ReadCommandFile("Tutorial_Input.js");
```

> 所有教程 JS 文件均由交互式操作自动生成，可以逐个命令学习 GeniE 的 JS API。
