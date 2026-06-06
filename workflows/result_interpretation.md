# GeniE 分析结果解读指南 (Result Interpretation Guide)

如何解读 GeniE 有限元分析结果，包括位移、应力、规范校核和屈曲评估。

---

## 1. 位移结果解读 (Displacement Results)

### 1.1 最大位移提取

```javascript
// 获取指定分析的最大位移
var maxDisp = Result.GetMaxDisplacement("Analysis_Main", "Solve_ULS");
print("最大总位移: " + (maxDisp.value * 1000).toFixed(2) + " mm");
print("发生位置 (节点): " + maxDisp.node);
print("位移分量: UX=" + (maxDisp.ux*1000).toFixed(2) + "mm, "
    + "UY=" + (maxDisp.uy*1000).toFixed(2) + "mm, "
    + "UZ=" + (maxDisp.uz*1000).toFixed(2) + "mm");
```

### 1.2 位移验收准则

| 结构类型 | 限制 | 准则来源 |
|---------|------|---------|
| 导管架顶部水平位移 | < H/300 (H=结构总高) | API RP 2A |
| 上部组块甲板挠度 | < L/300 (L=跨度) | ISO 19902 |
| 起重机基座顶部位移 | < H/200 | 制造商要求 |
| 平台整体侧移 | < H/250 | DNV-OS-C101 |

```javascript
// 位移验收检查
function checkDisplacementLimit(maxDisp_m, criterion, limitDesc) {
    if (maxDisp_m < criterion) {
        print("[PASS] " + limitDesc + ": " + (maxDisp_m*1000).toFixed(1)
            + "mm < " + (criterion*1000).toFixed(1) + "mm");
    } else {
        print("[FAIL] " + limitDesc + ": " + (maxDisp_m*1000).toFixed(1)
            + "mm > " + (criterion*1000).toFixed(1) + "mm !!");
    }
}

// 示例: 60m 高导管架, 顶部水平位移限制 = 60/300 = 0.20m
checkDisplacementLimit(maxDisp.value, 60.0/300.0, "导管架顶部水平位移 H/300");
```

---

## 2. 应力结果解读 (Stress Results)

### 2.1 von Mises 应力

von Mises 应力是判断材料是否屈服的主要指标：

```javascript
var maxVM = Result.GetMaxVonMisesStress("Analysis_Main", "Solve_ULS");
print("最大von Mises应力: " + (maxVM.value / 1e6).toFixed(1) + " MPa");
print("位置: " + maxVM.bodyName + " @ (" + maxVM.x.toFixed(2) + ", " + maxVM.y.toFixed(2) + ", " + maxVM.z.toFixed(2) + ")");

// 利用率 UF = von Mises应力 / (材料屈服强度 / 材料系数)
var fy = 355e6;      // S355 屈服强度
var gammaM = 1.15;   // 材料系数
var allowable = fy / gammaM;
var uf = maxVM.value / allowable;
print("von Mises 利用率: " + (uf * 100).toFixed(1) + "% (> 100% = 超出)");
```

### 2.2 主应力解读

```javascript
// 最大/最小主应力
var s1 = Result.GetMaxPrincipalStress(solverName);
var s3 = Result.GetMinPrincipalStress(solverName);
print("最大主应力 S1: " + (s1.value/1e6).toFixed(1) + " MPa");
print("最小主应力 S3: " + (s3.value/1e6).toFixed(1) + " MPa (负值=压缩)");

// 主应力比: |S3|/S1 > 3 表示以压缩为主
```

### 2.3 应力集中区域分析

高应力区域通常出现在：

| 位置 | 典型 UF | 原因 |
|------|---------|------|
| 管节点相贯线 | 0.6-0.9 | 几何应力集中 (SCF) |
| 板-梁连接处 | 0.4-0.7 | 拉力场/压力场转换 |
| 柱-甲板交接 | 0.5-0.8 | 刚度突变 |
| 开孔边缘 | 0.6-1.0 | 孔边应力集中 |
| 支座区域 | 0.3-0.6 | 反力集中 |

---

## 3. 规范校核结果解读 (Code Check Results)

### 3.1 利用率因子 (UF) 解读

UF (Utilization Factor) = 实际效应 / 容许效应

