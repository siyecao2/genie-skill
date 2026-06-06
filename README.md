# GeniE Skill — SESAM 海洋工程结构建模专家

GeniE 结构建模与有限元分析技能包，针对 **DNV SESAM GeniE V8.8-08**，覆盖导管架、上部组块、半潜平台、自升式平台等海洋工程结构的设计、建模、分析与规范校核全流程。

## 适用场景

- 导管架 (Jacket) 平台建模与桩土分析
- 上部组块 (Topside) 甲板结构设计与设备布置
- 半潜式平台 (Semisub) 浮筒/立柱/撑杆建模
- 自升式平台 (Jack-up) 桩腿与船体结构
- 起重机基座 (Crane Pedestal) 蒙皮曲面建模
- 管节点 (Tubular Joint) 梁/壳详细建模
- 波浪/风/流/设备/舱室载荷施加
- 线性静力 / 特征值屈曲 / 动力 / 地震分析
- AISC / API / EN1993 / ISO19902 / Norsok / DS / CSR 规范校核
- SESAM 模块间工作流（GeniE → Sestra → Xtract / GeniE → Wadam / Wajac → Splice）

## 目录结构

```
genie-skill/
├── SKILL.md                      # 核心技能文件（主入口，含子文件导航）
├── README.md                     # 本文件
├── snippets/                     # 代码片段（13 个）
│   ├── materials_and_sections.md # 材料与截面定义（Material/PipeSection/ISection 等）
│   ├── guiding_geometry.md       # 引导几何（GuidePlane/GuideLine/GuideSpline 等）
│   ├── beam_modeling.md          # 梁建模（StraightBeam/CurveOffset/BeamOrientation）
│   ├── plate_modeling.md         # 板壳建模（Plate/SkinCurves/Sweep）
│   ├── boundary_conditions.md    # 边界条件（SupportPoint/SupportCurve/RigidLink）
│   ├── loads.md                  # 载荷定义（LoadCase/PointLoad/LineLoad/SurfaceLoad）
│   ├── analysis.md                # 分析设置（MeshActivity/LinearAnalysis/LoadResults）
│   ├── code_check.md             # 规范校核（CapacityManager/Member/Joint）
│   ├── meshing.md                # 网格控制（MeshDensity/ElementType/MeshRefinement）
│   ├── model_transforms.md       # 模型变换（ModelTransformer/Translate/Rotate/Mirror）
│   ├── advanced_beam.md          # 高级梁建模（变截面/管节点/偏心/铰接）
│   ├── compartment_loads.md      # 舱室载荷（Compartment/液舱/进水）
│   └── freeform_shells.md        # 自由曲面（蒙皮/Sweep/放样）
├── templates/                    # 完整建模模板（6 个）
│   ├── jacket_model.md           # 导管架平台
│   ├── topside_model.md          # 上部组块
│   ├── semisub_model.md          # 半潜式平台
│   ├── crane_pedestal.md         # 起重机基座
│   ├── tubular_joint.md          # 管节点
│   └── wind_loads.md             # 风载荷
├── reference/                    # 参考文档（16 个）
│   ├── js_api_reference.md       # GeniE JS API 完整类参考（700+ 类）
│   ├── js_examples.md            # Tutorial JS 脚本示例集
│   ├── sesam_workflow.md         # SESAM 模块间工作流
│   ├── section_library.md        # 型钢截面库速查
│   ├── libraries_catalog.md      # 型钢库/材料库/KZY/XML 截面命名
│   ├── code_check_standards.md   # 规范校核标准详解
│   ├── panel_code_check.md       # 板格校核/CSR BC&OT
│   ├── hydro_properties.md       # 水动力属性（Morison/AirDrag/MarineGrowth）
│   ├── parametric_modeling.md    # 参数化建模（JScript编程/Excel交互）
│   ├── shell_fatigue.md          # 管节点壳疲劳（ConvertJoints/DNV RP-C203）
│   ├── tutorial_index.md         # 教程索引（B1-B12 基础 / A1-A16 高级）
│   ├── wizard_templates.md       # Excel 建模向导/Deck Wizard/Jacket Wizard
│   ├── genie_rules.md            # GeniE 兼容性规则
│   ├── report_generation.md      # 报告生成（Word/Excel）
│   ├── best_practices.md         # 最佳实践
│   ├── common_errors.md          # 常见错误排查
│   └── unit_systems.md           # 单位系统与换算
└── workflows/                    # 工作流指南（3 个）
    ├── standard_workflow.md      # 标准分析流程（6 阶段法）
    ├── model_checklist.md        # 建模检查清单
    └── result_interpretation.md  # 结果解读指南
```

