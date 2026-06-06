# GeniE 模型分析前检查清单 (Pre-Flight Model Checklist)

在运行 GeniE 分析之前，必须逐项检查以下清单。任何未通过的项目都可能导致分析失败或产生错误结果。

---

## 1. 兼容性规则 (Compatibility Rules)

- [ ] **兼容模式已设置**  
  `gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);` 必须在脚本最顶部执行
  
- [ ] **高级兼容规则已启用**  
  `gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);` 紧跟兼容模式之后

**验证代码:**
```javascript
if (!gCOMPAT.IsSesamCompatibilityModeActive()) {
    throw new Error("兼容模式未激活! 请在脚本开头设置兼容性。");
}
```

---

## 2. 默认属性检查 (Defaults Assignment)

- [ ] **所有梁已赋予截面**  
  确保每个 Beam 对象都通过 `Beam.Create(line, section)` 或后续 SetSection 获得了截面

- [ ] **所有板/壳已赋予材料和厚度**  
  确保每个 Plate/SkinCurves 对象的 thickness 和 material 属性已设置

- [ ] **无遗漏构件的默认属性**  
  使用 GeniE 界面检查属性面板中是否有红色高亮的未定义属性

**验证代码:**
```javascript
var bodies = GetAllBodies();
for (var i = 0; i < bodies.length; i++) {
    var b = bodies[i];
    if (b.type == "Beam" && b.section == null) {
        print("错误: 梁 " + b.name + " 未赋予截面!");
    }
    if ((b.type == "Plate" || b.type == "SkinCurves") && b.thickness <= 0) {
        print("错误: " + b.type + " " + b.name + " 未定义厚度!");
    }
}
```

---

## 3. GuidePlane 设置检查

- [ ] **所有 GuidePlane 的 snapmode = true**  
  如果 snapmode 未启用，在平面上创建的点可能不会精确落在平面上

- [ ] **GuidePlane 原点位置正确**  
  确认每个工作平面的 Origin 在预期的空间位置

- [ ] **工作平面法向正确**  
  特别是通过旋转创建的平面，确认法向量方向

**验证代码:**
```javascript
var allGPs = GuidePlane.GetAll();
for (var i = 0; i < allGPs.length; i++) {
    var gp = allGPs[i];
    if (!gp.snapmode) {
        print("修复: 设置 " + gp.name + ".snapmode = true");
        gp.snapmode = true;
    }
}
```

---

## 4. 拓扑检查 (Topology)

- [ ] **SimplifyTopology() 已调用**  
  必须在网格划分之前执行，用于合并重合节点、清理悬挂点

- [ ] **无异常的短边 (Short Edges)**  
  长度小于公差 10 倍的边可能导致网格划分失败

- [ ] **梁端点正确连接到节点**  
  撑杆端点应与弦杆交于同一点，无微小间隙

- [ ] **板边与梁边对齐**  
  板与梁的交界处节点应共位

**验证代码:**
```javascript
SimplifyTopology();
var freeEdges = CheckFreeEdges();
if (freeEdges.length > 0) {
    print("警告: 发现 " + freeEdges.length + " 条自由边, 可能存在未连接构件!");
}
```

---

## 5. 边界条件检查 (Boundary Conditions)

- [ ] **刚体运动已充分约束**  
  模型不得存在未约束的刚体运动自由度 (6个: UX, UY, UZ, RX, RY, RZ)

- [ ] **约束位置在实际支撑点**  
  例如: 导管架底部桩基点、甲板支撑柱底部

- [ ] **铰接 vs 固接选择正确**  
  - 桩基通常为铰接 (UX=UY=UZ=0, RX=RY=RZ=free) 或考虑桩-土相互作用
  - 甲板固定支座通常为固接 (全部 6 个自由度 = 0)

- [ ] **对称边界条件正确**  
  1/2 或 1/4 模型需检查对称面法向位移约束和切向转动约束

**常见约束配置:**
| 场景 | UX | UY | UZ | RX | RY | RZ |
|------|----|----|----|----|----|----|
| 海底铰接 | 0 | 0 | 0 | free | free | free |
| 海底固接 | 0 | 0 | 0 | 0 | 0 | 0 |
| XY面对称 | 0 | free | free | free | 0 | 0 |
| YZ面对称 | free | 0 | free | 0 | free | 0 |
| XZ面对称 | free | free | 0 | 0 | 0 | free |

---

## 6. 网格密度检查 (Mesh Densities)

- [ ] **所有物体已加入 MeshSet**  
  `meshSet.AddAllBodies()` 或逐个添加

