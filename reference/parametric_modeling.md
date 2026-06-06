# GeniE 参数化建模与 JScript 深度编程

> 基于 `Help\GuidingDocuments\Parametric\GeniE_parametric_models.pdf` (74页, Version 1 July 2009) 和 `Pile_Soil_Training_Example.pdf` 附录完整JS脚本。

## 概述

GeniE 内嵌 Microsoft JScript 引擎，支持完整的变量、控制流、函数、对象编程。所有交互式操作都会记录为JS命令输出到Journal文件。

## 1. Reference Point 建模（参考点建模）

核心思想：对象之间相互引用而非绝对坐标，修改一个参数即自动重建整个模型。

```javascript
// 基础框架
Bm1 = Beam(Point(0m,0m,0m), Point(0m,10m,0m));
Bm2 = Bm1.copyTranslate(Vector3d(0m,0m,10m));  // 引用复制
Bm3 = Beam(Bm1.end1, Bm2.end1);               // 引用梁端点
Bm4 = Beam(Bm1.end2, Bm2.end2);
Bm5 = Beam(Bm3.end1, Bm4.end2);
Bm6 = Beam(Bm3.end2, Bm5.project(Bm3.end2));   // 投影连接

// 修改参数后，只需改两行就可重建整个模型：
Bm1 = Beam(Point(0m,0m,0m), Point(0m,15m,0m));     // 长度改为15m
Bm2 = Bm1.copyTranslate(Vector3d(0m,0m,5m));       // 间距改为5m
```

> 注意：File > Export > Genie journal file 会丢失参考点关系，"Clean"导出使用绝对坐标。

## 2. GeniE 类型系统

| GeniE 类型 | 示例 | 转换方法 |
|-----------|------|---------|
| Length | `2.5 m`, `Radius.toDouble()` | `.toDouble()`, `Length(5.8)` |
| Vector3d | `Vector3d(1m, 0m, 0m)` | `.length()`, `.normalise()` |
| LocalSystem | `Beam1.localSystem()` | `.xVector`, `.yVector`, `.zVector` |

**关键规则**：
- `Length + Number` → 报错
- `Length * Number` → OK
- `Length + Length` → OK
- `Math`函数需要 `.toDouble()` 转换后再输入

```javascript
// 正确用法
Xcoord = Math.abs(Math.sqrt(Math.pow(Radius.toDouble(), 2) - ...));
// 错误用法
Point(3m + 5.8, 4, 5);  // TypeError! 用 Point(3m + 5.8mm, 4, 5)
```

## 3. 拓扑命名体系

建立可编程的名字模式，支持三重循环建模：

```javascript
// 定义方向数组
Nam2 = ["1","11","2","3","4","41","42","43","5","6","7"];
Nam3 = ["A","B","C","D","E","F","G","H","I","J"];
Nam4 = ["1","2","3","4","5","6","7","8","9"];

// 命名函数："Bx" + "41" + "C" + "2" → "Bx41C2"
function GetName(N1, N2, N3, N4) {
    return N1 + Nam2[N2] + Nam3[N3] + Nam4[N4];
}

// 对象获取函数
function GetObjectByName(N1, N2, N3, N4) {
    return GetNamedObject(GetName(N1, N2, N3, N4));
}

// 三重循环创建全框架梁
for (i = 0; i < NumX-1; i++) {
  for (j = 0; j < NumY-1; j++) {
    for (k = 0; k < NumZ-1; k++) {
      var aBeam = Beam(GetObjectByName("P",i,j,k), GetObjectByName("P",i+1,j,k));
      aBeam.section = Box1400;
      aBeam.material = S275;
      Rename(aBeam, GetName("Bx", i, j, k));
    }
  }
}
```

## 4. DynamicSet 动态集合

不同于StaticSet，DynamicSet在模型变化时自动更新成员：

```javascript
// 平面截取
dset1 = DynamicSet(LimitInPlane(ZPlane3d(0)));

// 组合查询：Z=0平面 + 翼缘宽<0.2m
dset1 = DynamicSet(LimitInPlane(ZPlane3d(0)) && LimitLower("Width", 0.2m));

// 命名集+直线
dset2 = DynamicSet(LimitAnd(LimitInSet(Frame_A), LimitLine(Point4, Point7)));

// 正则表达式名：所有 Bm 开头的对象
dset1 = DynamicSet(LimitString("Name", "Bm.*", true));

// 批量偏心
dset1 = DynamicSet(LimitLower("Diameter", 0.7m));
dset1.setBeamOffset(Vector3d(0m, 0m, 0.3m));
```

## 5. ModelObjects 遍历与分类

```javascript
// 按截面名分组所有梁
for (var sect in ModelObjects) {
  if (sect.supportsType(typeSection)) {
    newSet = Set();
    for (var bm in ModelObjects) {
      if (bm.supportsType(TypeStraightBeam) || bm.supportsType(TypeCurvedBeam)) {
        if (bm.section.name == sect.name) {
          newSet.add(bm);
        }
      }
    }
    newSet.name = "Set_" + sect.name;
  }
}

// 快捷遍历
for (var a in ModelObjects) {
  if (a.supportsType(typeStraightBeam))
    Print(a.Length);
}
```

## 6. 用户自定义函数与类

