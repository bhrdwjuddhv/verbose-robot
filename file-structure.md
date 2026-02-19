# File Structure - PharmaGuard (6-Hour Hackathon)

## Overview

This file structure is optimized for a 6-hour hackathon build. It's minimal, focused, and supports rapid development.

---

## Project Structure

```
pharmaguard/
├── client/                      # Frontend (React + Vite)
│   ├── src/
│   │   ├── pages/
│   │   │   ├── GenomeUpload.jsx
│   │   │   └── RiskDashboard.jsx
│   │   ├── components/
│   │   │   ├── FileDropzone.jsx
│   │   │   ├── DrugSelector.jsx
│   │   │   ├── RiskCard.jsx
│   │   │   ├── GeneViewer3D.jsx
│   │   │   └── ClinicalChat.jsx
│   │   ├── services/
│   │   │   └── api.js
│   │   ├── theme.js
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── package.json
│   └── vite.config.js
│
├── server/                      # Backend (Node.js + Express)
│   ├── routes/
│   │   └── api.js
│   ├── controllers/
│   │   ├── uploadController.js
│   │   ├── analysisController.js
│   │   └── chatController.js
│   ├── services/
│   │   ├── vcfParser.js
│   │   ├── starAlleleResolver.js
│   │   ├── phenotypeMapper.js
│   │   ├── riskEngine.js
│   │   └── llmService.js
│   ├── config/
│   │   ├── starAlleles.js
│   │   ├── phenotypeMaps.js
│   │   └── riskMatrix.js
│   ├── models/
│   │   └── Analysis.js
│   ├── uploads/
│   ├── server.js
│   └── package.json
│
└── README.md
```

---

## Frontend Structure

### `/client/src/pages/`

#### `GenomeUpload.jsx`
**Purpose:** Main upload page

**Contains:**
- FileDropzone component
- DrugSelector component
- Analyze button
- Loading state

**State:**
```javascript
{
  file: File | null,
  selectedDrugs: string[],
  uploading: boolean,
  error: string | null
}
```

**Flow:**
1. User drops VCF file
2. User selects drugs (or defaults to all 6)
3. Click "Analyze"
4. POST /api/analyze
5. Navigate to RiskDashboard

---

#### `RiskDashboard.jsx`
**Purpose:** Display analysis results

**Contains:**
- Gene filter tabs
- Grid of RiskCard components
- GeneViewer3D component
- ClinicalChat component
- Export button

**State:**
```javascript
{
  results: AnalysisResult[],
  selectedGene: string | 'All',
  loading: boolean
}
```

---

### `/client/src/components/`

#### `FileDropzone.jsx`
**Purpose:** Drag-and-drop file upload

**Features:**
- Material-UI dropzone styling
- File validation (5MB, .vcf/.vcf.gz)
- Upload progress indicator
- Error messages

**Props:**
```javascript
{
  onFileSelect: (file: File) => void,
  maxSize: number // 5MB
}
```

---

#### `DrugSelector.jsx`
**Purpose:** Multi-select drug chips

**Features:**
- 6 drug chips: CODEINE, WARFARIN, CLOPIDOGREL, SIMVASTATIN, AZATHIOPRINE, FLUOROURACIL
- Material-UI Chip components
- Visual selection feedback
- "Select All" option

**Props:**
```javascript
{
  selectedDrugs: string[],
  onChange: (drugs: string[]) => void
}
```

---

#### `RiskCard.jsx`
**Purpose:** Display single drug-gene risk result

**Features:**
- Color-coded by risk label
- Shows: drug, gene, diplotype, phenotype, confidence
- Clinical recommendation
- LLM explanation
- Expandable for details

**Props:**
```javascript
{
  drug: string,
  gene: string,
  riskLabel: 'Safe' | 'Adjust' | 'Toxic' | 'Unknown',
  confidence: number,
  diplotype: string,
  phenotype: string,
  recommendation: string,
  explanation: string
}
```

**Color Mapping:**
```javascript
Safe → Green (#4CAF50)
Adjust → Orange (#FF9800)
Toxic/Ineffective → Red (#F44336)
Unknown → Purple (#9C27B0)
```

---

#### `GeneViewer3D.jsx`
**Purpose:** 3D visualization of variants

**Features:**
- React Three Fiber canvas
- Simple sphere geometry for variants
- Color-coded by risk level
- Orbit controls (rotate + zoom)

**Props:**
```javascript
{
  variants: Array<{
    position: [x, y, z],
    risk: string,
    gene: string
  }>
}
```

