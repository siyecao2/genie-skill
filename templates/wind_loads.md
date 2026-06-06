# GeniE 风载荷模板 (Wind Loads Template)

> **严格参照**: SESAM `Help\Tutorials\TutorialsAdvancedModelling\A10_GeniE_Wind_Loads\JS\Wind_Loads_input.js` (33页全)

```javascript
// ============================================================
// 阶段 0: GenieRules
// ============================================================
GenieRules.Compatibility.version = "V8.8-08";
GenieRules.Tolerances.useTolerantModelling = true;

// ============================================================
// 阶段 1: 导入模型 (或新建)
// ============================================================
// 导入已有 Jacket+Topside 模型
// File → Import → Sesam XML → SE30_Jacket.xml

// ============================================================
// 阶段 2: 定义风剖面 (Wind Profiles)
// ============================================================
// 操作工况: 25 m/s, API 21st Edition
Wind_OPR = WindProfile();
Wind_OPR.name = "Wind_OPR";
Wind_OPR.formula = "ExtremeAPI21";
Wind_OPR.directionRelativeToWaveHeading = true;
Wind_OPR.direction = 0 deg;
Wind_OPR.averageSpeed = 25 m/s;

// 风暴工况: 50 m/s
Wind_Storm = WindProfile();
Wind_Storm.name = "Wind_Storm";
Wind_Storm.formula = "ExtremeAPI21";
Wind_Storm.directionRelativeToWaveHeading = true;
Wind_Storm.direction = 0 deg;
Wind_Storm.averageSpeed = 50 m/s;

// ============================================================
// 阶段 3: 空气阻力系数 (Air Drag on Beams)
// ============================================================
// 创建 Hydro Property → Air Drag tab
AirDragCoeff = HydroProperty();
AirDragCoeff.airDrag.Cdy = 0.7;   // Y 方向阻力系数
AirDragCoeff.airDrag.Cdz = 0.7;   // Z 方向阻力系数

// 将空气阻力系数赋给导管架和上部组块的梁
// (低于最低波谷的梁不需要选择)

// ============================================================
// 阶段 4: 波浪+风联合载荷 (Wajac 计算)
// ============================================================
// 编辑已有的波浪载荷条件
// Right-click Condition → Edit Wave Load Condition
// 在 Wind profile column 中:
//   - 前3个波指定 Wind_Storm
//   - 后3个波指定 Wind_OPR
// Wajac 按 36 步 @10° 间隔扫描
// Design Load 列选择 MaxOMoment (最大倾覆力矩)

// ============================================================
// 阶段 5: 设备风载荷 (GeniE 计算)
// ============================================================

// 5.1 创建设备
Generator = PrismEquipment(10 m, 4 m, 5 m, 100000 kg);
Generator.name = "Generator";

Pump1 = PrismEquipment(4 m, 20 m, 5 m, 75000 kg);
Pump1.name = "Pump1";

Pump2 = PrismEquipment(4 m, 20 m, 5 m, 75000 kg);
Pump2.name = "Pump2";

// 虚拟设备 (代表非结构板承受风压)
Dummy1 = PrismEquipment(48 m, 1 m, 6 m, 0 kg);
Dummy1.name = "Dummy_NorthWall";

Dummy2 = PrismEquipment(25 m, 1 m, 5 m, 0 kg);
Dummy2.name = "Dummy_EastWall";
// Dummy2 绕 Z 轴旋转 90°

// 5.2 定义风载荷工况
// WindOPR00: 操作+0°, Wind Profile=Wind_OPR
WindOPR00 = LoadCase(0, 0, 0);
WindOPR00.name = "WindOPR00";
// Property → Wind Field tab: Direction=0°, Wind Profile=Wind_OPR

// 5.3 放置设备至工况
WindOPR00.placeAtPoint(Generator, Point(14 m, 3.75 m, 19 m));
WindOPR00.placeAtPoint(Pump1, Point(18 m, 0, 28 m));
WindOPR00.placeAtPoint(Pump2, Point(10 m, 0, 28 m));
WindOPR00.placeAtPoint(Dummy2, Point(-22 m, 0, 36 m));
// Dummy1 不在 0°方向, 故不放置

// 5.4 设备侧向风载荷 (Equipment Side Loads)
// Properties → Equipment Side Loads tab:
//   选择 Wind Pressure → Constant
//   Drag coefficient = 1.0 (迎风面)
//   Suction factor = 0.0 (背风面, 默认)
// Pump1 被 Pump2 遮挡: Drag coefficient = 0.1

// 5.5 生成其余风载荷工况 (复制+修改)
// WindOPR37: Direction=37°, Dummy1 + Dummy2
// WindOPR90: Direction=90°, Dummy1 only
// WindSTM00/37/90: Wind Profile=Wind_Storm

// ============================================================
// 阶段 6: 载荷组合 (Wind + Wave)
// ============================================================
/*
LoadCombination Table (参照 A10 Page 24):

           LCGrav LCEq Wind Analysis.WLC
LCOPR00    1.0   1.1   1.6  1.6 (WLC4)  + 1.0 Buoyancy (WLC7)
LCOPR37    1.0   1.1   1.6  1.6 (WLC5)
LCOPR90    1.0   1.1   1.6  1.6 (WLC6)
LCSTM00    1.0   1.1   1.6  1.6 (WLC1)  + 1.0 Buoyancy
LCSTM37    1.0   1.1   1.6  1.6 (WLC2)
LCSTM90    1.0   1.1   1.6  1.6 (WLC3)
*/

// 创建第一个组合
LCOPR00 = LoadCombination(Analysis2);
LCOPR00.addCase(LCGrav, 1);
LCOPR00.addCase(LCEq, 1.1);
LCOPR00.addCase(Analysis2.WLC(4, 1), 1.6);
LCOPR00.addCase(Analysis2.WLC(7, 1), 1);     // Buoyancy
LCOPR00.addCase(WindOPR00, 1.6);
LCOPR00.name = "LCOPR00";

// 风暴组合设置设计工况 (Storm = Design Condition)
// LCSTM00.designCondition = lcStorm;  // 代码校核需要

// ============================================================
// 阶段 7: 风载荷在板上的定义 (非结构板 - 无需分析)
// ============================================================
// 创建垂直板
// Plate(Point, Point, Point, Point)
// 使用 Loads → Explicit Load → Surface Load → Constant Wind Pressure
// Footprint = Plate

// ============================================================
// 阶段 8: 风载荷作为节点点力 (LoadInterface 模式)
// ============================================================
// 创建设备 Dummy3 (代表整个上部组块迎风面积)
// 创建 LoadInterface (Properties → Load Interface)
// 将 LoadInterface 赋给设备和 6 个柱节点
// Wind pressure → Point Loads on columns

// ============================================================
// 阶段 9: 分析运行
// ============================================================
// Analysis2 运行:
//   Wajac (波浪+梁风载荷) + Sestra (结构分析)
//   所有 6 个工况组合
// Analysis2.execute();
```
