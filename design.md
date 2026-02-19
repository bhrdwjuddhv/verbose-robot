# Design Document - PharmaGuard (6-Hour Hackathon)

## Overview

PharmaGuard is a streamlined pharmacogenomic risk prediction system built for a 6-hour hackathon. The architecture prioritizes rapid development, demo readiness, and core functionality over complex features.

## Technology Stack

**Frontend:**
- React + Vite
- Material-UI (MUI)
- React Three Fiber (3D visualization)
- Axios

**Backend:**
- Node.js + Express
- MongoDB Atlas
- OpenAI GPT-4

**Deployment:**
- Frontend: Vercel
- Backend: Render

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND (Vercel)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ GenomeUpload │  │ RiskDashboard│  │  3D Viewer   │     │
│  │    Page      │  │     Page     │  │   Component  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │           Clinical Chat Interface                     │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ REST API
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                       BACKEND (Render)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ VCF Streaming│  │ Star Allele  │  │  CPIC Risk   │     │
│  │    Parser    │  │   Resolver   │  │   Engine     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         OpenAI GPT-4 Integration                      │  │
│  │         (Single Structured Prompt)                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                      ┌──────────────┐
                      │  MongoDB     │
                      │   Atlas      │
                      └──────────────┘
```

## Frontend Pages

### 1. GenomeUpload Page

**Purpose:** File upload and drug selection

**Components:**
- **FileDropzone** (Material-UI)
  - Drag-and-drop VCF upload
  - Max 5MB validation
  - .vcf/.vcf.gz extension check
  - Upload progress indicator

- **DrugSelector** (Material-UI Chips)
  - 6 drug chips: CODEINE, WARFARIN, CLOPIDOGREL, SIMVASTATIN, AZATHIOPRINE, FLUOROURACIL
  - Multi-select with visual feedback
  - "Analyze All" default option

- **AnalyzeButton**
  - Triggers POST /api/analyze
  - Shows loading state
  - Navigates to RiskDashboard on success

**State Management:**
```javascript
{
  file: File | null,
  selectedDrugs: string[],
  uploading: boolean,
  error: string | null
}
```

---

### 2. RiskDashboard Page

**Purpose:** Display analysis results with color-coded risk cards

**Components:**
- **RiskCard** (Material-UI Card)
  - Drug name + Gene name
  - Risk label with color:
    - Green: Safe
    - Orange: Adjust
    - Red: Toxic/Ineffective
    - Purple: Unknown
  - Diplotype notation (e.g., *1/*4)
  - Phenotype (e.g., Intermediate Metabolizer)
  - Confidence score (0-1)
  - Clinical recommendation
  - LLM explanation

- **GeneFilter** (Material-UI Tabs)
  - Filter by gene: All, CYP2D6, CYP2C19, CYP2C9, SLCO1B1, TPMT, DPYD

- **ExportButton**
  - Download results as JSON

**Layout:**
```
┌─────────────────────────────────────────────────────────┐
│  Gene Filter: [All] [CYP2D6] [CYP2C19] ...             │
├─────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ CODEINE      │  │ WARFARIN     │  │ CLOPIDOGREL  │ │
│  │ CYP2D6       │  │ CYP2C9       │  │ CYP2C19      │ │
│  │ [GREEN]      │  │ [ORANGE]     │  │ [RED]        │ │
│  │ *1/*1        │  │ *1/*3        │  │ *2/*2        │ │
│  │ Normal (0.9) │  │ Intermed(0.8)│  │ Poor (0.95)  │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

### 3. 3D Gene Viewer Component

**Purpose:** Simple 3D visualization of variants

**Implementation:**
- React Three Fiber
- Simple sphere geometry for each variant
- Color-coded by risk level
- Basic orbit controls (rotate + zoom)
- No complex animations (6-hour constraint)

**Code Structure:**
```javascript
<Canvas>
  <ambientLight />
  <OrbitControls />
  {variants.map(variant => (
    <Sphere
      position={[variant.x, variant.y, variant.z]}
      color={getRiskColor(variant.risk)}
    />
  ))}
</Canvas>
```

---

### 4. Clinical Chat Interface

**Purpose:** Ask clarifying questions about results

**Components:**
- **ChatInput** (Material-UI TextField)
  - Simple text input
  - Send button

- **ChatMessages** (Material-UI List)
  - User questions
  - GPT-4 responses
  - Scrollable history

**API Integration:**
- POST /api/chat
- Sends: { question, analysisId }
- Receives: { answer }

---

## Backend Architecture

### API Endpoints

#### POST /api/upload
```javascript
// Upload VCF file
Request: multipart/form-data
Response: { uploadId, filename }
```

#### POST /api/analyze
```javascript
// Analyze VCF
Request: {
  uploadId: string,
  drugs: string[] // optional
}
Response: {
  analysisId: string,
  results: AnalysisResult[]
}
```

#### GET /api/results/:id
```javascript
// Get analysis results
Response: {
  patient_id, drug, risk_label, confidence,
  diplotype, phenotype, variants,
  clinical_recommendation, llm_explanation
}
```

#### POST /api/chat
```javascript
// Clinical chat
Request: { question, analysisId }
Response: { answer }
```

---

### Core Modules

#### 1. VCF Streaming Parser

**Purpose:** Parse VCF files efficiently

**Implementation:**
```javascript
const readline = require('readline');
const fs = require('fs');

async function parseVCF(filePath) {
  const variants = [];
  const rl = readline.createInterface({
    input: fs.createReadStream(filePath),
    crlfDelay: Infinity
  });

  for await (const line of rl) {
    if (line.startsWith('#')) continue; // Skip headers
    const variant = parseVariantLine(line);
    if (isTargetGene(variant)) {
      variants.push(variant);
    }
  }
  
  return variants;
}
```

**Why Streaming:** Handles 5MB files efficiently without loading entire file into memory.

---

#### 2. Star Allele Resolver

**Purpose:** Resolve diplotypes from variants

**Data Structure:**
```javascript
const STAR_ALLELES = {
  CYP2D6: {
    '*1': [], // Wild-type
    '*2': [{ rsId: 'rs16947', alt: 'A' }],
    '*4': [{ rsId: 'rs3892097', alt: 'A' }]
  },
  // ... other genes
};
```

**Algorithm:**
1. Match variants to star allele patterns
2. Determine diplotype combination
3. Calculate confidence based on variant coverage
4. Default to *1/*1 if no matches

---

#### 3. Phenotype Mapper

**Purpose:** Map diplotypes to phenotypes

**Data Structure:**
```javascript
const PHENOTYPE_MAP = {
  CYP2D6: {
    '*1/*1': { phenotype: 'Normal Metabolizer', activity: 2.0 },
    '*1/*4': { phenotype: 'Intermediate Metabolizer', activity: 1.0 },
    '*4/*4': { phenotype: 'Poor Metabolizer', activity: 0.0 }
  }
};
```

---

#### 4. CPIC Risk Engine

**Purpose:** Classify drug-gene risks

**Data Structure:**
```javascript
const RISK_MATRIX = {
  CODEINE: {
    CYP2D6: {
      'Poor Metabolizer': { risk: 'Safe', reason: 'Reduced efficacy' },
      'Ultrarapid Metabolizer': { risk: 'Toxic', reason: 'Toxicity risk' },
      'Normal Metabolizer': { risk: 'Safe', reason: 'Normal metabolism' }
    }
  },
  WARFARIN: {
    CYP2C9: {
      'Poor Metabolizer': { risk: 'Adjust', reason: 'Dose reduction needed' }
    }
  }
};
```

**Output:**
```javascript
{
  drug: 'CODEINE',
  gene: 'CYP2D6',
  risk_label: 'Safe',
  confidence: 0.95,
  diplotype: '*1/*1',
  phenotype: 'Normal Metabolizer',
  clinical_recommendation: 'Standard dosing recommended'
}
```

---

#### 5. OpenAI GPT-4 Integration

**Purpose:** Generate plain-language explanations

**Single Structured Prompt:**
```javascript
const prompt = `
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
Keep each explanation under 150 words.
Avoid medical jargon.

Format as JSON:
{
  "explanations": [
    { "drug": "...", "gene": "...", "explanation": "..." }
  ]
}
`;
```

**Fallback Strategy:**
```javascript
const FALLBACK_EXPLANATIONS = {
  Safe: "Your genetic profile suggests this medication should work normally for you.",
  Adjust: "Your genes may affect how this medication works. Your doctor might adjust the dose.",
  Toxic: "Your genetic makeup creates a higher risk with this medication. Alternatives should be considered.",
  Unknown: "There isn't enough data about how your genes affect this medication."
};
```

---

### MongoDB Schema

```javascript
const AnalysisSchema = new mongoose.Schema({
  patient_id: String,
  upload_date: Date,
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
```

---

## UI Theme

### Material-UI Theme Configuration

```javascript
const theme = createTheme({
  palette: {
    mode: 'dark',
    primary: {
      main: '#00BCD4', // Cyan
    },
    background: {
      default: '#0A1929',
      paper: '#132F4C',
    },
    success: { main: '#4CAF50' }, // Green (Safe)
    warning: { main: '#FF9800' }, // Orange (Adjust)
    error: { main: '#F44336' },   // Red (Toxic)
    info: { main: '#9C27B0' },    // Purple (Unknown)
  },
  typography: {
    fontFamily: 'Roboto, sans-serif',
  },
});
```

### Risk Color Mapping

```javascript
function getRiskColor(riskLabel) {
  const colors = {
    Safe: '#4CAF50',      // Green
    Adjust: '#FF9800',    // Orange
    Toxic: '#F44336',     // Red
    Ineffective: '#F44336', // Red
    Unknown: '#9C27B0'    // Purple
  };
  return colors[riskLabel] || colors.Unknown;
}
```

---

## Data Flow

### Complete Analysis Flow

```
1. User uploads VCF + selects drugs
   ↓
2. POST /api/analyze
   ↓
3. VCF Streaming Parser
   - Extract variants for 6 target genes
   ↓
4. Star Allele Resolver
   - Match variants to star alleles
   - Resolve diplotypes
   ↓
5. Phenotype Mapper
   - Map diplotypes to phenotypes
   ↓
6. CPIC Risk Engine
   - Classify drug-gene risks
   - Generate clinical recommendations
   ↓
7. OpenAI GPT-4
   - Generate plain-language explanations
   - Single structured prompt
   ↓
8. Save to MongoDB
   ↓
9. Return results to frontend
   ↓
10. Display in RiskDashboard
    - Color-coded risk cards
    - 3D visualization
    - Clinical chat available
```

**Total Processing Time:** ~15-20 seconds

---

## Simplified Decisions (6-Hour Constraint)

### What We Include:
✅ Core VCF parsing (streaming)
✅ 6 genes, 6 drugs
✅ Star allele → phenotype → risk
✅ CPIC-aligned recommendations
✅ GPT-4 explanations (single prompt)
✅ Material-UI components
✅ Simple 3D visualization
✅ Basic clinical chat
✅ MongoDB storage

### What We Exclude:
❌ Complex authentication
❌ Multi-agent systems
❌ WebSockets/real-time
❌ Advanced 3D animations
❌ Multiple LLM providers
❌ Comprehensive error recovery
❌ Result persistence beyond session
❌ Copy number variation analysis
❌ Phasing algorithms

---

## Deployment

### Frontend (Vercel)
```bash
cd client
npm run build
vercel --prod
```

### Backend (Render)
```yaml
# render.yaml
services:
  - type: web
    name: pharmaguard-api
    env: node
    buildCommand: npm install
    startCommand: npm start
    envVars:
      - key: MONGODB_URI
      - key: OPENAI_API_KEY
```

### Environment Variables

**Frontend (.env):**
```
VITE_API_URL=https://pharmaguard-api.onrender.com
```

**Backend (.env):**
```
PORT=3000
MONGODB_URI=mongodb+srv://...
OPENAI_API_KEY=sk-...
CORS_ORIGIN=https://pharmaguard.vercel.app
```

---

## Success Criteria

1. ✅ Upload 5MB VCF file
2. ✅ Select drugs to analyze
3. ✅ See color-coded risk cards
4. ✅ View diplotypes + phenotypes
5. ✅ Read GPT-4 explanations
6. ✅ Interact with 3D viewer
7. ✅ Ask questions via chat
8. ✅ Complete analysis in <30 seconds
9. ✅ Deploy to Vercel + Render
10. ✅ Demo-ready presentation
