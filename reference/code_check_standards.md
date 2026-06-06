# Code Checking Standards Reference

> **来源**: 基于 `Help\ReferenceDocuments\` 目录下 20+ PDF 文档的实际内容编写。

## 概述

GeniE 通过 CapacityManager 框架支持对 Members（构件）和 Joints（节点）的规范校核。所有实现细节见各 PDF 文档（标注页码）。

---

## 1. AISC 360 (ANSI/AISC 360-xx)

**PDF**: `AISC.pdf` (14页) — "Implementation of AISC 360" (User Manual Vol 4 App C2)

**支持的版本**:
| 版本 | 对应手册 |
|------|---------|
| 360-05 (2005.3.9) | Steel Construction Manual 13th |
| 360-10 (2010.6.22) | Steel Construction Manual 14th |
| 360-16 (2016.7.7) | Steel Construction Manual 15th |

**两种设计方案**:
- **LRFD**（荷载与抗力系数设计）→ `AiscLrfdRun`，使用 resistance factors
- **ASD**（允许强度设计）→ `AiscAsdRun`，使用 safety factors

**全局参数**（AISC PDF Page 2-3）:
- 5个安全/抗力系数（默认值按规范给定）
- 截面分类依据 TABLE B4.1（受压构件的宽厚比限值）
- 二阶效应：一阶弯矩通过 B1 因子放大（360-05: C2.1b章; 360-10+: Appendix 8.2）
- 选项：非F2-F11截面是否按 F12.1 计算弯矩承载力
- 选项：计算时排除扭转对剪力校核的影响
- 选项：`Compute loads when needed` — 校核时临时计算载荷，可节省数据库内存

**限制**:
- 不覆盖连接设计
- 不覆盖正常使用极限状态设计
- 设计壁厚（B3章第12条）不自动处理

**JS 示例**:
```javascript
CapacityManager = CapacityManager();
var ccRun = CapacityManager.createRun(AiscLrfdRun());
ccRun.generalParameters.version = asLrfd2016;      // 15th Edition
ccRun.generalParameters.phiTension = 0.9;            // LRFD 抗力系数
ccRun.generalParameters.phiCompression = 0.9;
ccRun.generalParameters.computeLoadsWhenNeeded = true;
CapacityManager.runAll();
```

---

## 2. API RP 2A-WSD (21st Edition, Dec 2000)

**PDF**: `API_WSD.pdf` (24页) — "Implementation of API-WSD 21st edition" (User Manual Vol 4 App C1)

**支持的版本**:
- API WSD 2002 (含 Errata & Supplement 1, Dec 2002)
- API WSD 2005 (含 Errata & Supplement 2, 完全重写的节点校核章节 + Errata 3, Jan 2007)

**校核内容**:
- 圆柱构件（cylindrical members）容量校核
- 锥形过渡段（conical transitions）校核
- 管节点（tubular joints）校核

**关键选项**（API PDF Page 2-3）:

| 选项 | 说明 |
|------|------|
| Cap-end forces included | 轴力包含静水端部力效应（对应Wajac分析） |
| Individual brace to can end distance | 每根撑杆独立到加强段端部距离 vs 节点最小距离 |
| Tolerance Angle | 方位角容许偏差（默认5°），控制Y/K/X节点分类 |
| Joint validity range | **Use geometric limits**: 用实际/修正几何参数取较小值<br>**Use modified geometry**: 强制修改到有效范围内 |
| Joint Minimum Capacity | 方案1: 节点容量≥撑杆强度50%<br>方案2: 节点UC ≤ 撑杆UC × 85%（可自定义限制值和系数） |
| Required Chord Thickness | 当 UF>1 时迭代计算所需弦杆厚度（需给定增量步和最大厚度限） |

**JS 示例**:
```javascript
var ccRun = CapacityManager.createRun(ApiWsd2005Run());
ccRun.generalParameters.capEndForcesIncluded = true;
ccRun.generalParameters.toleranceAngle = 5 deg;
ccRun.generalParameters.jointValidityRange = jvrGeometricLimits;
ccRun.generalParameters.jointMinimumCapacity = jmcBraceStrength;
CapacityManager.runAll();
```

---

## 3. EN 1993-1-1 (Eurocode 3)

**PDF**: `EUROCODE.pdf` (15页) — "Implementation of EUROCODE – 1993-1-1"

**支持版本**: 2005版（含以下 Corrigenda）:
- NS-EN 1993-1-1:2005/AC:2009
- NS-EN 1993-1-1:2005/NA:2008/AC:2010
- NS-EN 1993-1-1:2005+A1:2014+NA:2015

**国家附录选项** (EUROCODE PDF Page 3):
- **Standard**: "neutral" 版本（无国家附录）
- **Norwegian National Annex**: 使用特定 γM0, γM1
- **Danish National Annex**: 可选 Normal Control 或 Stricter Control 两种严格程度

**关键参数**:
- 可自定义 `γM0` 和 `γM1`（选择国家附录后自动更新默认值）
- 交互因子计算方法：Method 1 (Annex A) 或 Method 2 (Annex B)，默认 Method 1
- 校核内容：截面临界校核 + 构件屈曲校核（isolated members only）

**JS 示例**:
```javascript
var ccRun = CapacityManager.createRun(EN199311Run());
ccRun.generalParameters.nationalAnnex = naNorwegian;
ccRun.generalParameters.gammaM0 = 1.05;
ccRun.generalParameters.gammaM1 = 1.05;
ccRun.generalParameters.interactionMethod = imMethod1;  // Annex A
CapacityManager.runAll();
```

---

## 4. ISO 19902:2007 & ISO 19902:2020

**PDF**: `ISO.pdf` (46页) — "Implementation of ISO 19902 1st and 2nd edition"

**支持版本**:
- 1st Edition, 1 Dec 2007 (含 Amendment 1, 2013)
- 2nd Edition, 2020

**校核内容** (ISO PDF Page 1-3):
- 管状构件 (chapter 13 - Strength of tubular members)
- 锥形过渡段 (conical transitions)
- 管节点 (chapter 14 - Strength of tubular joints)
- **非管状构件自动回退使用 EN 1993-1-1:2005**（特别重要！）
- D/t > 120 时回退使用 DNVGL-RP-C202 (2020版: D/t > 0.2E/Fy)

**关键选项**:

| 选项 | 说明 |
|------|------|
| Cap-end forces included | 包含静水端部力 |
| Use Comm. A.13 Axial Compression | 使用评注中的柱屈曲公式 (A.13.2-1) 和 (A.13.2-2) |
| Individual brace to can end distance | 独立撑杆距加强段端部距离 |
| Partial resistance factors | 构件: 5个抗力系数; 节点: 3个抗力系数 |
| 非管状构件系数 | 需用 ISO 19901-3 的建筑标准对应系数人工修正 |
| Min cut-off value for brace UF | 用于 Eq.(14.3-13) 的支杆利用率下限（默认0，即使用实际值） |
| C1, C2 factors | 表14.3-2默认值 |
| Joint validity range | 几何限制 vs 修正几何 |
| X-joint in tension Qu formula | ISO19902:2007 vs ISO19902:2020 公式 |

**JS 示例**:
```javascript
var ccRun = CapacityManager.createRun(ISO19902Run());
ccRun.generalParameters.edition = so2020;               // 2020版
ccRun.generalParameters.capEndForcesIncluded = true;
ccRun.generalParameters.jointValidityRange = jvrGeometricLimits;
// 5个构件抗力系数
ccRun.generalParameters.gammaR_Tension = 1.05;
ccRun.generalParameters.gammaR_Compression = 1.05;
ccRun.generalParameters.gammaR_Bending = 1.05;
ccRun.generalParameters.gammaR_HydroPressure = 1.05;
ccRun.generalParameters.gammaR_Shear = 1.05;
// 3个节点抗力系数
ccRun.generalParameters.gammaR_JointAxialTensionCompression = 1.30;
ccRun.generalParameters.gammaR_JointBending = 1.30;
ccRun.generalParameters.gammaR_JointPunchingShear = 1.30;
CapacityManager.runAll();
```

---

## 5. NORSOK N-004 (Rev.2, Oct 2004)

**PDF**: `NORSOK.pdf` (17页) — "Implementation of NORSOK N-004 rev.2, 2004"

**校核内容** (NORSOK PDF Page 1-3):
- 管状构件 (chapter 6.3)
- 管节点 (chapter 6.4)
- 锥形过渡段 (chapter 6.5)
- **非管状构件自动回退使用 EN 1993-1-1:2005**

**特殊选项**:
- **Method A / Method B**: Cap-end forces included = Method B（含Wajac端部力）
- **Comm. 6.3.3 Axial Compression**: 使用评注公式
- **Material factor**: 给定值≠1.15时按给定值；=1.15时按 6.3.7节计算
- **Joint check options**: 同API的参数设置（brace距离、容许角、有效范围）
- `Compute loads when needed`: 节省数据库内存
- **NORSOK 2013 版本**: 另有 `NORSOK2013.pdf` 支持

**JS 示例**:
```javascript
var ccRun = CapacityManager.createRun(NORSOKRun());
ccRun.generalParameters.capEndForcesIncluded = true;      // Method B
ccRun.generalParameters.materialFactor = 1.15;             // 按6.3.7自动计算
ccRun.generalParameters.individualBraceCanEndDistance = true;
ccRun.generalParameters.computeLoadsWhenNeeded = true;
CapacityManager.runAll();
```

---

## 6. Danish Standard DS 412 & DS 449

**PDF**: `DS.pdf` (44页) — "Danish Standard DS 412 And DS 449" (含完整翻译)

**概述**:
- **DS 412**: 钢结构使用规范 (1st Edition, April 1983)
- **DS 449**: 桩基海洋钢结构 (1st Edition, April 1983 + Annexes Oct 1983)
- **仅适用于圆管截面**（pipe cross sections only）
- 按安全等级（Safety Classes）分组预设部分系数

**特点**:
- DS 412 校核 von Mises 应力
- DS 449 校核管状构件 + 管节点（含 Annex D 稳定性条件）
- 预定义的 fy（屈服应力）和 fu（极限抗拉强度）从规范获取
- 安全等级分 Normal / High / Very High 三级

---

## 7. CSR (Common Structural Rules)

两种校核模式：

| 规范 | JS 类 | 适用船型 |
|------|-------|---------|
| CSR Bulk | `CSRBulkRun` | 散货船 (Bulk Carriers) |
| CSR Tanker | `CSRTankRun` | 油船 (Oil Tankers) |
| CSR BC&OT Panel | `PanelCodeCheckRun` | 散货船/油船板格校核 |

板格校核详情 → 见 `panel_code_check.md`

---

## 8. Earthquake RSA Code Check

**PDF**: `EarthquakeRSA.pdf` (11页) — "Earthquake Response Spectrum Analysis and Code Check" (新增于 GeniE V8.8)

**功能概述**:
- 基于外部模态结果文件（Sestra .sin/.rso）的响应谱分析后处理
- 模态响应组合方法（SRSS / CQC）
- 方向响应组合（100/40 规则等）
- Basaload 估算（有效惯性力法）
- 地震载荷组合（叠加操作载荷）
- 管节点最小容量校核

**输入文件要求**:
- 外部模态结果文件（含模态位移/应力）
- EarthquakeRSA 输入文件（定义谱参数、阻尼、组合方式）

---

## 9. API LRFD

**PDF**: `API_LRFD.pdf` — LRFD 版本，校核概念同 WSD 但使用抗力系数体系。

---

## 10. DNV Implementation Notes (特殊功能)

**PDF**:
- `DNVS_Implementation_of_BS-sections.xls` — BS (British Standard) 截面在 DNV 系统中的实现对照表
- `Notes_on_ISO_Critical_Joints.pdf` — ISO 关键节点校核的补充说明（包括 Critial Joint 定义和UF处理）
- `Complete_CmDoc.pdf` — 完整的命令行/JS API 文档

---

## 快速对照：Choose Standard → JS Class

| 规范 | JS 类 | PDF文件 | PDF实际页数 |
|------|-------|---------|:---:|
| AISC 360-16 LRFD | `AiscLrfdRun` | AISC.pdf | 14 |
| AISC 360-16 ASD | `AiscAsdRun` | AISC.pdf | 14 |
| API WSD 2005 | `ApiWsd2005Run` | API_WSD.pdf | 24 |
| API LRFD | `ApiLrfdRun` | API_LRFD.pdf | - |
| EN 1993-1-1 | `EN199311Run` | EUROCODE.pdf | 15 |
| ISO 19902:2007 | `ISO19902Run` (edition=2007) | ISO.pdf | 46 |
| ISO 19902:2020 | `ISO19902Run` (edition=2020) | ISO.pdf | 46 |
| NORSOK N-004 Rev.2 | `NORSOKRun` | NORSOK.pdf | 17 |
| NORSOK N-004:2013 | `NORSOK2013Run` | NORSOK2013.pdf | - |
| DS 412/449 | `DSRun` | DS.pdf | 44 |
| CSR Bulk | `CSRBulkRun` | - | - |
| CSR Tanker | `CSRTankRun` | - | - |
| CSR BC&OT Panel | `PanelCodeCheckRun` | - | - |

> **注**: ISO 对于非管状构件、NORSOK 对于非管状构件都会**自动回退**使用 EN 1993-1-1。API 则仅适用于圆管构件（接头校核也仅含管节点）。
