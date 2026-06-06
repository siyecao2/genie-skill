# GeniE 起重机基座模型模板 (Crane Pedestal Model Template)

本模板用于创建海上平台起重机基座结构模型，包括圆柱基座柱、环向加劲肋、甲板集成和载荷施加区域细化。

```javascript
// ============================================================
// 阶段 1: 兼容性设置与项目初始化
// ============================================================
gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);
gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);

var proj = Project.New();
proj.SetName("Crane_Pedestal_Model");
proj.SetDescription("起重机基座局部结构分析");

var tol = ToleranceManager();
tol.SetDefaultTolerance(0.0005);  // 精细公差

// ============================================================
// 阶段 2: 材料定义
// ============================================================
// 基座柱: S420 高强度钢 (高应力区)
var matS420 = Material.Create();
matS420.name = "S420";
matS420.type = StructuralSteel;
matS420.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS420.SetYieldStress(420e6);
matS420.SetUltimateStress(520e6);

// 甲板/加劲肋: S355
var matS355 = Material.Create();
matS355.name = "S355";
matS355.type = StructuralSteel;
matS355.SetLinearIsotropicProperties(210e9, 0.3, 7850);
matS355.SetYieldStress(355e6);
matS355.SetUltimateStress(490e6);

// ============================================================
// 阶段 3: 截面定义
// ============================================================
// 基座柱截面 (大直径圆管)
var secPedestal = PipeSection.Create(4.500, 0.060);  // D=4500mm, t=60mm
secPedestal.name = "PEDESTAL_MAIN";
secPedestal.material = matS420;

// 环向加劲肋 (T型截面)
var secRingStiff = TSection.Create();
secRingStiff.name = "RING_STIFF_T300";
secRingStiff.material = matS355;
secRingStiff.SetDimensions(0.300, 0.200, 0.020, 0.020);  // T300x200x20x20

// 甲板支撑梁
var secSupportBeam = ISection.Create();
secSupportBeam.name = "SUPPORT_BEAM_H500";
secSupportBeam.material = matS355;
secSupportBeam.SetDimensions(0.500, 0.250, 0.016, 0.025);

// ============================================================
// 阶段 4: 几何参数
// ============================================================
var pedestalRadius = 2.250;      // 基座柱半径 (m)
var pedestalHeight = 8.000;      // 基座柱总高 (m)
var pedestalZBot  = 60.000;      // 基座底部高程 (甲板面) (m)
var pedestalZTop  = pedestalZBot + pedestalHeight;

var deckThickness  = 0.025;      // 甲板厚度 (m)
var deckExtent     = 8.000;      // 甲板扩展区域半径 (从柱中心)

// 环向加劲肋高程
var ringZLevels = [
    pedestalZBot + 0.5,   // 底部加强环
    pedestalZBot + 3.0,   // 中部加强环
    pedestalZBot + 5.5,   // 中上部加强环
    pedestalZBot + 7.5    // 顶部法兰环 (紧邻载荷点)
];

// ============================================================
// 阶段 5: GuidePlane 工作平面
// ============================================================
// 甲板面工作平面
var gpDeck = GuidePlane.Create();
gpDeck.name = "GP_Deck";
gpDeck.SetOrigin(Point(0, 0, pedestalZBot));
gpDeck.SetNormal(Vector(0, 0, 1));
gpDeck.snapmode = true;

// 基座顶部工作平面
var gpTop = GuidePlane.Copy(gpDeck, Point(0, 0, pedestalZTop));
gpTop.name = "GP_PedestalTop";
gpTop.snapmode = true;

// 各环向加劲肋工作平面
var gpRings = [];
for (var i = 0; i < ringZLevels.length; i++) {
    var gpRing = GuidePlane.Copy(gpDeck, Point(0, 0, ringZLevels[i]));
    gpRing.name = "GP_Ring_Z" + ringZLevels[i].toFixed(1);
    gpRing.snapmode = true;
    gpRings.push(gpRing);
}

// ============================================================
// 阶段 6: 基座圆柱建模 (Pedestal Column)
// ============================================================
// 基座柱中心线
var pedestalAxis = Line(
    Point(0, 0, pedestalZBot),
    Point(0, 0, pedestalZTop)
);

// 基座柱梁
var pedestalBeam = Beam.Create(pedestalAxis, secPedestal);
pedestalBeam.name = "PEDESTAL_COLUMN";

// 基座圆柱壳板 (SkinCurves 蒙皮)
var nSegments = 36;  // 36 段精确近似圆

function createCirclePoints(cx, cy, radius, z, nSeg) {
    var pts = [];
    for (var i = 0; i < nSeg; i++) {
        var angle = (i / nSeg) * 2 * Math.PI;
        pts.push(Point(cx + radius * Math.cos(angle), cy + radius * Math.sin(angle), z));
    }
    return pts;
}

var bottomCircle = createCirclePoints(0, 0, pedestalRadius, pedestalZBot, nSegments);
var topCircle    = createCirclePoints(0, 0, pedestalRadius, pedestalZTop, nSegments);

// 通过上下圆周导引创建蒙皮曲面
var bottomGuides = [];
var topGuides = [];
for (var j = 0; j < nSegments; j++) {
    var jn = (j + 1) % nSegments;
    bottomGuides.push(Line(bottomCircle[j], bottomCircle[jn]));
    topGuides.push(Line(topCircle[j], topCircle[jn]));
}

var pedestalSkin = SkinCurves.Create(bottomGuides, topGuides);
pedestalSkin.name = "SKIN_Pedestal";
pedestalSkin.thickness = 0.060;  // 与梁截面厚度一致
pedestalSkin.material = matS420;

// ============================================================
// 阶段 7: 环向加劲肋 (Ring Stiffeners)
// ============================================================
var ringStiffeners = [];

for (var r = 0; r < ringZLevels.length; r++) {
    var ringZ = ringZLevels[r];
    var circlePts = createCirclePoints(0, 0, pedestalRadius, ringZ, nSegments);
    
    // 环向梁 - 分段圆弧
    for (var s = 0; s < nSegments; s++) {
        var sn = (s + 1) % nSegments;
        var ringBeam = Beam.Create(
            Line(circlePts[s], circlePts[sn]),
            secRingStiff
        );
        ringBeam.name = "RING_STIFF_Z" + ringZ.toFixed(1) + "_" + s;
        ringStiffeners.push(ringBeam);
    }
}

// 顶部加强法兰 (更密的环)
var topRingPts = createCirclePoints(0, 0, pedestalRadius, ringZLevels[ringZLevels.length - 1], 72);
for (var t = 0; t < 72; t++) {
    var tn = (t + 1) % 72;
    var topRingBeam = Beam.Create(
        Line(topRingPts[t], topRingPts[tn]),
        secRingStiff
    );
    topRingBeam.name = "TOP_FLANGE_" + t;
}

// ============================================================
// 阶段 8: 甲板集成 (Deck Integration)
// ============================================================
// 甲板板 - 方形区域围绕基座
var deckHalf = deckExtent;
var deckPlate = Plate.CreateByPoints([
    Point(-deckHalf, -deckHalf, pedestalZBot),
    Point( deckHalf, -deckHalf, pedestalZBot),
    Point( deckHalf,  deckHalf, pedestalZBot),
    Point(-deckHalf,  deckHalf, pedestalZBot)
]);
deckPlate.name = "PLT_Deck_Local";
deckPlate.thickness = deckThickness;
deckPlate.material = matS355;

// 甲板支撑梁 (从基座柱径向辐射)
var nSupports = 8;  // 8根径向支撑梁
for (var u = 0; u < nSupports; u++) {
    var angle = (u / nSupports) * 2 * Math.PI;
    var dirX = Math.cos(angle);
    var dirY = Math.sin(angle);
    
    var supBeam = Beam.Create(
        Line(
            Point(dirX * pedestalRadius, dirY * pedestalRadius, pedestalZBot),
            Point(dirX * deckExtent,    dirY * deckExtent,    pedestalZBot)
        ),
        secSupportBeam
    );
    supBeam.name = "SUPPORT_RADIAL_" + u;
}

// ============================================================
// 阶段 9: 边界条件 (Support Curves)
// ============================================================
// 甲板板四边简支
var bcDeckEdge = BC.Create();
bcDeckEdge.name = "BC_DeckEdge_SimplySupported";
bcDeckEdge.SetDisplacement(0, 0, 0);      // UX=UY=UZ=0
bcDeckEdge.SetRotation(999, 999, 999);    // 铰接

// 选取甲板板四边施加边界 (通过支持曲线)
var deckEdgeBottom = SupportCurve.Create(
    Line(Point(-deckHalf, -deckHalf, pedestalZBot), Point(deckHalf, -deckHalf, pedestalZBot))
);
deckEdgeBottom.name = "SC_DeckBottom";
deckEdgeBottom.SetBC(bcDeckEdge);

var deckEdgeTop = SupportCurve.Create(
    Line(Point(-deckHalf, deckHalf, pedestalZBot), Point(deckHalf, deckHalf, pedestalZBot))
);
deckEdgeTop.name = "SC_DeckTop";
deckEdgeTop.SetBC(bcDeckEdge);

var deckEdgeLeft = SupportCurve.Create(
    Line(Point(-deckHalf, -deckHalf, pedestalZBot), Point(-deckHalf, deckHalf, pedestalZBot))
);
deckEdgeLeft.name = "SC_DeckLeft";
deckEdgeLeft.SetBC(bcDeckEdge);

var deckEdgeRight = SupportCurve.Create(
    Line(Point(deckHalf, -deckHalf, pedestalZBot), Point(deckHalf, deckHalf, pedestalZBot))
);
deckEdgeRight.name = "SC_DeckRight";
deckEdgeRight.SetBC(bcDeckEdge);

// ============================================================
// 阶段 10: 起重机载荷施加点
// ============================================================
// 起重机载荷作用点 (基座柱顶部中心上方)
var craneLoadPoint = Point(0, 0, pedestalZTop + 0.5);

// 创建参考点用于集中力施加
var refPoint = ReferencePoint.Create(craneLoadPoint);
refPoint.name = "RP_Crane_Load";

// 将参考点与基座顶部耦合 (刚性连接)
var coupling = RigidLink.Create();
coupling.name = "RL_CraneToPedestal";
coupling.SetMaster(refPoint);
// 耦合基座顶部所有节点 (在实际操作中通过选取顶部环)
for (var v = 0; v < topCircle.length; v++) {
    coupling.AddSlave(topCircle[v]);
}

// ============================================================
// 阶段 11: 载荷定义
// ============================================================
// LC 101: 自重
var lcGravity = LoadCase.Create();
lcGravity.name = "Gravity";
lcGravity.type = Permanent;
lcGravity.number = 101;
lcGravity.ActivateSelfWeight(0, 0, -1, 1.0);

// LC 501: 起重机工作载荷 (竖向 + 弯矩)
var lcCraneOp = LoadCase.Create();
lcCraneOp.name = "Crane_Operating";
lcCraneOp.type = Live;
lcCraneOp.number = 501;

// 起重机额定起重 + 自重效应
var Fz_crane = -800e3;      // 竖向力 800kN (向下)
var Mx_crane =  2500e3;     // 倾覆弯矩 2500 kN·m
var My_crane =  1800e3;     // 侧向弯矩 1800 kN·m
var Fx_crane =  150e3;      // 水平力 150kN
var Fy_crane =  120e3;      // 水平力 120kN

lcCraneOp.AddNodalForce(refPoint, Fx_crane, Fy_crane, Fz_crane, Mx_crane, My_crane, 0);

// LC 502: 起重机极端载荷 (风暴自存)
var lcCraneStorm = LoadCase.Create();
lcCraneStorm.name = "Crane_Storm_Survival";
lcCraneStorm.type = Environmental;
lcCraneStorm.number = 502;
lcCraneStorm.AddNodalForce(refPoint, 80e3, 90e3, -200e3, 800e3, 600e3, 0);

// ============================================================
// 阶段 12: 载荷组合
// ============================================================
var lcULS_Operating = LoadCombination.Create();
lcULS_Operating.name = "ULS_Crane_Operating";
lcULS_Operating.type = ULS;
lcULS_Operating.number = 5001;
lcULS_Operating.AddLoadCaseFactor(lcGravity,  1.30);
lcULS_Operating.AddLoadCaseFactor(lcCraneOp,  1.50);

var lcULS_Storm = LoadCombination.Create();
lcULS_Storm.name = "ULS_Crane_Storm";
lcULS_Storm.type = ULS;
lcULS_Storm.number = 5002;
lcULS_Storm.AddLoadCaseFactor(lcGravity,    1.00);
lcULS_Storm.AddLoadCaseFactor(lcCraneStorm, 1.35);

// ============================================================
// 阶段 13: 网格与分析设置
// ============================================================
SimplifyTopology();

var act = Activity.Create();
act.name = "Activity_CranePedestal";

// 网格控制 - 载荷区域精细网格
var meshCtrl = MeshControl.Default();
meshCtrl.SetGlobalElementSize(0.3);                     // 全局 0.3m
meshCtrl.SetLocalRefinementOnPoint(craneLoadPoint, 0.05, 0.80);  // 载荷点附近精细网格
meshCtrl.SetLocalRefinementOnCylinder(Point(0, 0, pedestalZBot + pedestalHeight/2),
    Point(0, 0, pedestalZTop), pedestalRadius + 1.0, 0.08);  // 基座柱附近细化
meshCtrl.SetMinElementQuality(0.3);

var meshSet = MeshSet.Create();
meshSet.name = "Mesh_CranePedestal";
meshSet.AddAllBodies();
meshSet.SetMeshControl(meshCtrl);
meshSet.GenerateMesh();

// 求解器
var solverOp = LinearStaticSolver();
solverOp.name = "Solve_Crane_Operating";
solverOp.AddLoadCombination(lcULS_Operating);

var solverStorm = LinearStaticSolver();
solverStorm.name = "Solve_Crane_Storm";
solverStorm.AddLoadCombination(lcULS_Storm);

// 分析
var analysis = Analysis.Create();
analysis.name = "Analysis_CranePedestal";
analysis.AddSolver(solverOp);
analysis.AddSolver(solverStorm);
analysis.SetActivity(act);

// analysis.Solve();

// ============================================================
// 阶段 14: 结果输出
// ============================================================
print("===== 起重机基座模型创建完成 =====");
print("基座柱: D=" + (pedestalRadius * 2).toFixed(1) + "m, H=" + pedestalHeight.toFixed(1) + "m");
print("环向加劲肋层数: " + ringZLevels.length);
print("径向支撑梁数: " + nSupports);
print("起重机载荷点: (" + craneLoadPoint.x + ", " + craneLoadPoint.y + ", " + craneLoadPoint.z + ")");
print("载荷工况: 操作工况 (LC501) + 风暴工况 (LC502)");
print("精细网格区域: 载荷点 50mm, 基座柱 80mm");
print("===============================");
```

## 使用说明

1. 根据实际起重机规格修改载荷值 (Fz, Mx, My)
2. `RigidLink` 用于将集中力均匀传递到基座柱顶部所有节点
3. 环向加劲肋间距和数量应根据屈曲分析结果调整
4. `SupportCurve` 模拟甲板边界条件，实际项目中可能需改为弹性约束
5. 载荷施加区域的网格细化对局部应力精度至关重要
