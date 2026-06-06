# GeniE 模型分析前检查清单 (Pre-Flight Model Checklist)

在运行 GeniE 分析之前，必须逐项检查以下清单。任何未通过的项目都可能导致分析失败或产生错误结果。

---

## 1. 兼容性规则 (Compatibility Rules)

- [ ] **GeniE 版本兼容性已设置**  
  ```javascript
  GenieRules.Compatibility.version = "V8.8-08";
  ```
  必须在脚本最顶部（所有材料/截面定义之前）执行。

- [ ] **公差建模已启用**  
  ```javascript
  GenieRules.Tolerances.useTolerantModelling = true;
  GenieRules.Tolerances.angleTolerance = 2 deg;
  ```

---

## 2. 默认属性检查 (Defaults Assignment)

- [ ] **所有梁已赋予截面**  
  确保每个 `StraightBeam(Point, Point)` 返回的对象都设置了 `.section` 属性。

- [ ] **所有板/壳已赋予材料和厚度**  
  确保每个 `Plate()` / `SkinCurves()` / `CoverCurves()` 对象的 `.thickness` 和 `.material` 属性已设置。

- [ ] **默认材料已正确设置**  
  ```javascript
  S355.setDefault(Material);
  ```

- [ ] **板厚方向正确**  
  使用 `EndOn(Thickness(...))` 或 `BeginOn(...)` 指定厚度偏移方向。

- **在 GeniE 界面中验证**: 属性面板中检查是否有红色高亮（未定义属性）。

---

## 3. GuidePlane 设置检查

- [ ] **GuidePlane 原点位置正确**  
  确认每个工作平面的原点在预期的空间位置。

- [ ] **工作平面法向正确**  
  `GuidePlane(Point(x,y,z), Vector3d(nx,ny,nz))` 法向量指向预期方向。

- [ ] **GuidePlane 设置活跃工作平面**  
  在 GeniE 界面双击 GP 名称将其设为当前工作平面。

---

## 4. 拓扑检查 (Topology)

- [ ] **SimplifyTopology() 已调用**  
  必须在网格划分之前执行，用于合并重合节点、清理悬挂点。
  ```javascript
  SimplifyTopology();
  ```

- [ ] **梁端点正确连接到节点**  
  撑杆端点应与弦杆交于同一点，无微小间隙。在界面中使用 `Display → Snap To Points` 检查。

- [ ] **板边与梁边对齐**  
  板与梁的交界处节点应共位。共享 GuidePlane 有助于确保对齐。

- [ ] **无异常短边**  
  长度小于公差 10 倍的边可能导致网格划分失败。检查 `GenieRules.Tolerances.angleTolerance` 设置。

---

## 5. 边界条件检查 (Boundary Conditions)

- [ ] **刚体运动已充分约束**  
  模型不得存在未约束的刚体运动自由度（6 个: TX, TY, TZ, RX, RY, RZ）。

- [ ] **约束位置在实际支撑点**  
  例如: 导管架底部桩基点、甲板支撑柱底部。

- [ ] **正确的 API 语法**  
  ```javascript
  sp = SupportPoint(Point(x, y, z));
  sp.boundary = BoundaryCondition(Fixed, Fixed, Fixed, Free, Free, Free);
  ```

- [ ] **铰接 vs 固接选择正确**  
  - 桩基通常为铰接 (TX=TY=TZ=Fixed, RX=RY=RZ=Free) 或考虑桩-土相互作用
  - 甲板固定支座通常为固接 (全部 6 个自由度 = Fixed)
  - 半潜对称模型只需 3 点约束 (参照 A7 教程)

- [ ] **对称边界条件正确**  
  1/2 或 1/4 模型需检查对称面法向位移约束和切向转动约束。

**常见约束配置:**
| 场景 | TX | TY | TZ | RX | RY | RZ |
|------|----|----|----|----|----|----|
| 海底铰接 | Fixed | Fixed | Fixed | Free | Free | Free |
| 海底固接 | Fixed | Fixed | Fixed | Fixed | Fixed | Fixed |
| XY 面对称 | Fixed | Free | Free | Free | Fixed | Fixed |
| YZ 面对称 | Free | Fixed | Free | Fixed | Free | Fixed |
| XZ 面对称 | Free | Free | Fixed | Fixed | Fixed | Free |

---

## 6. 网格密度检查 (Mesh Densities)

- [ ] **全局 MeshDensity 已定义并分配**
  ```javascript
  md = MeshDensity();
  md.elementLength = 0.5 m;
  ```

- [ ] **全局单元尺寸合理**
  - 整体分析: 0.3-0.8 m
  - 局部精细: 0.05-0.15 m
  - 管节点疲劳: 0.015-0.025 m (热点区域)