**Implementation:**
```javascript
<Canvas>
  <ambientLight intensity={0.5} />
  <pointLight position={[10, 10, 10]} />
  <OrbitControls />
  {variants.map((v, i) => (
    <Sphere
      key={i}
      position={v.position}
      args={[0.5, 32, 32]}
      material-color={getRiskColor(v.risk)}
    />
  ))}
</Canvas>
```

---

#### `ClinicalChat.jsx`
**Purpose:** Ask questions about results

**Features:**
- Simple text input
- Message history
- GPT-4 powered responses

**Props:**
```javascript
{
  analysisId: string
}
```

**State:**
```javascript
{
  messages: Array<{ role: 'user' | 'assistant', content: string }>,
  input: string,
  loading: boolean
}
```

---

### `/client/src/services/`

#### `api.js`
**Purpose:** Centralized API client

**Methods:**
```javascript
// Upload VCF file
uploadVCF(file: File): Promise<{ uploadId: string }>

// Analyze VCF
analyzeVCF(uploadId: string, drugs: string[]): Promise<{ analysisId: string, results: [] }>

// Get results
getResults(analysisId: string): Promise<AnalysisResult[]>

// Chat
sendChatMessage(analysisId: string, question: string): Promise<{ answer: string }>
```

**Configuration:**
```javascript
const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000';
```

---

### `/client/src/theme.js`

**Purpose:** Material-UI theme configuration

```javascript
import { createTheme } from '@mui/material/styles';

export const theme = createTheme({
  palette: {
    mode: 'dark',
    primary: {
      main: '#00BCD4', // Cyan
    },
    background: {
      default: '#0A1929',
      paper: '#132F4C',
    },
    success: { main: '#4CAF50' },  // Green (Safe)
    warning: { main: '#FF9800' },  // Orange (Adjust)
    error: { main: '#F44336' },    // Red (Toxic)
    info: { main: '#9C27B0' },     // Purple (Unknown)
  },
  typography: {
    fontFamily: 'Roboto, sans-serif',
  },
});
```

---

## Backend Structure

### `/server/routes/`

#### `api.js`
**Purpose:** Define all API endpoints

```javascript
const express = require('express');
const router = express.Router();

router.post('/upload', uploadController.handleUpload);
router.post('/analyze', analysisController.analyze);
router.get('/results/:id', analysisController.getResults);
router.post('/chat', chatController.handleChat);

module.exports = router;
```

---

### `/server/controllers/`

#### `uploadController.js`
**Purpose:** Handle file uploads

**Methods:**
```javascript
handleUpload(req, res) {
  // Validate file
  // Save to uploads/
  // Return uploadId
}
```

---

#### `analysisController.js`
**Purpose:** Orchestrate analysis pipeline

**Methods:**
```javascript
async analyze(req, res) {
  const { uploadId, drugs } = req.body;
  
  // 1. Parse VCF
  const variants = await vcfParser.parse(uploadId);
  
  // 2. Resolve diplotypes
  const diplotypes = starAlleleResolver.resolve(variants);
  
  // 3. Map phenotypes
  const phenotypes = phenotypeMapper.map(diplotypes);
  
  // 4. Classify risks
  const risks = riskEngine.classify(phenotypes, drugs);
  
  // 5. Generate explanations
  const explanations = await llmService.explain(risks);
  
  // 6. Save to MongoDB
  const analysis = await Analysis.create({ results: risks });
  
  // 7. Return results
  res.json({ analysisId: analysis._id, results: risks });
}

getResults(req, res) {
  // Query MongoDB by analysisId
  // Return results
}
```

---

#### `chatController.js`
**Purpose:** Handle clinical chat

**Methods:**
```javascript
async handleChat(req, res) {
  const { analysisId, question } = req.body;
  
  // Get analysis results
  const analysis = await Analysis.findById(analysisId);
  
  // Call GPT-4 with context
  const answer = await llmService.chat(question, analysis.results);
  
  res.json({ answer });
}
```

---

### `/server/services/`

#### `vcfParser.js`
**Purpose:** Parse VCF files using streaming

**Methods:**
```javascript
async parse(uploadId) {
  const filePath = `uploads/${uploadId}.vcf`;
  const variants = [];
  
  const rl = readline.createInterface({
    input: fs.createReadStream(filePath),
    crlfDelay: Infinity
  });
  
  for await (const line of rl) {
    if (line.startsWith('#')) continue;
    const variant = parseVariantLine(line);
    if (isTargetGene(variant)) {
      variants.push(variant);
    }
  }
  
  return variants;
}

function isTargetGene(variant) {
  const targetGenes = ['CYP2D6', 'CYP2C19', 'CYP2C9', 'SLCO1B1', 'TPMT', 'DPYD'];
  return targetGenes.includes(variant.gene);
}
```

