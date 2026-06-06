# GeniE 管节点模型模板 (Tubular Joint Model Template)

> **严格参照**: SESAM `Help\Tutorials\TutorialsAdvancedModelling\A3_GeniE_Tubular_Joint\JS\Tubular_Joint_Beams_input.js` (梁模型) + `Tubular_Joint_Shell_input.js` (壳模型) + `Tubular_Joint_Shell_Refined_input.js` (精化壳)

```javascript
// ============================================================
// 阶段 0: GenieRules
// ============================================================
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;
GenieRules.Joint.Rules  // 确保 Joint Creation Rules 已设置
GenieRules.Joint.AutoCreateBySpecifiedMember = true;

// ============================================================
// 阶段 1: 材料
// ============================================================
S355 = MaterialLinear(355000000 Pa, 7850 kg/m^3, 2.1e+011 Pa, 0.3, 1.2e-005 delC^-1, 0.03 N*s/m);
S355.setDefault(Material);

// ============================================================
// 阶段 2: 截面 (参照 A3 实际)
// ============================================================
// 弦杆
CHORD = PipeSection(1.200 m, 0.050 m);
CHORD.name = "CHORD_1200x50";

// 支杆
BRACE_1 = PipeSection(0.610 m, 0.025 m);
BRACE_1.name = "BRACE_610x25";

BRACE_2 = PipeSection(0.610 m, 0.025 m);
BRACE_3 = PipeSection(0.508 m, 0.020 m);

// ============================================================
// 阶段 3: 梁模型 - 创建梁
// ============================================================
// 弦杆 (水平, X方向)
var chordP1 = Point(-6.0 m, 0, 0);
var chordP2 = Point( 6.0 m, 0, 0);
var chord = StraightBeam(chordP1, chordP2);
chord.name = "Chord";
chord.section = CHORD;

// 支杆1: 45°斜交
var brace1 = StraightBeam(Point(0, 0, 0), Point(-2.5 m, 0, 2.5 m));
brace1.name = "Brace_1";
brace1.section = BRACE_1;

// 支杆2: 45°对称斜交
var brace2 = StraightBeam(Point(0, 0, 0), Point( 2.5 m, 0, 2.5 m));
brace2.name = "Brace_2";
brace2.section = BRACE_2;

// 支杆3: 垂直
var brace3 = StraightBeam(Point(0, 0, 0), Point(0, 0, 3.0 m));
brace3.name = "Brace_3";
brace3.section = BRACE_3;

// ============================================================
// 阶段 4: 创建预定义节点 (用于梁模型 Code Check)
// ============================================================
// 节点会在梁相交处自动生成
Jt1 = Joint(Point(0, 0, 0));
Jt1.name = "Jt1_Central";

// ============================================================
// 阶段 5: 边界条件
// ============================================================
// 弦杆两端简支
var sp1 = SupportPoint(chordP1);
sp1.boundary = BoundaryCondition(Fixed, Fixed, Fixed, Free, Free, Free);

var sp2 = SupportPoint(chordP2);
sp2.boundary = BoundaryCondition(Free, Fixed, Fixed, Free, Free, Free);

// ============================================================
// 阶段 6: 载荷
// ============================================================
LC1 = LoadCase(0, 0, -9.81);
LC1.name = "Gravity";

LC2 = LoadCase(0, 0, 0);
LC2.name = "Brace_Compression";
// 支杆顶端施加轴力
// (通过 PointLoad 或 BeamEndLoad 施加)

// ============================================================
// 阶段 7: 梁模型分析
// ============================================================
AnalysisBeam = Analysis(true);
AnalysisBeam.add(MeshActivity());
AnalysisBeam.add(LinearAnalysis());
AnalysisBeam.setActive();
SimplifyTopology();
// AnalysisBeam.execute();

// ============================================================
// 阶段 8: 壳模型转换 (Convert Joint to Shells)
// ============================================================
// 方式一: 全局网格 (筛选用)
// 选择节点 Jt1, RMB → Convert joint(s) to shells
// 参数:
//   - Make converted beams non-structural (波浪载荷可传递)
//   - Mesh Density: 200mm
//   - Mesh Option: Patch Surf Quad Mesher (推荐, Advanced Front 将被移除)
//   - Brace Split: 3×BraceDiameter = 1.83m
//   - Chord Split: 3×ChordDiameter = 3.6m
//   - 不勾选 Create Refinement / Transition Zones

// 方式二: 精化网格 (疲劳分析用, DNV RP-C203 Ch4.2)
//   - 勾选 Create Refinement Zones: 2 layers, Total Zone Width = -40mm (2×20mm)
//   - 勾选 Create Transition Zones: 80mm transition
//   - Mesh Density: 20mm (≈ 0.1√(r·t))
//   - Chord plug: 需在波浪载荷分析中断开 (否则误差~19%)

// 方式三: 重叠支杆节点
//   - 确认 Through Brace (Bm700/Bm808/Bm583)
//   - Brace split 需考虑截面连续性和最小长度

// ============================================================
// 阶段 9: 壳模型分析 (在转换后执行)
// ============================================================
AnalysisShell = Analysis(true);
AnalysisShell.add(MeshActivity());
AnalysisShell.add(LinearAnalysis());
AnalysisShell.add(LoadResultsActivity());
AnalysisShell.setActive();

// ============================================================
// 阶段 10: Code Check (梁模型结果)
// ============================================================
/*
var cc = CapacityManager();
cc.name = "TUBULAR_JOINT_CC";
cc.standard = "ISO19902";
cc.setAnalysisResults(AnalysisBeam);

// Member Check
var memChord = Member();
memChord.fromStructure([chord]);

var memBraces = Member();
memBraces.fromStructure([brace1, brace2, brace3]);

// Joint Check (勾选 jvrGeometricLimits 或 jvrModifiedGeometry)
cc.execute();
*/
```
