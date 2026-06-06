# Code Check - GeniE SESAM Snippets
> **SESAM 源**: `GeniE V8.8-08 Help\UserDocumentation\CheckCode\CodeCheck.htm + \Help\ReferenceDocuments\`

## CapacityManager Creation & Setup

```javascript
// CapacityManager is the main code check orchestrator
var capMan = CapacityManager();
capMan.name = "JACKET_CODE_CHECK";

// Set the code checking standard
capMan.standard = "ISO19902";  // Offshore structures
// Alternatives: "AISC_ASD", "AISC_LRFD", "API_WSD", "API_LRFD",
//               "EN1993_1_1", "NORSOK_N004", "DS449", "CSR"

// Set analysis results to check against
capMan.setAnalysisResults(anaFull);

// Global capacity parameters
capMan.materialFactor = 1.15;       // Gamma_M for steel (ISO 19902)
capMan.bucklingCurve = "a";          // Buckling curve for tubulars
capMan.effectiveLengthFactor_K = 1.0;
capMan.unbracedLength_L = "segment"; // Use segment length
```

## Member Code Checking

```javascript
// Define members from structure
var memberLegs = Member();
memberLegs.name = "JACKET_LEGS";
memberLegs.fromStructure(getBeamsByPattern("LEG_*"));

var memberBraces = Member();
memberBraces.name = "BRACES";
memberBraces.fromStructure(getBeamsByPattern("BRACE_*"));

var memberDeckBeams = Member();
memberDeckBeams.name = "DECK_BEAMS";
memberDeckBeams.fromStructure(getBeamsByPattern("DECK_BM_*"));

// Member-specific parameters
memberLegs.codeCheck.bucklingLengthFactor_Ky = 1.0;
memberLegs.codeCheck.bucklingLengthFactor_Kz = 1.0;
memberLegs.codeCheck.bucklingLength_Ly = 5.0 m;
memberLegs.codeCheck.bucklingLength_Lz = 5.0 m;

memberBraces.codeCheck.bucklingLengthFactor_Ky = 0.8;  // Reduced for braces
memberBraces.codeCheck.bucklingLengthFactor_Kz = 0.8;
memberBraces.codeCheck.bucklingCurve = "a";

// Add members to capacity manager
capMan.addMember(memberLegs);
capMan.addMember(memberBraces);
capMan.addMember(memberDeckBeams);
```

## Joint Code Checking

```javascript
// Joint definition for tubular joint punching shear check
var jointK1 = Joint();
jointK1.name = "JOINT_K1";
jointK1.type = "K";            // K-joint, or "T", "Y", "KT", "X", "DoubleK"
jointK1.chord = getBeam("LEG_A1");

// Add braces framing into the joint
jointK1.addBrace(getBeam("BRACE_K1A"));
jointK1.addBrace(getBeam("BRACE_K1B"));
jointK1.jointCanThickness = 0.05 m;  // Can thickness at joint

// Joint-specific code check parameters
jointK1.codeCheck.gap = 0.05 m;              // Gap between braces at chord face
jointK1.codeCheck.chordEndFixity = "fixed";   // Chord end condition
jointK1.codeCheck.braceEndFixity = "pinned";  // Brace end condition
jointK1.codeCheck.beta = 0.6;                 // d/D ratio (brace diameter / chord diameter)
jointK1.codeCheck.gamma = 15.0;               // D/2T ratio (chord radius / chord thickness)
jointK1.codeCheck.tau = 0.5;                  // t/T ratio (brace thickness / chord thickness)
jointK1.codeCheck.theta = 45 deg;             // Brace-to-chord angle

// Add joint to capacity manager
capMan.addJoint(jointK1);
```

## CapacityRun Creation & Execution

```javascript
// Create a capacity check run
var capRun = CapacityRun();
capRun.name = "ULS_CHECK";
capRun.loadCombination = combULS;
capRun.codeCheck = "member";  // Member check only

// Alternative: joint-only check
var capRunJoint = CapacityRun();
capRunJoint.name = "JOINT_CHECK";
capRunJoint.loadCombination = combULS;
capRunJoint.codeCheck = "joint";

// Combined member + joint check
var capRunFull = CapacityRun();
capRunFull.name = "FULL_CHECK";
capRunFull.loadCombination = combULS;
capRunFull.codeCheck = "both";

// Execute the code check
capMan.addCapacityRun(capRunFull);
capMan.execute();

print("Code check execution completed");
print("Standard: " + capMan.standard);
```

## Available Standards

```javascript
// AISC ASD - Allowable Stress Design (American)
var stdAISC_ASD = "AISC_ASD";
// capMan.standard = stdAISC_ASD;
// capMan.AISC_edition = 9;  // 9th edition