| UF 范围 | 状态 | 措施 |
|---------|------|------|
| UF < 0.5 | 低利用 (保守) | 可优化截面 |
| 0.5 ≤ UF < 0.8 | 正常 | 设计合理 |
| 0.8 ≤ UF < 0.95 | 接近极限 | 关注疲劳和局部效应 |
| 0.95 ≤ UF < 1.0 | 高利用 | 需详细校核 |
| UF ≥ 1.0 | **超限** | **必须修改设计!** |

### 3.2 梁构件校核结果提取

```javascript
var ccResults = BeamCodeCheck.GetResults("CC_Beams");

// 列出所有超限构件
var overstressed = ccResults.filter(function(r) {
    return r.maxUF >= 1.0;
});

if (overstressed.length > 0) {
    print("===== 超限构件 =====");
    for (var i = 0; i < overstressed.length; i++) {
        var o = overstressed[i];
        print(o.beamName + ": UF=" + o.maxUF.toFixed(3)
            + " (" + o.failureMode + ")"
            + " @ 载荷组合 " + o.loadCombination);
    }
} else {
    print("所有构件均满足规范要求 (UF < 1.0)");
}

// 最高 UF 统计
var sortedByUF = ccResults.sort(function(a, b) { return b.maxUF - a.maxUF; });
print("\n===== UF 前10高 =====");
for (var j = 0; j < Math.min(10, sortedByUF.length); j++) {
    var s = sortedByUF[j];
    print((j+1) + ". " + s.beamName + ": UF=" + s.maxUF.toFixed(3)
        + " [" + s.failureMode + "]");
}
```

### 3.3 管节点校核结果 (Punching Shear)

```javascript
var jcResults = JointCapacityCheck.GetResults("JC_Tubular");

// 冲剪校核
for (var k = 0; k < jcResults.length; k++) {
    var jr = jcResults[k];
    print("节点: " + jr.jointName);
    print("  冲剪利用率: " + (jr.punchingShearUF * 100).toFixed(1) + "%");
    print("  弦杆壁承载 UF: " + (jr.chordWallBearingUF * 100).toFixed(1) + "%");
    print("  控制载荷组合: " + jr.governingLoadComb);
    
    // Qf 因子 (弦杆载荷效应)
    if (jr.Qf < 1.0) {
        print("  Qf=" + jr.Qf.toFixed(3) + " (弦杆受压, 节点承载力折减)");
    }
}
```

---

## 4. 屈曲评估 (Buckling Assessment)

### 4.1 梁屈曲校核

```javascript
// 线性屈曲分析
var bucklingSolver = LinearBucklingSolver();
bucklingSolver.name = "Solve_Buckling";
bucklingSolver.AddLoadCombination(lcULS);
bucklingSolver.SetNumModes(5);

// 获取屈曲特征值
var bucklingModes = Result.GetBucklingModes("Solve_Buckling");
for (var b = 0; b < bucklingModes.length; b++) {
    var mode = bucklingModes[b];
    var lambda = mode.eigenvalue;  // 屈曲载荷因子
    print("屈曲模态 " + (b+1) + ": λ=" + lambda.toFixed(2)
        + " (" + (lambda >= 1.0 ? "安全" : "屈曲!") + ")");
}

// 验收准则: 第一阶 λ > 1.0 (λ × 施加的载荷 = 屈曲载荷)
```

### 4.2 板屈曲检查

板屈曲（局部屈曲）在壳单元模型中可通过应力分布判断：

```javascript
// 检查板面内压缩应力
var plateCompressionRegions = Result.GetElementsWhere(
    "Analysis_Main", "Solve_ULS",
    "S3 < -200e6",  // 主压应力超过 200 MPa
    "Plate"
);
print("高压缩区域数 (板): " + plateCompressionRegions.length);
```

---

## 5. 梁内力/弯矩图解读 (Beam Force/Moment Diagrams)

```javascript
// 提取梁截面内力
function getBeamForces(beamName, loadCaseName) {
    var forces = Result.GetBeamSectionForces(beamName, loadCaseName);
    print("===== 梁 " + beamName + " 截面内力 =====");
    for (var i = 0; i < forces.length; i++) {
        var f = forces[i];
        print("位置 " + f.position.toFixed(2) + " (0=始端, 1=末端):");
        print("  N="  + (f.axial/1e3).toFixed(1) + "kN");
        print("  Vy=" + (f.shearY/1e3).toFixed(1) + "kN, Vz=" + (f.shearZ/1e3).toFixed(1) + "kN");
        print("  Mx=" + (f.torsion/1e3).toFixed(1) + "kNm (扭矩)");
        print("  My=" + (f.momentY/1e3).toFixed(1) + "kNm, Mz=" + (f.momentZ/1e3).toFixed(1) + "kNm");
    }
}

// 示例: 检查导管架腿底部的弯矩
// getBeamForces("LEG_A1", "ULS_Comb1");
```