- [ ] **全局单元尺寸合理**  
  - 整体分析: 0.3-0.8m
  - 局部精细: 0.05-0.15m
  - 管节点: 0.015-0.05m (热点区域)

- [ ] **局部细化区域已定义**  
  在高应力梯度区域 (节点、开孔、支座) 设置细化

- [ ] **最小单元质量 ≥ 0.3**  
  低于 0.3 的单元可能导致收敛问题

- [ ] **板使用四边形网格 (推荐)**  
  `meshCtrl.SetPlateQuadMesh(true);`

---

## 7. 载荷链接检查 (Load Case-Activity Links)

- [ ] **每个分析中包含的载荷工况都有明确的活动关联**

- [ ] **LoadCase 编号 (number) 不重复**  
  永久载荷: 100-199, 活载荷: 200-299, 环境载荷: 300-399

- [ ] **自重方向正确**  
  `ActivateSelfWeight(gx, gy, gz, factor)` 中 gz 通常为 -1 (Z轴向下)

- [ ] **集中力/弯矩施加在正确的节点/构件上**

- [ ] **压力载荷方向正确**  
  正压力指向板/壳的内部还是外部?

**验证代码:**
```javascript
var loadCaseNumbers = [];
// ... 收集所有 LoadCase number
var hasDuplicates = loadCaseNumbers.length !== new Set(loadCaseNumbers).size;
if (hasDuplicates) {
    print("错误: 存在重复的载荷工况编号!");
}
```

---

## 8. 载荷组合检查 (Load Combinations)

- [ ] **所有需要的载荷组合已创建**

- [ ] **载荷分项系数符合设计规范**  
  检查 γ_G, γ_Q, γ_E 系数是否与规范一致

- [ ] **载荷组合类型正确** (ULS / SLS / ALS / PLS)

- [ ] **载荷组合编号不重复**  
  ULS: 1000-1999, SLS: 2000-2999, ALS: 3000-3999

- [ ] **参与的载荷工况方向相容**  
  不应在同一个组合中合并互斥的风向

---

## 9. 分析步骤结构 (Step Structure)

- [ ] **Activity 已创建并与 Analysis 关联**

- [ ] **Step 顺序合理**  
  通常: 静力分析 → 模态分析 → 动力分析 (如有)

- [ ] **Solver 类型正确**  
  LinearStaticSolver / ModalSolver / DynamicSolver

---

## 10. 命名唯一性 (Name Uniqueness)

- [ ] **无重复的构件名称**

- [ ] **无重复的载荷工况/组合名称**

- [ ] **无重复的边界条件名称**

- [ ] **无重复的 Activity/MeshSet/Solver 名称**

**验证代码:**
```javascript
function checkDuplicateNames(objects, category) {
    var names = objects.map(function(o) { return o.name; });
    var seen = {};
    for (var i = 0; i < names.length; i++) {
        if (seen[names[i]]) {
            print("重复名称: " + category + " '" + names[i] + "' 出现多次!");
        }
        seen[names[i]] = true;
    }
}
```

---

## 快速自检脚本

将此脚本追加到模型脚本末尾，运行于 `Solve()` 之前:

```javascript
// ===== 分析前快速自检 =====
print("===== 分析前检查 =====");

// 1. 兼容性
if (!gCOMPAT.IsSesamCompatibilityModeActive()) {
    print("[FAIL] 兼容模式未激活!");
} else {
    print("[OK] 兼容模式已激活");
}

// 2. 边界条件
var bcCount = BC.GetAll().length;
print("[INFO] 边界条件数量: " + bcCount);

// 3. 网格
var bodies = GetAllBodies();
print("[INFO] 结构体数量: " + bodies.length);

// 4. 载荷
var lcCount = LoadCase.GetAll().length;
var lcbCount = LoadCombination.GetAll().length;
print("[INFO] 载荷工况: " + lcCount + ", 载荷组合: " + lcbCount);

// 5. 拓扑
SimplifyTopology();
print("[OK] 拓扑已简化");

print("===== 检查完成 =====");
```

---

## 常见错误与解决

| 错误信息 | 原因 | 解决方法 |
|---------|------|---------|
| Singular stiffness matrix | 刚体运动未约束 | 检查 BC, 约束全部 6 个刚体自由度 |
| Mesh generation failed | 几何缺陷/短边 | SimplifyTopology(), 增大公差 |
| No section assigned | 梁缺少截面 | 为所有 Beam 赋予 Section |
| Load case not found | 载荷工况未加入 Solver | Solver.AddLoadCase(lc) |
| Negative plate thickness | 板厚度未定义或为负 | 设置 plate.thickness = 正值 |