```javascript
// 局部坐标到全局坐标转换
function VecTra(LocX, LocY, LocZ, LocalCoordSys) {
  return LengthVector3D(
    LocalCoordSys.xVector.normalise() * LocX +
    LocalCoordSys.yVector.normalise() * LocY +
    LocalCoordSys.zVector.normalise() * LocZ
  );
}
// 使用
Bm1.setEndOffset(2, VecTra(1m, 1m, 0m, Bm1.localSystem()));

// 用户自定义类（2D几何交点）
class Point2d {
  constructor function Point2d(x, y) { m_x = Length(x); m_y = Length(y); }
  function X() { return m_x; }
  function Y() { return m_y; }
  var m_x, m_y;
}

class Line2d {
  // ...含intersect(other)方法
}

l1 = new Line2d(new Point2d(0, 10m), new Point2d(5m, 0));
l2 = new Line2d(new Point2d(10m, 0), new Point2d(0, 5m));
p = l1.intersect(l2);  // 自动计算交点
```

## 7. Excel 交互

```javascript
// 打开Excel文件读取数据
MyBook = GetObject("C:\\\\Workspace\\\\InputData.xls");
Elevation = Length(MyBook.Worksheets("sheet1").Range("C8").Value);

// 写入数据到Excel
for (i = 1; i < NumberOfData; i++) {
  MyBook.ActiveSheet.Cells(Row-1, Col).Offset(i, 0).Value = MyData[i];
}
```

## 8. JScript 载荷密度函数

```javascript
// 线载荷（参数形参，param∈[0,1]沿长度）
function component1D(param) {
  return Vector3d(1 N/m, param * 2 N/m, 0);
}

// 面载荷静水压力（20°横倾，吃水4.5m）
HeelAngle = 20;
Draught = 4.5 m;
HeelAngleRad = (HeelAngle * Math.PI) / 180;
function pressureFunction(x, y, z) {
  Zlocal = (z * Math.cos(HeelAngleRad)) - (y * Math.sin(HeelAngleRad));
  if (Zlocal < Draught) {
    return 1 G * 1025 kg/m^3 * (Draught - Zlocal);
  } else {
    return 0;
  }
}
```

## 9. 分段梁 + 批量函数更新

```javascript
// 从图纸表格自动更新数百个梁的端部加强
function DiagonalProp(Bea, End1_Offset, End2_Offset, Main_Section,
                       End1_X_center, End2_X_center, End1_Section, End2_Section) {
  var NumBeam = Bea.length;
  for (i = 1; i < NumBeam; i++) {
    Bea[i].section = Main_Section;
    if (End1_X_center != 0m) { Bea[i].setEndOffset(1, Vector3d(End1_X_center, 0m, 0m)); }
    if (End2_X_center != 0m) { Bea[i].setEndOffset(2, Vector3d(End2_X_center, 0m, 0m)); }
    if (End1_Offset != 0m) {
      Bea[i].divideSegmentAt(1, End1_Offset / Bea[i].length());
      Bea[i].SetSegmentSection(1, End1_Section);
    }
    // ...类似处理End2
  }
}

// 从图纸表格调用
var Bea = [Dx1A1, Dx1C1, Dx1E1, Dx1G1, /* 数百个 */];
DiagonalProp(Bea, 1.837m, 2.102m, D39, 0.15m, 0.16m, Node_Diagonal_X, Node_Diagonal_X);
```

## 10. 自动报告生成

```javascript
// 自动生成VonMises应力图并插入报告
function plotResult(loadcasein, setin) {
  for (var object in ModelObjects) {
    if (object.supportsType(typeModelView) && object.name() == "Demo_ModelView") {
      Delete(object);
    }
  }
  ModelView_temp = ModelView();
  ModelView_temp.addElement(DisplayConfiguration("Results - with Mesh", moPaper));
  ModelView_temp.addElement(ResultPresentation());
  ModelView_temp.resultPresentation.resultComponent = rsStress;
  ModelView_temp.resultPresentation.calculationType = rsVonMises;
  ModelView_temp.resultPresentation.optionMinmax = false;
  ModelView_temp.addElement(VisibleModel());
  loadcasein.setCurrent();
  if (setin.supportsType(typeSet)) {
    Demo_ModelView.visibleModel.include(setin);
    Demo_ModelView.activate();
  }
  Graphics.fitModel();
  var Plotfile = loadcasein.name() + setin.name() + ".jpg";
  Graphics.saveImage(Plotfile);
  return Figure(loadcasein.name() + " - " + setin.name() + " - VonMises", Plotfile);
}

// 生成Word报告
Case_1 = Report("Case_1");
Case_1.add(ChapterStructure());
Case_1.element(1).add(TablePlateCoordinate());
Case_1.element(1).add(TablePlateProperty());
Case_1.add(plotResult(Operation, UpperDeck));
Case_1.add(plotResult(Storm, UpperDeck));
Case_1.saveAs("Case_1.doc", mrWordXML);
```

## 11. 已知局限性

- JScript不支持原生文件操作（可用Excel替代）
- 对象自动命名有一定限制
- 不支持自定义对话框和按钮（.NET脚本正在评估中）
- getNamedObject vs 直接变量名有时行为不同
- `Clean Journal` 会丢失Reference Point关系
