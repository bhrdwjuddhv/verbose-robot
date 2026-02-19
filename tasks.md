# Implementation Plan - PharmaGuard (6-Hour Hackathon)

## Overview

This is a **6-hour execution plan** for building PharmaGuard. Tasks are prioritized for rapid development and demo readiness.

---

## Hour 0-2: Foundation & Core Logic

### Priority 1: Project Setup (30 min)

- [ ] 1.1 Initialize frontend with Vite + React
  ```bash
  npm create vite@latest client -- --template react
  cd client && npm install @mui/material @emotion/react @emotion/styled
  npm install @react-three/fiber @react-three/drei three
  npm install axios
  ```

- [ ] 1.2 Initialize backend with Express
  ```bash
  mkdir server && cd server
  npm init -y
  npm install express mongoose cors dotenv multer
  npm install openai
  ```

- [ ] 1.3 Setup MongoDB Atlas
  - Create cluster
  - Get connection string
  - Add to .env

- [ ] 1.4 Configure Material-UI dark theme
  - Create theme.js with cyan primary + dark mode
  - Setup risk color palette

---

### Priority 2: VCF Parser (45 min)

- [ ] 2.1 Create VCF streaming parser
  - Use Node.js readline
  - Parse variant lines
  - Extract: chr, pos, ref, alt, rsId

- [ ] 2.2 Implement target gene filter
  - Filter for 6 genes: CYP2D6, CYP2C19, CYP2C9, SLCO1B1, TPMT, DPYD
  - Use genomic coordinates

- [ ] 2.3 Test with sample VCF
  - Create sample_test.vcf
  - Verify variant extraction

---

### Priority 3: Star Allele Resolution (45 min)

- [ ] 3.1 Define star allele patterns
  - Create starAlleles.js config
  - Define patterns for 6 genes
  - Focus on common alleles (*1, *2, *3, *4)

- [ ] 3.2 Implement matching algorithm
  - Match variants to star allele patterns
  - Resolve diplotype combinations
  - Calculate confidence score

