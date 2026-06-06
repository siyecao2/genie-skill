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

### B5: Topside with Detailed Modelling of Joint
- **页数**: 60 | **版本**: V8.2
- **内容**: Meshing Rules → Guiding Geometry → Cellar Deck/Main Deck/Columns → Deck Rows (XZ面1-6 + YZ面A-E) → Loads → Boundary Conditions → Static Analysis → Results → Save → **Detailed Joint Modelling**（梁模型转板壳） → FE Mesh → Combined Beam+Shell分析
- **特点**: 18步完整流程、梁→壳转换、合并模型分析

### B6: Member Code Checking
- **页数**: 29 | **版本**: V8.2
- **内容**: Import B5模型 → Capacity Manager + Capacity Members → Add Run(API WSD) → 修改Buckling数据 → Generate Code Check Loads → 执行校核 → 理解forces/moments → Redesign失败构件 → Rerun分析+校核 → Create Code Check Report
- **特点**: 构件校核全流程、重设计循环、报告生成

### B8: Piled Jacket Analysis
- **页数**: 52 | **版本**: V8.2
- **内容**: New Workspace + Sections/Thicknesses/Materials → Guide Planes → Legs → Bracings → 改变Leg上部截面 → Vertical Stubs at Top → Conductor Supports + Conductors → Simplified Topside → Sets → Mesh Property for Topside Plates → **Piles** → **Soil** → **Location**(环境) → Current and Waves → Wave Load Condition → Hydro Properties → Loads → Wave Load Analysis → Load Combinations → Run → Results
- **特点**: 22步、40m水深4腿导管架中含桩土+波浪载荷完整分析 (需 Wajac+Splice+Sestra license)

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

### A7: Semisubmersible Panel and Structural (FE) Modelling
- **页数**: 42 | **版本**: V8.2
- **内容**: 13步: Guiding Geometry for Pontoon → Column → **Panel Model**(T1.FEM) → Full Model → **Morison Model**(T2.FEM) → Derrick → Compartments → Equipments → Support Points → **FE Structural Model**(T3.FEM)
- **特点**: 同一概念模型生成三种分析模型：Panel+Morison+Structural FE

### A8: Transportation With Contact
- **页数**: 17 | **版本**: V8.3
- **内容**: Create Model → Point-Point Connections(PPC初始间隙) → Gap/Contact Analysis(非线性) → View Results
- **特点**: 导管架运输分析、非线性间隙/接触、PPC用于关闭的初始间隙

### A9: Tension/Compression Analysis
- **页数**: 11 | **版本**: V8.2
- **内容**: 创建10x10x10空间框架 → 选定梁赋予 Truss(Pipe1=compression only, Pipe2=tension only) → 非线性拉压分析 → 结果展示
- **特点**: Truss单元应用、Compression-only(脱开)和Tension-only(拉索)

### A10: Wind Loads
- **页数**: 33 | **版本**: V8.4
- **内容**: Import Jacket+Topside → Wind Profiles(Operating+Storm) → Wind on Beams(由Wajac按Morison方程计算) → Wind on Equipments(GeniE内部计算) → Load Combinations → Structural Analysis(风+浪组合) → Wind on Plates(GeniE) → Wind as Point Loads on Joints
- **特点**: V8.4新增风载荷功能、Wajac+GeniE联合计算

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

### A16: Conversion of Tubular Joints
- **页数**: 24 | **版本**: V8.8
- **内容**: 基于完整导管架模型(预定义Jt1/Jt2/Jt3在-15.0m高程) → Convert joint(s) to shells → 无重叠支杆(Jt1/Jt2)+重叠支杆(Jt3/11根支杆/3组重叠) → 全球网格(应力筛选) + 精化区网格(DNV RP-C203 Ch4.2疲劳)
- **配套文件**: `Tubular_joint_initial.gnx` / `Tubular_joint_completed.gnx` / `Tubular_joint_all_completed.gnx`
- **需要模块**: GeniE V8.8-02 + CGEO + Wajac/Splice/Sestra/Xtract

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