---

#### `starAlleleResolver.js`
**Purpose:** Resolve diplotypes from variants

**Methods:**
```javascript
resolve(variants) {
  const diplotypes = {};
  
  for (const gene of TARGET_GENES) {
    const geneVariants = variants.filter(v => v.gene === gene);
    diplotypes[gene] = resolveDiplotype(gene, geneVariants);
  }
  
  return diplotypes;
}

function resolveDiplotype(gene, variants) {
  const alleles = STAR_ALLELES[gene];
  
  // Match variants to star allele patterns
  const matches = [];
  for (const [allele, pattern] of Object.entries(alleles)) {
    if (matchesPattern(variants, pattern)) {
      matches.push(allele);
    }
  }
  
  // Default to *1/*1 if no matches
  if (matches.length === 0) {
    return { allele1: '*1', allele2: '*1', notation: '*1/*1', confidence: 0.9 };
  }
  
  // Return diplotype
  return {
    allele1: matches[0],
    allele2: matches[1] || matches[0],
    notation: `${matches[0]}/${matches[1] || matches[0]}`,
    confidence: 0.95
  };
}
```

---

#### `phenotypeMapper.js`
**Purpose:** Map diplotypes to phenotypes

**Methods:**
```javascript
map(diplotypes) {
  const phenotypes = {};
  
  for (const [gene, diplotype] of Object.entries(diplotypes)) {
    phenotypes[gene] = mapPhenotype(gene, diplotype.notation);
  }
  
  return phenotypes;
}

function mapPhenotype(gene, notation) {
  const map = PHENOTYPE_MAPS[gene];
  const phenotype = map[notation] || { phenotype: 'Unknown Metabolizer', activity: null };
  
  return {
    gene,
    diplotype: notation,
    phenotype: phenotype.phenotype,
    activityScore: phenotype.activity
  };
}
```

---

#### `riskEngine.js`
**Purpose:** Classify drug-gene risks

**Methods:**
```javascript
classify(phenotypes, drugs) {
  const results = [];
  
  for (const drug of drugs) {
    for (const [gene, phenotype] of Object.entries(phenotypes)) {
      const risk = getRisk(drug, gene, phenotype.phenotype);
      
      results.push({
        drug,
        gene,
        risk_label: risk.label,
        confidence: phenotype.confidence || 0.9,
        diplotype: phenotype.diplotype,
        phenotype: phenotype.phenotype,
        clinical_recommendation: risk.recommendation
      });
    }
  }
  
  return results;
}

function getRisk(drug, gene, phenotype) {
  const matrix = RISK_MATRIX[drug]?.[gene]?.[phenotype];
  
  if (!matrix) {
    return {
      label: 'Unknown',
      recommendation: 'Insufficient data for this drug-gene-phenotype combination.'
    };
  }
  
  return matrix;
}
```

---

#### `llmService.js`
**Purpose:** Generate GPT-4 explanations

**Methods:**
```javascript
async explain(results) {
  const prompt = buildPrompt(results);
  
  try {
    const response = await openai.chat.completions.create({
      model: 'gpt-4',
      messages: [{ role: 'user', content: prompt }],
      temperature: 0.7,
      max_tokens: 1000
    });
    
    const explanations = JSON.parse(response.choices[0].message.content);
    
    // Merge explanations into results
    return results.map(r => ({
      ...r,
      llm_explanation: explanations.find(e => e.drug === r.drug && e.gene === r.gene)?.explanation || getFallback(r.risk_label)
    }));
    
  } catch (error) {
    // Fallback to templates
    return results.map(r => ({
      ...r,
      llm_explanation: getFallback(r.risk_label)
    }));
  }
}

function buildPrompt(results) {
  return `
You are a genetic counselor explaining pharmacogenomic results.

Analysis Results:
${results.map(r => `
- Drug: ${r.drug}
- Gene: ${r.gene}
- Diplotype: ${r.diplotype}
- Phenotype: ${r.phenotype}
- Risk: ${r.risk_label}
- Recommendation: ${r.clinical_recommendation}
`).join('\n')}

Provide a brief, plain-language explanation for each drug-gene interaction.
Keep each explanation under 150 words. Avoid medical jargon.