## 核心改进（2026-05-21）

基于 GeniE V8.8-08 Help 文档（UserDocumentation / Tutorials / GuidingDocuments / jscript API）重写：

| 模块 | 改进内容 |
|------|---------|
| snippets/* | 13 个代码片段全部使用真实 GeniE JS API 语法，无虚构函数 |
| templates/* | 6 个模板全部用 `MaterialLinear/PipeSection/StraightBeam/SkinCurves/SweepCurve` 等真实 API 重写 |
| reference/js_api_reference.md | **新建**：700+ 类 API 完整参考 |
| reference/sesam_workflow.md | **新建**：GeniE→Sestra→Wadam/Wajac→Splice 跨模块工作流 |
| reference/section_library.md | **新建**：型钢截面库查询（AISC/NSF_EN/BS4） |
| reference/tutorial_index.md | **新建**：B1-B12 基础 + A1-A16 高级教程索引 |
| workflows/standard_workflow.md | **重写**：消除 22 个虚构 API，全部替换为真实 GeniE JS |
| workflows/model_checklist.md | **更新**：适配 V8.8 GenieRules 兼容性规范 |

## 使用方法

1. **Skill 激活时**：AI 自动读取 SKILL.md，按子文件导航表按需读取对应文件
2. **代码生成**：从 snippets/ 获取代码片段，从 templates/ 获取完整模板
3. **参数查询**：查 SKILL.md 内嵌材料参数速查表、截面类型速查表、教程速查表
4. **API 查询**：查 reference/js_api_reference.md 获取 700+ 类 API 完整参考
5. **错误排查**：查 reference/common_errors.md 或 SKILL.md 常见错误码速查
6. **建模检查**：查 workflows/model_checklist.md

## MCP 工具集成

本 Skill 可与 [simulation-mcp](https://github.com/siyecao2/simulation-mcp) 的 genie 工具配合使用：

| MCP 工具 | 功能 |
|----------|------|
| `genie_get_sesam_info` | 获取 SESAM 安装信息概览 |
| `genie_list_profiles` | 列出型钢截面库 |
| `genie_get_api_help` | 查询 JS API 类文档 |
| `genie_create_project` | 创建 GeniE 项目目录与模板 |
| `genie_get_model_info` | 获取模型信息（解析 .gnx/.js） |
| `genie_parse_gnx` | 解析 GNX XML 模型文件 |
| `genie_parse_js` | 解析 GeniE JS 脚本结构 |
| `genie_get_script_template` | 获取脚本模板（beam/plate/jacket） |
| `genie_run_script` | 运行 GeniE 脚本 |
| `genie_generate_structure` | 自动生成结构脚本 |
| `genie_search_help` | 全文搜索 GeniE Help 文档 |

## 本地 SESAM 模块

| 模块 | 版本 | 用途 |
|------|------|------|
| GeniE | V8.8-08 | 结构建模（核心） |
| Sestra | V10.17-02 | 有限元求解 |
| Wadam | V10.3-02 | 波浪衍射/辐射 |
| Wajac | V7.10-01 | Morison 波浪载荷 |
| Splice | V8.1-00 | 桩土相互作用 |
| Usfos | V9.0-00 | 非线性倒塌 |
| HydroD | V4.10-01 / V7.0-01 | 水动力分析 |
| Xtract | (内嵌) | 结果后处理 |
| Postresp | V7.2-03 | 响应后处理 |
| Mimosa | V6.3-10 | 系泊分析 |

## 版本

- **GeniE 版本**：V8.8-08（2023年11月发布）
- **语法**：JavaScript 脚本严格遵循 GeniE V8.x 接口规范
- **兼容环境**：SESAM 2023 集成环境
- **单位系统**：SI（N, m, kg, Pa, deg）
- **最后更新**：2026-05-21
