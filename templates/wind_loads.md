# GeniE 风载荷模板 (Wind Loads Template)

本模板用于在结构模型上施加风载荷，包括风剖面定义、设备风阻和结构构件风阻，支持多种风向。

```javascript
// ============================================================
// 阶段 1: 风载荷基础设置
// ============================================================
gCOMPAT.Set_SesamCompatibilityMode(true, 22, 0, 1);
gCOMPAT.Set_AdvancedSesamCompatibilityRules(true);

// 假定已有结构模型 (proj, 所有梁/板/设备已定义)
// 以下代码假定在已有模型基础上附加风载荷

// ============================================================
// 阶段 2: 风场参数定义
// ============================================================
// 基础风参数
var windBasicSpeed_1hr = 40.0;      // 1小时平均风速 (m/s)
var windBasicSpeed_10min = 44.5;    // 10分钟平均风速 (m/s)
var windBasicSpeed_3sec = 55.0;     // 3秒阵风风速 (m/s)
var airDensity = 1.225;             // 空气密度 (kg/m³)
var referenceHeight = 10.0;         // 参考高度 (m) - 通常为海平面以上10m

// 风剖面参数 (对数律或指数律)
var windProfile = {
    type: "PowerLaw",               // 指数律风剖面
    referenceSpeed: windBasicSpeed_1hr,
    referenceHeight: referenceHeight,
    exponent: 0.12,                 // 海面粗糙度指数 (开阔海域 ≈ 0.10-0.14)
    gradientHeight: 250.0,          // 梯度风高度 (m)
    surfaceRoughness: 0.002         // 海面粗糙度长度 (m)
};

// 风向定义
var windDirections = [
    {name: "North",   angle: 0.0},                     // 北风 0°
    {name: "NE",      angle: 45.0  * Math.PI / 180},   // 东北
    {name: "East",    angle: 90.0  * Math.PI / 180},   // 东风
    {name: "SE",      angle: 135.0 * Math.PI / 180},   // 东南
    {name: "South",   angle: 180.0 * Math.PI / 180},   // 南风
    {name: "SW",      angle: 225.0 * Math.PI / 180},   // 西南
    {name: "West",    angle: 270.0 * Math.PI / 180},   // 西风
    {name: "NW",      angle: 315.0 * Math.PI / 180}    // 西北
];

// 通常分析 4-8 个主方向即可
var activeWindDirs = windDirections.slice(0, 8);

// ============================================================
// 阶段 3: 风剖面函数
// ============================================================
// 计算给定高程的风速 (指数律)
function windSpeedAtHeight(z) {
    // 海平面以上高度
    var h = Math.max(z, 0.0);  // 确保不小于0
    
    if (h <= windProfile.referenceHeight) {
        return windProfile.referenceSpeed;
    }
    
    return windProfile.referenceSpeed * Math.pow(
        h / windProfile.referenceHeight,
        windProfile.exponent
    );
}

// 计算给定高程的风压 (Pa)
function windPressureAtHeight(z) {
    var v = windSpeedAtHeight(z);
    return 0.5 * airDensity * v * v;
}

print("风剖面参数:");
print("  参考风速: " + windProfile.referenceSpeed.toFixed(1) + " m/s @" + windProfile.referenceHeight.toFixed(0) + "m");
print("  指数: " + windProfile.exponent.toFixed(2));
print("  海面风速 (0m): " + windSpeedAtHeight(0).toFixed(1) + " m/s");
print("  50m风速: " + windSpeedAtHeight(50).toFixed(1) + " m/s");
print("  100m风速: " + windSpeedAtHeight(100).toFixed(1) + " m/s");

// ============================================================
// 阶段 4: 结构构件拖曳系数定义
// ============================================================
// 圆管构件拖曳系数 (Cd, 基于雷诺数和表面粗糙度)
function getCdForPipe(diameter) {
    // 简化: 取 Cd=0.7 对光滑圆管, Cd=1.0 对粗糙/小直径管
    if (diameter < 0.3) return 1.0;   // 小管: 较高Cd
    if (diameter < 1.0) return 0.85;  // 中管
    return 0.70;                      // 大管: 较低Cd
}

// 型钢构件拖曳系数
var cdISection = 1.6;    // H型钢/工字钢 (取决于Direction)
var cdChannel   = 1.8;   // 槽钢
var cdAngle     = 1.5;   // 角钢

// 板/面板拖曳系数
var cdPlate = 1.2;       // 平板 (垂直风向)

// 遮蔽效应折减系数 (Shielding Factor)
var shieldingFactor = 0.85;  // 后排构件风速折减

// ============================================================
// 阶段 5: 构件风载荷施加 (结构部分)
// ============================================================
// 遍历所有梁构件施加风载荷
// 注意: 以下为伪代码框架, 实际需要根据 GeniE API 调整

function applyWindToStructuralMembers(windDir, windDirName) {
    
    // 获取风向单位向量
    var dirX = Math.cos(windDir);
    var dirY = Math.sin(windDir);
    
    var loadCaseName = "Wind_" + windDirName;
    var lcNum = 300 + windDirections.findIndex(function(d) {
        return d.name === windDirName;
    });
    
    var lcWind = LoadCase.Create();
    lcWind.name = loadCaseName;
    lcWind.type = Environmental;
    lcWind.number = lcNum;
    
    // === 梁构件风载荷 ===
    // 方法1: 使用 GeniE Wind Manager (推荐)
    var windManager = WindLoadManager.Create();
    windManager.SetWindProfile(windProfile.type, windProfile.referenceSpeed,
        windProfile.referenceHeight, windProfile.exponent);
    windManager.SetWindDirection(dirX, dirY, 0.0);  // 水平风向
    windManager.SetAirDensity(airDensity);
    
    // 为不同类型构件设置拖曳系数
    windManager.SetShapeCoefficientForPipeMembers(0.70);      // 默认圆管
    windManager.SetShapeCoefficientForISection(cdISection);   // 工字钢
    windManager.SetShieldingFactor(shieldingFactor);
    
    // 施加风载到所有暴露构件
    windManager.ApplyToAllBeams();
    windManager.GenerateLoadCase(lcWind);
    
    // === 板构件风载荷 ===
    // 方法2: 手动施加板面压力
    var windPressure = windPressureAtHeight(60.0);  // 甲板高度风压
    
    // 对每块板施加压力 (在 GeniE 中选中所有板块)
    // PlatePressure.Apply(plateSet, -windPressure * cdPlate, dirX, dirY, 0);
    
    print("风载荷施加完成: " + loadCaseName + " (LC" + lcNum + ")");
    return lcWind;
}

// ============================================================
// 阶段 6: 设备风载荷 (Equipment Wind Drag)
// ============================================================
function applyWindToEquipment(windDir, windDirName, equipments) {
    
    var dirX = Math.cos(windDir);
    var dirY = Math.sin(windDir);
    
    var lcNum = 400 + windDirections.findIndex(function(d) {
        return d.name === windDirName;
    });
    
    var lcEquipWind = LoadCase.Create();
    lcEquipWind.name = "Wind_Equip_" + windDirName;
    lcEquipWind.type = Environmental;
    lcEquipWind.number = lcNum;
    
    // 设备风载荷参数
    // 每个设备: {name, centerZ(高程), areaX(投影面积X), areaY(投影面积Y), cd}
    for (var e = 0; e < equipments.length; e++) {
        var eq = equipments[e];
        
        // 计算设备中心高度的风压
        var pressure = windPressureAtHeight(eq.centerZ);
        
        // 投影面积: 计算在当前风向下的有效投影面积
        var projArea = Math.abs(eq.areaX * dirX) + Math.abs(eq.areaY * dirY);
        
        // 风载荷力
        var windForce = pressure * projArea * eq.cd;
        
        // 施加到设备参考点 (需预先定义的参考点)
        var eqPoint = Point(eq.x, eq.y, eq.centerZ);
        lcEquipWind.AddNodalForce(
            eqPoint,
            windForce * dirX,   // Fx
            windForce * dirY,   // Fy
            0.0,                // Fz (忽略竖向风)
            0, 0, 0
        );
    }
    
    return lcEquipWind;
}

// ============================================================
// 阶段 7: 设备清单定义
// ============================================================
var equipmentList = [
    {
        name: "Compressor",   x: 5.0,  y: 3.0,  centerZ: 62.0,
        areaX: 4.0, areaY: 3.0, cd: 1.3,  // 压缩机投影面积和拖曳系数
        massPoint: null       // 关联到已有的质量点
    },
    {
        name: "Generator",    x: -4.0, y: -2.0, centerZ: 54.0,
        areaX: 3.5, areaY: 2.5, cd: 1.2,
        massPoint: null
    },
    {
        name: "Separator",    x: -3.0, y: 6.0,  centerZ: 62.0,
        areaX: 2.0, areaY: 5.0, cd: 0.8,  // 卧式容器, 低Cd
        massPoint: null
    },
    {
        name: "FlareBoom",    x: 12.0, y: 0.0,  centerZ: 75.0,
        areaX: 0.6, areaY: 0.6, cd: 1.0,  // 火炬臂
        massPoint: null
    },
    {
        name: "Helideck",     x: 0.0,  y: -10.0, centerZ: 65.0,
        areaX: 20.0, areaY: 20.0, cd: 1.2, // 直升机甲板
        massPoint: null
    }
];

// ============================================================
// 阶段 8: 批量生成风载荷工况
// ============================================================
var windLoadCasesStruct = [];
var windLoadCasesEquip  = [];

for (var w = 0; w < activeWindDirs.length; w++) {
    var dir = activeWindDirs[w];
    
    // 结构构件风载荷
    var lcStruct = applyWindToStructuralMembers(dir.angle, dir.name);
    windLoadCasesStruct.push(lcStruct);
    
    // 设备风载荷
    var lcEquip = applyWindToEquipment(dir.angle, dir.name, equipmentList);
    windLoadCasesEquip.push(lcEquip);
}

// ============================================================
// 阶段 9: 风载荷组合
// ============================================================
// 将结构风载和设备风载合并到同一工况
var windCombinedCases = [];

for (var c = 0; c < activeWindDirs.length; c++) {
    var dir = activeWindDirs[c];
    
    var lcWindCombined = LoadCase.Create();
    lcWindCombined.name = "Wind_Total_" + dir.name;
    lcWindCombined.type = Environmental;
    lcWindCombined.number = 350 + c;
    
    // 合并结构风载和设备风载的效果
    // (GeniE 中通过 LoadCase 继承或手动组合)
    
    windCombinedCases.push(lcWindCombined);
    printed("合并风载荷工况: " + lcWindCombined.name + " (LC" + lcWindCombined.number + ")");
}

// ============================================================
// 阶段 10: 风载荷工况列表展示
// ============================================================
print("\n===== 风载荷工况汇总 =====");
print("风向数: " + activeWindDirs.length);
print("空气密度: " + airDensity.toFixed(3) + " kg/m³");
print("参考风速: " + windBasicSpeed_1hr.toFixed(1) + " m/s (1hr)");
print("");

print("结构风载荷工况:");
for (var i = 0; i < windLoadCasesStruct.length; i++) {
    print("  " + windLoadCasesStruct[i].name + " (LC" + windLoadCasesStruct[i].number + ")");
}

print("\n设备风载荷工况:");
for (var j = 0; j < windLoadCasesEquip.length; j++) {
    print("  " + windLoadCasesEquip[j].name + " (LC" + windLoadCasesEquip[j].number + ")");
}

print("\n合并风载荷工况:");
for (var k = 0; k < windCombinedCases.length; k++) {
    print("  " + windCombinedCases[k].name + " (LC" + windCombinedCases[k].number + ")");
}
print("===============================");
```

## 使用说明

1. 此模板需附加到已有结构模型上运行（先运行 jacket/topside 模板）
2. `WindLoadManager` 是 GeniE 内置的风载荷管理器，自动处理构件投影面积
3. 风剖面参数需根据项目所在地的气象数据调整
4. 拖曳系数 Cd 可参考 DNV-RP-C205 或 ISO 19901-1
5. 设备风载荷的投影面积应从设备布置图中获取
6. 遮蔽系数 0.85 适用于典型的导管架/上部组块结构
7. 如需风雪/冰载荷，可在同一模板中扩展