- [ ] **局部细化区域已定义**  
  在高应力梯度区域（节点、开孔、支座）使用额外的 `MeshDensity` 对象并分配至 NamedSet。

- [ ] **板网格使用 Quad 优先**  
  ```javascript
  GenieRules.Meshing.preferredMesher = "QuadSurf";
  ```

---

## 7. 载荷工况检查 (Load Cases)

- [ ] **LoadCase 参数正确**  
  `LoadCase(ax, ay, az)` 中，自重用 `LoadCase(0, 0, -9.81)` 表示 Z 向重力。

- [ ] **LoadCase 名称有意义且不重复**

- [ ] **设备载荷位置正确**  
  ```javascript
  LC.placeAtPoint(Equipment, Point(x, y, z));
  ```
  确认 Equipment 放置在结构正确的甲板高度上。

- [ ] **水动力工况已配置**（如适用）  
  - `DummyHydroLoadCase(WetSurface)` 用于 Wadam 湿表面识别
  - 波高/周期/方向与水动力分析一致

---

## 8. 载荷组合检查 (Load Combinations)

- [ ] **所有需要的载荷组合已创建**
  ```javascript
  Comb1 = LoadCombination();
  Comb1.addCase(LCGrav, 1.0);
  Comb1.addCase(LCOper, 1.5);
  ```

- [ ] **载荷分项系数符合设计规范**
  | 极限状态 | γ_G (永久) | γ_Q (活载) | γ_E (环境) |
  |---------|:--------:|:--------:|:--------:|
  | ULS-a   | 1.30     | 1.50     | 0.70     |
  | ULS-b   | 1.00     | 0.70     | 1.35     |
  | SLS     | 1.00     | 1.00     | 1.00     |

- [ ] **设计工况已设置**（如需要 Code Check）
  ```javascript
  Comb1.designCondition = lcStorm;  // 指定为设计工况
  ```

- [ ] **载荷组合方向相容**  
  不应在同一组合中合并互斥的风向/波向。

---

## 9. 分析步骤结构 (Analysis Structure)

- [ ] **Analysis 已创建并设为 Active**
  ```javascript
  Analysis1 = Analysis(true);
  Analysis1.add(MeshActivity());
  Analysis1.add(LinearAnalysis());
  Analysis1.setActive();
  ```

- [ ] **MeshActivity 在 LinearAnalysis 之前**  
  网格划分必须先于求解。

- [ ] **Model 类型正确**（梁/壳/梁板混合）

- [ ] **可选: LoadResults 活动已添加**
  ```javascript
  Analysis1.add(LoadResultsActivity());
  ```

---

## 10. 命名唯一性 (Name Uniqueness)

- [ ] **无重复的构件名称**

- [ ] **无重复的载荷工况/组合名称**

- [ ] **无重复的边界条件名称**

- [ ] **无重复的 Activity/MeshDensity 名称**  
  （GeniE 会在重复命名时自动给警告）

---

## 快速自检脚本

将此脚本追加到模型脚本末尾，运行于分析之前:

```javascript
// ===== 分析前快速自检 =====
print("===== 分析前检查 =====");

// 1. 兼容性
GenieRules.Compatibility.version = "V8.8-08";
print("[OK] GeniE V8.8-08 兼容模式已设置");

// 2. 拓扑
SimplifyTopology();
print("[OK] 拓扑已简化");

// 3. 检查已知问题信号
print("[INFO] 检查界面属性面板中是否有红色高亮项");
print("[INFO] 检查所有 Beam 的 .section 属性是否已设置");
print("[INFO] 检查所有 Plate/SkinCurves 的 .thickness + .material 是否已设置");

print("===== 检查完成, 可以执行 Analysis.execute() =====");
```

---

## 常见错误与解决

| 错误信息 | 原因 | 解决方法 |
|---------|------|---------|
| Singular stiffness matrix | 刚体运动未约束 | 检查 BC, 约束全部 6 个刚体自由度 |
| Mesh generation failed | 几何缺陷/短边 | SimplifyTopology(), 增大 angleTolerance |
| No section assigned | 梁缺少截面 | 为所有 Beam 赋予 `.section` 属性 |
| Load case not found | 载荷工况未加入活动的 Analysis | Analysis 设置前确认所有 LoadCase 已定义 |
| Negative plate thickness | 板厚度未定义或为负 | 设置 `plate.thickness = Thickness(正值)` |
| Red highlight in Property panel | 构件缺少属性 | 逐个检查红色标记构件，补充缺失属性 |

---

> **SESAM 源**: `GeniE V8.8-08 Help\Tutorials\` + `Help\jscript\SupportPoint.html` + `Help\GuidingDocuments\Mesh_Guidance.pdf`