- [ ] 3.3 Implement wild-type default
  - Assign *1/*1 if no variants found

---

## Hour 2-4: Risk Engine & AI

### Priority 4: Phenotype Mapping (30 min)

- [ ] 4.1 Create phenotype mapping tables
  - Define CPIC phenotype maps for 6 genes
  - Map diplotypes to metabolizer phenotypes

- [ ] 4.2 Implement mapper function
  - Lookup diplotype → phenotype
  - Return activity score

---

### Priority 5: CPIC Risk Engine (45 min)

- [ ] 5.1 Define drug-gene risk matrix
  - Create riskMatrix.js config
  - Define risks for 6 drugs × 6 genes
  - Map phenotypes to risk labels (Safe/Adjust/Toxic/Unknown)

- [ ] 5.2 Implement risk classifier
  - Lookup phenotype → risk label
  - Generate clinical recommendations
  - Calculate confidence

- [ ] 5.3 Test risk classification
  - Verify CPIC alignment
  - Test all 6 drugs

---

### Priority 6: OpenAI GPT-4 Integration (45 min)

- [ ] 6.1 Setup OpenAI client
  - Initialize with API key
  - Configure GPT-4 model

- [ ] 6.2 Create structured prompt
  - Single prompt for all drug-gene pairs
  - Request JSON format output
  - Include: drug, gene, diplotype, phenotype, risk, recommendation

- [ ] 6.3 Implement fallback explanations
  - Template-based fallbacks
  - Handle API failures gracefully

- [ ] 6.4 Test explanation generation
  - Verify output format
  - Test fallback mechanism

---

## Hour 4-5: Frontend UI

### Priority 7: GenomeUpload Page (30 min)

- [ ] 7.1 Create FileDropzone component
  - Material-UI dropzone
  - Drag-and-drop support
  - File validation (5MB, .vcf/.vcf.gz)

- [ ] 7.2 Create DrugSelector component
  - 6 drug chips (Material-UI)
  - Multi-select functionality
  - Visual feedback

- [ ] 7.3 Implement upload flow
  - POST /api/upload
  - Show progress
  - Navigate to results on success

---

### Priority 8: RiskDashboard Page (45 min)

- [ ] 8.1 Create RiskCard component
  - Material-UI Card
  - Color-coded by risk label
  - Display: drug, gene, diplotype, phenotype, confidence
  - Show clinical recommendation
  - Show LLM explanation

- [ ] 8.2 Implement gene filter
  - Material-UI Tabs
  - Filter by gene

- [ ] 8.3 Layout risk cards
  - Grid layout
  - Responsive design

- [ ] 8.4 Add export functionality
  - Download results as JSON

---

### Priority 9: 3D Gene Viewer (30 min)

- [ ] 9.1 Setup React Three Fiber canvas
  - Basic scene with lighting
  - Orbit controls

- [ ] 9.2 Render variant spheres
  - Map variants to 3D positions
  - Color-code by risk level
  - Simple sphere geometry

- [ ] 9.3 Add basic interactions
  - Rotate + zoom
  - No complex animations (time constraint)

---

### Priority 10: Clinical Chat (15 min)

- [ ] 10.1 Create ChatInterface component
  - Material-UI TextField for input
  - Material-UI List for messages

- [ ] 10.2 Implement chat API
  - POST /api/chat
  - Send question + analysisId
  - Display GPT-4 response

---

## Hour 5-6: Integration & Deployment

### Priority 11: Backend API Integration (30 min)

- [ ] 11.1 Create API endpoints
  - POST /api/upload (file upload)
  - POST /api/analyze (run analysis)
  - GET /api/results/:id (get results)
  - POST /api/chat (clinical chat)

- [ ] 11.2 Implement MongoDB storage
  - Save analysis results
  - Query by analysisId

- [ ] 11.3 Add CORS configuration
  - Allow frontend origin

---

### Priority 12: Testing & Bug Fixes (20 min)

- [ ] 12.1 Test complete flow
  - Upload VCF
  - Select drugs
  - View results
  - Check 3D viewer
  - Test chat

- [ ] 12.2 Fix critical bugs
  - Focus on demo-breaking issues only

- [ ] 12.3 Verify color coding
  - Green (Safe)
  - Orange (Adjust)
  - Red (Toxic/Ineffective)
  - Purple (Unknown)

---

### Priority 13: Deployment (40 min)

- [ ] 13.1 Deploy frontend to Vercel
  ```bash
  cd client
  npm run build
  vercel --prod
  ```

- [ ] 13.2 Deploy backend to Render
  - Create web service
  - Connect GitHub repo
  - Add environment variables
  - Deploy

- [ ] 13.3 Configure environment variables
  - Frontend: VITE_API_URL
  - Backend: MONGODB_URI, OPENAI_API_KEY, CORS_ORIGIN

- [ ] 13.4 Test deployed application
  - Upload test VCF
  - Verify end-to-end flow
  - Check CORS

---

### Priority 14: Demo Preparation (10 min)

- [ ] 14.1 Create sample VCF file
  - Include variants for demo
  - Test all 6 drugs

- [ ] 14.2 Prepare demo script
  - Upload flow
  - Drug selection
  - Results visualization
  - 3D viewer
  - Clinical chat

- [ ] 14.3 Take screenshots
  - For presentation/documentation

---

## Time Allocation Summary

| Phase | Time | Tasks |
|-------|------|-------|
| Setup & Core Logic | 2h | VCF parser, star allele resolution |
| Risk Engine & AI | 2h | Phenotype mapping, CPIC risk, GPT-4 |
| Frontend UI | 1h | Upload page, dashboard, 3D viewer, chat |
| Integration & Deploy | 1h | API integration, testing, deployment |

---

## Critical Path

**Must Complete for Demo:**
1. ✅ VCF parsing (streaming)
2. ✅ Star allele → phenotype → risk
3. ✅ GPT-4 explanations
4. ✅ Color-coded risk cards
5. ✅ Basic 3D visualization
6. ✅ Deploy to Vercel + Render

**Nice to Have (if time permits):**
- Clinical chat
- Advanced 3D interactions
- Export functionality
- Gene filtering

---

## Contingency Plans

### If Behind Schedule:

**Cut at Hour 4:**
- Skip 3D viewer (use static images)
- Skip clinical chat
- Focus on core risk dashboard

**Cut at Hour 5:**
- Skip MongoDB (use in-memory storage)
- Skip export functionality
- Simplify UI styling

**Minimum Viable Demo:**
1. Upload VCF
2. Show color-coded risk cards
3. Display diplotypes + phenotypes
4. Show GPT-4 explanations
5. Deploy to Vercel + Render

---

## Success Metrics

- [ ] Complete analysis in <30 seconds
- [ ] Display results for all 6 drugs
- [ ] Show color-coded risk levels
- [ ] Generate GPT-4 explanations
- [ ] Deploy successfully
- [ ] Demo runs smoothly

---

## Notes

- **Focus on demo readiness over perfection**
- **Use Material-UI components (faster than custom CSS)**
- **Single GPT-4 prompt (avoid multiple API calls)**
- **Streaming VCF parser (handles 5MB efficiently)**
- **Hardcode CPIC data (no external API calls)**
- **Test frequently (catch bugs early)**