// AISC LRFD - Load and Resistance Factor Design (American)
var stdAISC_LRFD = "AISC_LRFD";
// capMan.standard = stdAISC_LRFD;
// capMan.AISC_edition = 14;  // 14th edition

// API WSD - Working Stress Design (offshore)
var stdAPI_WSD = "API_WSD";
// capMan.standard = stdAPI_WSD;
// capMan.API_edition = 22;  // RP 2A 22nd edition

// API LRFD - offshore LRFD
var stdAPI_LRFD = "API_LRFD";

// EN 1993-1-1 - Eurocode 3 (steel structures)
var stdEN1993 = "EN1993_1_1";
// capMan.standard = stdEN1993;
// capMan.EN1993_annex = "B";  // National annex

// ISO 19902 - Fixed steel offshore structures
var stdISO19902 = "ISO19902";

// NORSOK N-004 - Norwegian offshore standard
var stdNorsok = "NORSOK_N004";

// DS 449 - Danish offshore standard
var stdDS = "DS449";

// CSR - Common Structural Rules (ships)
var stdCSR = "CSR";
```

## Global vs Local Code Check Parameters

```javascript
var capMan2 = CapacityManager();
capMan2.name = "DETAILED_CHECK";
capMan2.standard = "ISO19902";
capMan2.setAnalysisResults(anaFull);

// Global parameters (apply to all members unless overridden)
capMan2.materialFactor = 1.15;
capMan2.bucklingCurve = "a";
capMan2.effectiveLengthFactor_K = 1.0;
capMan2.corrosionAllowance = 3 mm;

// Local overrides for specific members
var memberCritical = Member();
memberCritical.name = "CRITICAL_JOINT_ZONE";
memberCritical.fromStructure(getBeamsByPattern("CRIT_*"));

// Override global parameters locally
memberCritical.codeCheck.materialFactor = 1.25;     // Higher safety factor
memberCritical.codeCheck.bucklingCurve = "c";        // More conservative curve
memberCritical.codeCheck.effectiveLengthFactor_Ky = 1.2;  // Higher K-factor
memberCritical.codeCheck.effectiveLengthFactor_Kz = 1.2;
memberCritical.codeCheck.corrosionAllowance = 5 mm;  // Higher corrosion allowance

capMan2.addMember(memberCritical);
```

## Results Investigation (UF)

```javascript
// After executing code check, investigate results
capMan.execute();

// Get utilization factors
var results = capMan.results();

// Iterate through results to find critical members
var maxUF = 0;
var criticalMember = "";
var criticalLoadCase = "";

for (var i = 0; i < results.memberResults.length; i++) {
    var memberRes = results.memberResults[i];
    var uf = memberRes.utilizationFactor;

    if (uf > maxUF) {
        maxUF = uf;
        criticalMember = memberRes.memberName;
        criticalLoadCase = memberRes.loadCaseName;
    }

    if (uf > 1.0) {
        print("OVERSTRESSED: " + memberRes.memberName +
              " UF = " + uf.toFixed(3) +
              " (LC: " + memberRes.loadCaseName + ")");
    }
}

print("Maximum UF: " + maxUF.toFixed(3) + " in " + criticalMember);

// Joint results
for (var j = 0; j < results.jointResults.length; j++) {
    var jointRes = results.jointResults[j];
    if (jointRes.utilizationFactor > 0.8) {
        print("Joint " + jointRes.jointName +
              " UF = " + jointRes.utilizationFactor.toFixed(3));
    }
}
```

## Redesign Workflow

```javascript
// Iterative redesign workflow
function optimizeSections(capacityManager, targetUF) {
    var maxIterations = 10;
    var converged = false;

    for (var iter = 0; iter < maxIterations && !converged; iter++) {
        print("--- Redesign iteration " + (iter + 1) + " ---");
        capacityManager.execute();

        var results = capacityManager.results();
        converged = true;

        for (var i = 0; i < results.memberResults.length; i++) {
            var res = results.memberResults[i];
            var uf = res.utilizationFactor;

            if (uf > 1.05) {
                // Overstressed - increase section
                var beam = getBeam(res.memberName);
                var currentDia = beam.section.diameter;
                var newDia = currentDia * (1 + (uf - 1.0) * 0.5);  // Scale up
                beam.section = PipeSection(newDia, beam.section.thickness);
                converged = false;
                print("  Upsized " + res.memberName + " to D=" + newDia);
            } else if (uf < targetUF - 0.1 && targetUF < 0.9) {
                // Under-utilized - consider downsizing
                print("  " + res.memberName + " under-utilized (UF=" +
                      uf.toFixed(3) + ")");
                // Optional downsizing logic here
            }
        }
    }

    if (converged) {
        print("Design converged within " + maxIterations + " iterations");
    }
}

// Usage
optimizeSections(capMan, 0.85);
```