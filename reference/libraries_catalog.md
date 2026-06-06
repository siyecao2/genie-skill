# GeniE 型钢库与材料库

> 所有库文件位于 `Libraries\` 目录，可通过 `Edit → Properties → Section → Section Library` 导入。

---

## 型钢库文件

| 文件 | 标准 | 类型 | 用途 |
|------|------|------|------|
| **aisc_v3.kzy** | AISC v3 | KZY | 美国钢结构协会型钢库 (W/HSS/Channel/Angle/Pipe) |
| **NSF_EN.KZY** | NS-EN 10210/10219 | KZY | 挪威标准型钢库 (HE/Box/Pipe) — 教程B5使用 |
| **BS4_Sections_Part1_1993.xml** | BS 4 Part 1:1993 | XML | 英国标准型钢库 |
| **Material_library.xml** | - | XML | 材料属性库 |
| **anglebar.xml** | - | XML | 角钢截面数据库 |
| **flatbar.xml** | - | XML | 扁钢截面数据库 |
| **bulb.xml** | - | XML | 球扁钢(Holland Profile)截面库 |
| **tbar.xml** | - | XML | T型钢截面数据库 |

---

## 型钢库导入方式

### GUI 导入
```
Edit → Properties → Section 标签 → Create/Edit Section → Section Library 标签
→ 浏览选择 .KZY 或 .xml 文件
→ 筛选 Section Type / Name Match
→ 勾选 Selected → 只导入选中截面
```

### JS 导入
```javascript
// 从 Section Library 导入单个截面
HE400A = ISection(0.39 m, 0.3 m, 0.011 m, 0.019 m, 0.027 m);
HE400A.description = "NVS lib : HE 400 A NS-EN 10034";
HE400A.libraryGeneralSection = GeneralSection(
    0.0159 m^2,              // 面积
    1.9e-06 m^4,             // Iy
    0.0004507 m^4,           // Iz
    8.56e-05 m^4,            // Iv
    0 m^4,                   // Iyz
    0.0001045 m^3,           // Wy (top)
    0.00231 m^3,             // Wz (left)
    0.000571 m^3,            // Wv (top)
    0.007517 m^2,            // Shear Area Y
    0.003870 m^2,            // Shear Area Z
    0 m, -2.78e-17 m,        // Shear center Y, Z
    0.001281 m^3,            // Plastic section modulus Wy
    0.000433 m^3,            // Plastic section modulus Wz (left)
    0.00256 m^3,             // Plastic section modulus Wv
    0.000855 m^3             // Plastic section modulus Wz (right)
);
```

### 截面类型 JS 构造器

```javascript
// I 型钢
ISection(height, flangeWidth, webThk, flangeThk, radius)

// 箱型/方管
BoxSection(height, width, webThk, flangeThk)

// 管截面
PipeSection(diameter, thickness)

// 角钢
LSection(height, width, webThk, flangeThk)  // 不等边
BarSection(thickness, width)                // 扁钢

// 槽钢
ChannelSection(height, width, webThk, flangeThk)

// 非对称 I 型(T型钢)
UnsymISection(height, webThk, topFlangeThk, bottomFlangeThk, ..., topFlangeWidth, bottomFlangeWidth, ...)

// 锥段
ConeSection(dynamicThickness)  // 1=取大壁厚, 0=取小壁厚
```

---

## 材料库 (Material_library.xml)

材料定义模板：

```javascript
// 标准结构钢 (S355)
S355 = MaterialLinear(
    355000000 Pa,           // 屈服强度
    7850 kg/m^3,            // 密度
    2.1e+011 Pa,            // 杨氏模量
    0.3,                    // 泊松比
    1.2e-005 delC^-1,       // 热膨胀系数
    0.03 N*s/m              // 阻尼系数
);

// 低密度补偿材料 (厚板简化建模)
SuperMaterial = MaterialLinear(2.0E8, 1780, 2.1e+11 Pa, 0.3, 1.2e-05, 0.03);
```

### 常用材料等级

| 等级 | Yield (MPa) | ρ (kg/m³) | E (GPa) | ν |
|------|:----------:|:----------:|:-------:|:---:|
| S235 | 235 | 7850 | 210 | 0.3 |
| S275 | 255 | 7850 | 210 | 0.3 |
| S355 | 355 | 7850 | 210 | 0.3 |
| Steel (gen) | 200 | 7850 | 210 | 0.3 |

---

## 截面编号规则 (Frame.js 模式)

```javascript
// AISC 命名: W + 高度mm + X + 质量kg/m
W1000X350 = ISection(1.008, 0.302, 0.0211, 0.04);

// NVS 管命名: OD + 外径mm + X + 壁厚mm
OD457X40 = PipeSection(0.457, 0.04);

// NVS 箱型命名: HFRHS + 高x宽x壁厚mm
HFRHS500X300X20 = BoxSection(0.5, 0.3, 0.02, 0.02);

// 角钢命名: L + 高x宽x腹板厚x翼缘厚mm
Ixa_L250x90x10x15 = LSection(0.25, 0.09, 0.0105, 0.015);

// 槽钢命名: MC + 高mm + X + 质量kg/m
MC250X50 = ChannelSection(0.254, 0.104, 0.0146, 0.0146);

// T型钢命名: Tbar + 高x宽x腹板厚x翼缘厚mm
Tbar425x120x12x25 = UnsymISection(0.45, 0.012, 0.012, 0.006, 1e-6, 0.12, 0.06, 0.025);
```

---

## CSR 腐蚀余量脚本 (corr_add_to_gross_rev3_in.js)

位于 `Libraries\JS\corr_add_to_gross_rev3_in.js`，用于将净厚度FEM模型反算回毛厚度：

```javascript
var fac = 0.5;            // 腐蚀余量系数
var tres = 0.5 mm;        // 储备厚度

// 向上取整到0.5mm
function RoundUp0_5(numberToRound) {
    var result = Math.round(2*(numberToRound)+0.499)/2;
    return result;
}
// 命名: 原板厚名_Ca_2_3 (表示2.3mm腐蚀余量)
```

---

## 关键文件路径

```
Libraries\
├── aisc_v3.kzy                 # AISC 型钢库 (推荐用于海工结构)
├── NSF_EN.KZY                  # 挪威标准型钢库 (教程B5使用)
├── BS4_Sections_Part1_1993.xml # 英国标准
├── Material_library.xml         # 材料库
├── anglebar.xml / flatbar.xml  # 角钢/扁钢数据库
├── bulb.xml / tbar.xml          # 球扁钢/T型钢数据库
├── keyfile.txt                  # 库文件说明
└── JS\
    └── corr_add_to_gross_rev3_in.js  # CSR腐蚀余量脚本
```