---

## 6. 板应力云图解读 (Plate Stress Contours)

### 6.1 板应力分量

板应力关键分量：

| 分量 | 含义 | 关注场景 |
|------|------|---------|
| Smx | 面内膜应力 X方向 | 板面内拉伸/压缩 |
| Smy | 面内膜应力 Y方向 | 板面内拉伸/压缩 |
| Smxy | 面内剪应力 | 剪切面板 |
| Sbx | 弯曲应力 X方向 | 板弯曲 |
| Sby | 弯曲应力 Y方向 | 板弯曲 |
| Sbx+Smx | 顶面总应力 X方向 | 表面应力最大处 |

### 6.2 热点应力提取

```javascript
// 提取热点区域的应力
var hotSpots = [
    {name: "管节点焊趾", x: 0.0, y: 0.0, z: 0.0, radius: 0.5},
    {name: "开孔边缘",   x: 5.0, y: 3.0, z: 10.0, radius: 0.3}
];

for (var h = 0; h < hotSpots.length; h++) {
    var hs = hotSpots[h];
    var hotStress = Result.GetStressAtRegion(
        "Analysis_Main", "Solve_ULS",
        hs.x, hs.y, hs.z, hs.radius
    );
    print("热点 '" + hs.name + "': max VM=" + (hotStress.maxVM/1e6).toFixed(1) + " MPa");
    
    // 疲劳评估: 热点应力 × SCF
    // var nominalStress = hotStress.maxVM / SCF;
}
```

---

## 7. 报告生成 (Report Generation)

### 7.1 自动生成分析报告

```javascript
// 生成标准分析报告
// 导出到 Xtract 进行高级后处理
// Xtract 读取 Sestra .Rxx 结果文件
report.name = "Analysis_Report_ULS";
report.SetTitle("结构分析报告 - 极限状态");
report.SetAuthor(proj.author);
report.SetDate(new Date().toISOString());

// 添加章节
report.AddSection("1. 项目概述");
report.AddParagraph("项目名称: " + proj.name);
report.AddParagraph("分析类型: 线性静力分析");

report.AddSection("2. 位移结果");
report.AddParagraph("最大位移: " + (maxDisp.value * 1000).toFixed(2) + " mm");
report.AddParagraph("位移准则: H/300 = " + (60/300*1000).toFixed(1) + " mm");

report.AddSection("3. 应力结果");
report.AddParagraph("最大von Mises应力: " + (maxVM.value/1e6).toFixed(1) + " MPa");

report.AddSection("4. 规范校核");
report.AddParagraph("超限构件数: " + overstressed.length);

// 生成 PDF
// report.Generate("C:/Reports/ULS_Report.pdf");
```

---

## 8. 导出到 Xtract/Postresp (Advanced Post-processing)

```javascript
// 导出全套结果到 Xtract
Result.ExportToXtract(
    "Analysis_Main",
    "C:/Results/Analysis_Results.xtr"
);

// 导出节点位移 CSV
Result.ExportNodeDisplacementsCSV(
    "Analysis_Main",
    "Solve_ULS",
    "C:/Results/displacements.csv"
);

// 导出梁内力 CSV
Result.ExportBeamForcesCSV(
    "Analysis_Main",
    "Solve_ULS",
    "C:/Results/beam_forces.csv"
);

print("结果文件已导出。");
print("可使用 Xtract/Postresp 打开 .xtr 文件进行高级后处理。");
```

---

## 关键验收准则汇总

| 检查项 | 准则 | 规范来源 |
|--------|------|---------|
| 导管架水平位移 | < H/300 | API RP 2A §C.3.3 |
| 甲板挠度 | < L/360 | ISO 19902 §12.3 |
| 杆件应力 UF | < 1.00 | ISO 19902 §12.2 |
| 管节点冲剪 UF | < 1.00 | API RP 2A §C4.3 |
| 屈曲特征值 λ | > 1.00 | DNV-OS-C101 §6.4 |
| 疲劳寿命 | > 设计寿命 × FDF | DNV-RP-C203 |
| 桩基承载力 UF | < 1.00 | API RP 2A §6.3 |