Format as JSON:
{
  "explanations": [
    { "drug": "...", "gene": "...", "explanation": "..." }
  ]
}
`;
}

function getFallback(riskLabel) {
  const fallbacks = {
    Safe: "Your genetic profile suggests this medication should work normally for you.",
    Adjust: "Your genes may affect how this medication works. Your doctor might adjust the dose.",
    Toxic: "Your genetic makeup creates a higher risk with this medication. Alternatives should be considered.",
    Unknown: "There isn't enough data about how your genes affect this medication."
  };
  return fallbacks[riskLabel] || fallbacks.Unknown;
}

async chat(question, results) {
  const context = results.map(r => `${r.drug} (${r.gene}): ${r.risk_label}`).join('\n');
  
  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      { role: 'system', content: 'You are a genetic counselor answering questions about pharmacogenomic results.' },
      { role: 'user', content: `Context:\n${context}\n\nQuestion: ${question}` }
    ],
    temperature: 0.7,
    max_tokens: 300
  });
  
  return response.choices[0].message.content;
}
```

---

### `/server/config/`

#### `starAlleles.js`
**Purpose:** Star allele variant patterns

```javascript
module.exports = {
  CYP2D6: {
    '*1': [], // Wild-type
    '*2': [{ rsId: 'rs16947', alt: 'A' }],
    '*4': [{ rsId: 'rs3892097', alt: 'A' }]
  },
  CYP2C19: {
    '*1': [],
    '*2': [{ rsId: 'rs4244285', alt: 'A' }]
  },
  // ... other genes
};
```

---

#### `phenotypeMaps.js`
**Purpose:** Diplotype-to-phenotype mappings

```javascript
module.exports = {
  CYP2D6: {
    '*1/*1': { phenotype: 'Normal Metabolizer', activity: 2.0 },
    '*1/*4': { phenotype: 'Intermediate Metabolizer', activity: 1.0 },
    '*4/*4': { phenotype: 'Poor Metabolizer', activity: 0.0 }
  },
  // ... other genes
};
```

---

#### `riskMatrix.js`
**Purpose:** Drug-gene-phenotype risk classifications

```javascript
module.exports = {
  CODEINE: {
    CYP2D6: {
      'Poor Metabolizer': {
        label: 'Safe',
        recommendation: 'Reduced efficacy. Consider alternative analgesic.'
      },
      'Ultrarapid Metabolizer': {
        label: 'Toxic',
        recommendation: 'Avoid codeine. High risk of toxicity.'
      },
      'Normal Metabolizer': {
        label: 'Safe',
        recommendation: 'Standard dosing recommended.'
      }
    }
  },
  WARFARIN: {
    CYP2C9: {
      'Poor Metabolizer': {
        label: 'Adjust',
        recommendation: 'Reduce dose by 25-50%. Monitor INR closely.'
      }
    }
  },
  // ... other drugs
};
```

---

### `/server/models/`

#### `Analysis.js`
**Purpose:** MongoDB schema for analysis results

```javascript
const mongoose = require('mongoose');

const AnalysisSchema = new mongoose.Schema({
  patient_id: String,
  upload_date: { type: Date, default: Date.now },
  results: [{
    drug: String,
    gene: String,
    risk_label: String,
    confidence: Number,
    diplotype: String,
    phenotype: String,
    variants: [Object],
    clinical_recommendation: String,
    llm_explanation: String
  }]
});

module.exports = mongoose.model('Analysis', AnalysisSchema);
```

---

## Key Design Decisions

### Why This Structure?

1. **Minimal Folders:** Only essential directories (pages, components, services, controllers, config)
2. **Flat Hierarchy:** Avoid deep nesting (easier to navigate in 6 hours)
3. **Clear Separation:** Frontend (client/) vs Backend (server/)
4. **Config-Driven:** Star alleles, phenotypes, risks in separate config files (easy to update)
5. **Single Responsibility:** Each file has one clear purpose
6. **No Over-Engineering:** No complex state management, no middleware layers, no abstractions

### What We Avoid:

❌ Deep folder nesting
❌ Complex state management (Redux, Zustand)
❌ Multiple API layers
❌ Extensive middleware
❌ Over-abstraction
❌ Unnecessary utilities

### What We Prioritize:

✅ Rapid development
✅ Easy debugging
✅ Clear data flow
✅ Minimal dependencies
✅ Demo readiness

---

## File Count Summary

**Frontend:** ~15 files
**Backend:** ~15 files
**Total:** ~30 files

This is manageable for a 6-hour build.
