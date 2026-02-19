# PharmaGuard – Pharmacogenomic Risk Prediction System
## File Structure Documentation

**RIFT 2026 Hackathon Submission**

---

## Table of Contents

1. [High-Level Project Structure](#high-level-project-structure)
2. [Frontend Architecture](#frontend-architecture)
3. [Backend Architecture](#backend-architecture)
4. [Core Processing Modules](#core-processing-modules)
5. [API Structure](#api-structure)
6. [Configuration Files](#configuration-files)
7. [Separation of Concerns](#separation-of-concerns)

---

## High-Level Project Structure

```
pharmaguard/
├── frontend/                    # React + Vite frontend application
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── .env
│   └── .env.example
│
├── backend/                     # Node.js + Express backend API
│   ├── server.js
│   ├── routes/
│   ├── controllers/
│   ├── services/
│   ├── modules/
│   ├── middleware/
│   ├── validators/
│   ├── utils/
│   ├── config/
│   ├── test-data/
│   ├── uploads/
│   ├── package.json
│   ├── .env
│   └── .env.example
│
├── file-structure.md            # This document
└── README.md                    # Setup and deployment guide
```


### Architecture Overview

**Monorepo Structure:** The project uses a monorepo approach with clearly separated frontend and backend directories. This separation ensures:
- Independent deployment to Vercel (frontend) and Render (backend)
- Clear API contract boundaries
- Isolated dependency management
- Simplified CORS configuration

**Why This Structure:** This architecture prevents disqualification by maintaining strict separation between client and server logic, ensuring JSON schema compliance is validated server-side before any response is sent to the client.

---

## Frontend Architecture

### Complete Frontend Structure

```
frontend/
├── src/
│   ├── components/              # Reusable React components
│   │   ├── FileUpload.jsx       # VCF file upload interface
│   │   ├── ResultsDisplay.jsx   # Main results visualization
│   │   ├── GeneCard.jsx         # Individual gene result card
│   │   ├── InteractionCard.jsx  # Drug-gene interaction display
│   │   ├── ExplanationModal.jsx # Detailed explanation popup
│   │   ├── LoadingSpinner.jsx   # Processing indicator
│   │   └── ErrorMessage.jsx     # Error display component
│   │
│   ├── pages/                   # Page-level components
│   │   ├── HomePage.jsx         # Landing page with upload
│   │   └── ResultsPage.jsx      # Results visualization page
│   │
│   ├── services/                # API communication layer
│   │   └── api.js               # Axios-based API client
│   │
│   ├── utils/                   # Frontend utilities
│   │   ├── colorMapping.js      # Risk level to color conversion
│   │   └── validators.js        # Client-side file validation
│   │
│   ├── hooks/                   # Custom React hooks
│   │   ├── useFileUpload.js     # File upload state management
│   │   └── useAnalysis.js       # Analysis polling logic
│   │
│   ├── styles/                  # Global styles
│   │   └── index.css            # TailwindCSS imports + custom styles
│   │
│   ├── App.jsx                  # Root application component
│   └── main.jsx                 # Vite entry point
│
├── public/                      # Static assets
│   └── favicon.ico
│
├── package.json                 # Frontend dependencies
├── vite.config.js               # Vite build configuration
├── tailwind.config.js           # TailwindCSS configuration
├── .env                         # Environment variables (gitignored)
└── .env.example                 # Environment template
```


### Frontend Component Breakdown

#### `src/components/FileUpload.jsx`
**Purpose:** Handles VCF file selection and upload to backend  
**Responsibility:**
- File input interface with drag-and-drop support
- Client-side validation (extension: .vcf/.vcf.gz, size: <50MB)
- Upload progress tracking
- Error message display for invalid files

**Why It Exists:** Provides immediate user feedback before server processing, reducing unnecessary API calls and improving UX during demo.

---

#### `src/components/ResultsDisplay.jsx`
**Purpose:** Main container for displaying pharmacogenomic analysis results  
**Responsibility:**
- Receives validated JSON from backend
- Organizes results by gene (CYP2D6, CYP2C19, CYP2C9, VKORC1, SLCO1B1, DPYD)
- Implements expandable sections for each gene
- Coordinates child components (GeneCard, InteractionCard)

**Why It Exists:** Centralizes result rendering logic and ensures consistent display of hackathon-required JSON structure.

---

#### `src/components/GeneCard.jsx`
**Purpose:** Displays diplotype and phenotype information for a single gene  
**Responsibility:**
- Shows gene name, diplotype notation (e.g., *1/*4)
- Displays metabolizer phenotype (Poor, Intermediate, Normal, Rapid, Ultrarapid)
- Shows CPIC activity score
- Provides visual hierarchy for gene-level information

**Why It Exists:** Modularizes gene-specific display logic, making the UI maintainable and demo-ready.

---

#### `src/components/InteractionCard.jsx`
**Purpose:** Displays individual drug-gene interaction with color-coded risk level  
**Responsibility:**
- Shows drug name and associated gene
- Displays risk level with color coding:
  - **Red:** high_risk
  - **Yellow:** moderate_risk
  - **Green:** low_risk
  - **Gray:** no_known_interaction
- Shows phenotype and CPIC evidence level
- Clickable to open ExplanationModal

**Why It Exists:** Implements the critical color-coding requirement from hackathon evaluation criteria. Ensures judges can quickly assess risk visualization.

---

#### `src/components/ExplanationModal.jsx`
**Purpose:** Popup modal displaying detailed LLM-generated explanations and CPIC recommendations  
**Responsibility:**
- Shows plain-language explanation of drug-gene interaction
- Displays CPIC-aligned clinical recommendation
- Shows recommendation strength (Strong, Moderate, Optional)
- Lists alternative medications if applicable
- Provides close/dismiss functionality

**Why It Exists:** Fulfills the "Explainable AI" requirement by presenting LLM-generated content in an accessible format for non-technical users.

---

#### `src/components/LoadingSpinner.jsx`
**Purpose:** Visual indicator during file upload and analysis processing  
**Responsibility:**
- Animated spinner with processing status text
- Estimated time remaining display
- Prevents user confusion during backend processing

**Why It Exists:** Ensures smooth demo experience by providing feedback during the 15-30 second processing window.

---

#### `src/components/ErrorMessage.jsx`
**Purpose:** Displays user-friendly error messages  
**Responsibility:**
- Shows error messages from backend (invalid file, parsing errors, etc.)
- Provides actionable guidance (e.g., "Please upload a .vcf file")
- Dismissible alert component

**Why It Exists:** Handles error states gracefully, preventing demo failures and ensuring judges see professional error handling.

---

### Frontend Services

#### `src/services/api.js`
**Purpose:** Centralized API communication layer using Axios  
**Responsibility:**
- Configures Axios instance with backend base URL
- Implements three API methods:
  - `uploadVCF(file)` → POST /api/upload
  - `analyzeVCF(uploadId, patientId)` → POST /api/analyze
  - `getResults(analysisId)` → GET /api/results/:id
- Handles HTTP errors and network failures
- Adds request/response interceptors for logging

**Why It Exists:** Abstracts API communication, making it easy to switch backend URLs between development and production (Vercel → Render).

---

### Frontend Utilities

#### `src/utils/colorMapping.js`
**Purpose:** Maps risk levels to TailwindCSS color classes  
**Responsibility:**
- Exports `getRiskColor(riskLevel)` function
- Returns color classes:
  - `high_risk` → `bg-red-100 border-red-500 text-red-900`
  - `moderate_risk` → `bg-yellow-100 border-yellow-500 text-yellow-900`
  - `low_risk` → `bg-green-100 border-green-500 text-green-900`
  - `no_known_interaction` → `bg-gray-100 border-gray-500 text-gray-900`

**Why It Exists:** Centralizes color logic to ensure consistent risk visualization across all components. Critical for evaluation criteria.

---

#### `src/utils/validators.js`
**Purpose:** Client-side file validation before upload  
**Responsibility:**
- `validateFileExtension(filename)` → checks .vcf or .vcf.gz
- `validateFileSize(file)` → checks <50MB limit
- Returns validation errors with user-friendly messages

**Why It Exists:** Provides immediate feedback to users, reducing unnecessary backend calls and improving demo responsiveness.

---

### Frontend Hooks

#### `src/hooks/useFileUpload.js`
**Purpose:** Custom hook managing file upload state  
**Responsibility:**
- Manages state: `selectedFile`, `uploading`, `uploadProgress`, `uploadError`
- Implements `handleFileSelect()` and `handleUpload()` methods
- Calls `api.uploadVCF()` and handles responses

**Why It Exists:** Encapsulates upload logic for reuse across components, following React best practices.

---

#### `src/hooks/useAnalysis.js`
**Purpose:** Custom hook managing analysis polling and results fetching  
**Responsibility:**
- Polls `api.getResults()` every 2 seconds until analysis completes
- Manages state: `results`, `loading`, `error`
- Implements automatic retry logic

**Why It Exists:** Handles asynchronous analysis workflow, ensuring results display as soon as backend processing completes.

---

## Backend Architecture

### Complete Backend Structure

```
backend/
├── server.js                    # Express app entry point
│
├── routes/
│   └── api.js                   # API route definitions
│
├── controllers/
│   ├── uploadController.js      # Handles file upload requests
│   ├── analysisController.js    # Orchestrates analysis pipeline
│   └── resultsController.js     # Retrieves analysis results
│
├── services/
│   └── processingPipeline.js    # Coordinates all processing modules
│
├── modules/                     # Core pharmacogenomic logic
│   ├── vcfParser.js             # VCF file parsing
│   ├── geneFilter.js            # Target gene filtering
│   ├── diplotypeResolver.js     # Star allele diplotype resolution
│   ├── phenotypeMapper.js       # Diplotype-to-phenotype mapping
│   ├── drugRiskEngine.js        # Drug-gene risk classification
│   ├── cpicRecommendationEngine.js  # CPIC recommendation generation
│   └── llmExplanationService.js # LLM-powered explanations
│
├── validators/
│   └── jsonSchemaValidator.js   # Zod-based output validation
│
├── middleware/
│   ├── errorHandler.js          # Centralized error handling
│   ├── fileUploadMiddleware.js  # Multer configuration
│   └── corsMiddleware.js        # CORS configuration
│
├── utils/
│   ├── logger.js                # Logging utility
│   ├── fileCleanup.js           # Temporary file cleanup
│   └── pathSanitizer.js         # Security: path sanitization
│
├── config/
│   ├── starAlleles.js           # Star allele definitions
│   ├── phenotypeMaps.js         # Diplotype-phenotype mappings
│   ├── drugRiskMatrix.js        # Drug-gene risk classifications
│   └── cpicRecommendations.js   # CPIC recommendation templates
│
├── test-data/                   # Sample VCF files for testing
│   ├── sample_wildtype.vcf
│   ├── sample_cyp2d6_poor.vcf
│   ├── sample_cyp2d6_ultra.vcf
│   ├── sample_mixed.vcf
│   └── sample_malformed.vcf
│
├── uploads/                     # Temporary file storage (gitignored)
│   └── temp/
│
├── package.json                 # Backend dependencies
├── .env                         # Environment variables (gitignored)
└── .env.example                 # Environment template
```


### Backend Core Files

#### `server.js`
**Purpose:** Express application entry point and server initialization  
**Responsibility:**
- Initializes Express app
- Configures middleware (CORS, body-parser, file upload)
- Mounts API routes
- Starts HTTP server on configured port
- Implements graceful shutdown for file cleanup

**Why It Exists:** Central configuration point ensuring all middleware and routes are properly initialized before accepting requests.

---

### Backend Routes

#### `routes/api.js`
**Purpose:** Defines all API endpoints and maps them to controllers  
**Responsibility:**
- `POST /api/upload` → `uploadController.handleUpload`
- `POST /api/analyze` → `analysisController.startAnalysis`
- `GET /api/results/:id` → `resultsController.getResults`
- Applies middleware (authentication, validation) to routes

**Why It Exists:** Centralizes route definitions, making API structure clear for judges reviewing code and ensuring consistent endpoint naming.

---

### Backend Controllers

#### `controllers/uploadController.js`
**Purpose:** Handles VCF file upload requests  
**Responsibility:**
- Receives multipart/form-data file upload
- Validates file extension (.vcf, .vcf.gz)
- Validates file size (<50MB)
- Generates unique upload ID (UUID)
- Stores file in temporary directory
- Returns upload confirmation with ID

**Why It Exists:** Isolates upload logic from processing logic, enabling independent testing and ensuring upload validation happens before any processing begins.

---

#### `controllers/analysisController.js`
**Purpose:** Orchestrates the complete pharmacogenomic analysis pipeline  
**Responsibility:**
- Receives analysis request with upload ID
- Validates upload ID exists
- Calls `processingPipeline.execute(uploadId, patientId)`
- Generates unique analysis ID
- Returns analysis status with estimated time
- Handles processing errors gracefully

**Why It Exists:** Acts as the orchestration layer, coordinating all processing modules while maintaining clean separation of concerns.

---

#### `controllers/resultsController.js`
**Purpose:** Retrieves and returns analysis results  
**Responsibility:**
- Receives GET request with analysis ID
- Retrieves results from in-memory storage or cache
- Returns 404 if analysis not found
- Returns 200 with complete JSON results
- Ensures JSON schema validation before response

**Why It Exists:** Separates result retrieval from processing, enabling independent caching strategies and ensuring results are always schema-validated before delivery.

---

### Backend Services

#### `services/processingPipeline.js`
**Purpose:** Coordinates execution of all pharmacogenomic processing modules  
**Responsibility:**
- Executes modules in sequence:
  1. VCF Parser → extract variants
  2. Gene Filter → filter target genes
  3. Diplotype Resolver → resolve star alleles
  4. Phenotype Mapper → map to metabolizer phenotypes
  5. Drug Risk Engine → classify drug-gene risks
  6. CPIC Recommendation Engine → generate recommendations
  7. LLM Explanation Service → generate explanations
  8. JSON Schema Validator → validate output
- Implements error handling at each step
- Logs processing progress
- Returns validated JSON output

**Why It Exists:** This is the **critical orchestration layer** that ensures:
- All modules execute in correct order
- Data flows correctly between modules
- JSON schema validation happens before any response
- Errors are caught and handled gracefully
- **Prevents disqualification** by guaranteeing schema compliance

---

## Core Processing Modules

### `modules/vcfParser.js`

**Purpose:** Parse VCF files and extract genomic variants  
**Input:** File path to uploaded VCF file  
**Output:** Array of variant objects

```javascript
{
  chromosome: string,
  position: number,
  refAllele: string,
  altAllele: string,
  gene: string,
  rsId: string | null,
  quality: number | null,
  filter: string
}
```

**Internal Responsibility:**
- Uses `@gmod/vcf` library to parse VCF format
- Extracts variant records from VCF body
- Handles both .vcf and .vcf.gz formats
- Gracefully handles malformed records (logs error, continues processing)
- Extracts chromosome, position, REF, ALT, QUAL, FILTER fields

**Why It's Separated:** VCF parsing is complex and error-prone. Isolating it enables:
- Independent testing with various VCF formats
- Easy debugging of parsing issues
- Reusability across different analysis workflows
- Clear error boundaries (parsing vs. interpretation)

---

### `modules/geneFilter.js`

**Purpose:** Filter variants to only include six target pharmacogenomic genes  
**Input:** Array of all variants from VCF parser  
**Output:** Array of variants filtered to target genes

**Target Genes:**
- CYP2D6 (chromosome 22)
- CYP2C19 (chromosome 10)
- CYP2C9 (chromosome 10)
- VKORC1 (chromosome 16)
- SLCO1B1 (chromosome 12)
- DPYD (chromosome 1)

**Internal Responsibility:**
- Maps genomic positions to gene names
- Filters variants by chromosome and position ranges
- Annotates variants with gene names
- Returns only variants within target gene boundaries

**Why It's Separated:** Gene filtering is a distinct bioinformatics operation. Separation enables:
- Easy modification of target gene list
- Performance optimization (early filtering reduces downstream processing)
- Clear data transformation boundary
- Independent testing with known gene coordinates

---

### `modules/diplotypeResolver.js`

**Purpose:** Resolve diplotypes from variants using star allele nomenclature  
**Input:** Array of variants filtered to target genes  
**Output:** Map of gene names to diplotype objects

```javascript
{
  gene: string,
  allele1: string,      // e.g., "*1"
  allele2: string,      // e.g., "*4"
  notation: string,     // e.g., "*1/*4"
  confidence: string    // "high" | "medium" | "low"
}
```

**Internal Responsibility:**
- Loads star allele definitions from `config/starAlleles.js`
- For each target gene:
  - Matches variants against known star allele patterns
  - Determines most likely diplotype combination
  - Assigns confidence level based on variant coverage
- Defaults to *1/*1 (wild-type) if no variants found
- Handles ambiguous cases with confidence scoring

**Why It's Separated:** Diplotype resolution is the **core pharmacogenomic logic**. Separation enables:
- Independent validation against PharmVar database
- Easy updates when new star alleles are discovered
- Clear testing with known variant-diplotype mappings
- Isolation of complex matching algorithms
- **Critical for correctness** - this module determines all downstream results

---

### `modules/phenotypeMapper.js`

**Purpose:** Map diplotypes to metabolizer phenotypes using CPIC guidelines  
**Input:** Map of gene names to diplotype objects  
**Output:** Map of gene names to phenotype objects

```javascript
{
  gene: string,
  diplotype: string,
  phenotype: string,           // "Poor Metabolizer", "Intermediate Metabolizer", etc.
  activityScore: number | null // CPIC activity score (0.0 - 3.0)
}
```

**Internal Responsibility:**
- Loads phenotype mapping tables from `config/phenotypeMaps.js`
- For each gene-diplotype pair:
  - Looks up corresponding phenotype
  - Retrieves CPIC activity score
  - Assigns "Unknown Metabolizer" if mapping not found
- Ensures all six target genes have phenotype assignments

**Why It's Separated:** Phenotype mapping is **data-driven and CPIC-specific**. Separation enables:
- Easy updates when CPIC guidelines change
- Independent validation against CPIC tables
- Clear distinction between genetic data (diplotypes) and functional interpretation (phenotypes)
- Simplified testing with known diplotype-phenotype pairs

---

### `modules/drugRiskEngine.js`

**Purpose:** Classify drug-gene interaction risks based on phenotypes  
**Input:** Map of gene names to phenotype objects  
**Output:** Array of drug-gene interaction objects

```javascript
{
  gene: string,
  drug: string,
  phenotype: string,
  riskLevel: string,    // "high_risk" | "moderate_risk" | "low_risk" | "no_known_interaction"
  cpicLevel: string     // CPIC evidence level: "A" | "B" | "C" | "D"
}
```

**Internal Responsibility:**
- Loads drug-gene risk matrix from `config/drugRiskMatrix.js`
- For each gene-phenotype pair:
  - Identifies relevant drugs (10-15 per gene)
  - Looks up risk level based on phenotype
  - Assigns CPIC evidence level
  - Defaults to "no_known_interaction" if not in matrix
- Returns comprehensive list of all drug-gene interactions

**Why It's Separated:** Risk classification is **clinically critical and evidence-based**. Separation enables:
- Independent validation against CPIC guidelines
- Easy addition of new drugs as evidence emerges
- Clear distinction between phenotype (genetic function) and risk (clinical impact)
- Simplified testing with known phenotype-risk mappings
- **Critical for demo** - this determines which interactions are highlighted

---

### `modules/cpicRecommendationEngine.js`

**Purpose:** Generate CPIC-aligned clinical recommendations for drug-gene interactions  
**Input:** Array of drug-gene interaction objects  
**Output:** Array of recommendation objects

```javascript
{
  gene: string,
  drug: string,
  riskLevel: string,
  recommendation: string,      // Clinical action text
  strength: string,            // "Strong" | "Moderate" | "Optional"
  cpicGuideline: string,       // Reference to CPIC guideline version
  alternatives: string[]       // Alternative medications
}
```

**Internal Responsibility:**
- Loads recommendation templates from `config/cpicRecommendations.js`
- For each drug-gene interaction:
  - Retrieves appropriate recommendation based on risk level and phenotype
  - Assigns recommendation strength (Strong, Moderate, Optional)
  - Includes CPIC guideline reference
  - Lists alternative medications for high-risk interactions
- Ensures all interactions receive recommendations

**Why It's Separated:** Recommendations are **guideline-based and template-driven**. Separation enables:
- Independent validation against published CPIC guidelines
- Easy updates when guidelines are revised
- Clear distinction between risk classification and clinical action
- Simplified testing with known interaction-recommendation pairs
- **Critical for evaluation** - judges will assess CPIC alignment

---

### `modules/llmExplanationService.js`

**Purpose:** Generate plain-language explanations using LLM (OpenAI or Gemini)  
**Input:** Array of drug-gene interactions and recommendations  
**Output:** Map of interaction keys to explanation strings

```javascript
{
  drug_name: string,
  gene_name: string,
  explanation: string    // Plain-language explanation (<150 words)
}
```

**Internal Responsibility:**
- Initializes OpenAI or Gemini SDK with API key
- For each drug-gene interaction:
  - Constructs prompt with gene, drug, phenotype, risk level, recommendation
  - Calls LLM API with 10-second timeout
  - Extracts plain-language explanation from response
  - Falls back to template-based explanation if API fails
- Implements retry logic for transient failures
- Ensures all interactions receive explanations (never returns empty)

**Why It's Separated:** LLM integration is **external, asynchronous, and failure-prone**. Separation enables:
- Easy switching between OpenAI and Gemini
- Independent testing with mocked LLM responses
- Graceful fallback without affecting other modules
- Clear boundary between deterministic logic and AI-generated content
- **Critical for demo stability** - LLM failures must not crash the system

**Fallback Strategy:** If LLM API fails, uses template-based explanations:
```javascript
"Your genetic makeup affects how your body processes [drug]. 
This creates a [risk_level] risk. [recommendation]"
```

---

### `validators/jsonSchemaValidator.js`

**Purpose:** Validate final output against hackathon JSON schema using Zod  
**Input:** Complete analysis result object  
**Output:** Validation result with errors (if any)

**Zod Schema Definition:**
```javascript
{
  patient_id: string,
  analysis_date: string (ISO 8601),
  genes: array of gene objects,
  drug_interactions: array of interaction objects,
  recommendations: array of recommendation objects,
  explanations: array of explanation objects
}
```

**Internal Responsibility:**
- Defines complete Zod schema matching hackathon requirements
- Validates output structure before response
- Checks all required fields are present
- Validates data types (strings, numbers, enums)
- Validates enum values (risk levels, phenotypes, strengths)
- Returns detailed validation errors if schema fails

**Why It's Separated:** Schema validation is the **final gatekeeper preventing disqualification**. Separation enables:
- Independent testing with valid and invalid outputs
- Clear validation error messages for debugging
- Easy schema updates if hackathon requirements change
- **Absolute guarantee** that only valid JSON is returned to judges
- Single source of truth for output structure

**Critical Rule:** If validation fails, the system returns a 500 error rather than invalid JSON. This prevents disqualification.

---

### Backend Middleware

#### `middleware/errorHandler.js`
**Purpose:** Centralized error handling for all API endpoints  
**Responsibility:**
- Catches all errors from controllers and modules
- Logs errors with timestamp, component name, error details
- Translates technical errors to user-friendly messages
- Returns structured error responses:
  ```javascript
  {
    error_code: string,
    message: string,
    details: string (optional, only in development)
  }
  ```
- Ensures appropriate HTTP status codes (400, 404, 500)
- Never exposes stack traces or internal details to clients

**Why It Exists:** Centralized error handling ensures:
- Consistent error response format
- Comprehensive error logging for debugging
- Professional error messages for judges
- **Demo stability** - no crashes, always graceful degradation

---

#### `middleware/fileUploadMiddleware.js`
**Purpose:** Configures Multer for secure file uploads  
**Responsibility:**
- Configures upload destination: `uploads/temp/`
- Sets file size limit: 50MB
- Implements file filter: only .vcf and .vcf.gz extensions
- Generates unique filenames to prevent collisions
- Rejects invalid files before they reach controllers

**Why It Exists:** File upload security is critical. This middleware:
- Prevents denial-of-service attacks (file size limits)
- Prevents malicious file uploads (extension validation)
- Isolates upload configuration from business logic
- **Prevents disqualification** by rejecting invalid files early

---

#### `middleware/corsMiddleware.js`
**Purpose:** Configures CORS for cross-origin requests from frontend  
**Responsibility:**
- Allows requests from Vercel frontend domain
- Configures allowed methods: GET, POST
- Configures allowed headers: Content-Type, Authorization
- Enables credentials if needed

**Why It Exists:** CORS configuration is **deployment-critical**. This middleware:
- Enables frontend-backend communication in production
- Prevents CORS errors during demo
- Centralizes origin whitelist for easy updates
- **Essential for demo** - without proper CORS, frontend cannot communicate with backend

---

### Backend Utilities

#### `utils/logger.js`
**Purpose:** Centralized logging utility  
**Responsibility:**
- Implements log levels: ERROR, WARN, INFO, DEBUG
- Formats log messages with timestamps
- Logs to console (development) and file (production)
- Filters PII from log messages (patient names, SSNs, etc.)
- Provides structured logging for error tracking

**Why It Exists:** Comprehensive logging enables:
- Debugging during development
- Error tracking in production
- Compliance with privacy requirements (no PII in logs)
- **Demo troubleshooting** - if something fails, logs reveal why

---

#### `utils/fileCleanup.js`
**Purpose:** Automatic cleanup of temporary uploaded files  
**Responsibility:**
- Deletes uploaded VCF files after processing completes
- Implements 1-hour retention policy (files deleted after 1 hour)
- Cleans up partial files if upload fails
- Runs cleanup job every 15 minutes

**Why It Exists:** File cleanup is **security and resource management**. This utility:
- Prevents disk space exhaustion
- Protects patient privacy (genomic data not retained)
- Handles failed uploads gracefully
- **Compliance requirement** - genomic data must not be stored long-term

---

#### `utils/pathSanitizer.js`
**Purpose:** Security utility to prevent directory traversal attacks  
**Responsibility:**
- Validates file paths before file operations
- Rejects paths containing `../`, `..\\`, or absolute paths
- Ensures all file operations stay within `uploads/` directory
- Returns sanitized paths or throws security error

**Why It Exists:** Path sanitization is **critical security**. This utility:
- Prevents attackers from accessing files outside upload directory
- Protects server filesystem
- **Prevents disqualification** - security vulnerabilities could fail evaluation

---

### Backend Configuration Files

#### `config/starAlleles.js`
**Purpose:** Star allele definitions for six target genes  
**Content:** Hardcoded star allele variant patterns based on PharmVar database

**Example Structure:**
```javascript
const CYP2D6_ALLELES = {
  "*1": [],  // Wild-type (no variants)
  "*2": [{ rsId: "rs16947", position: 2850, alt: "A" }],
  "*3": [{ rsId: "rs35742686", position: 2549, alt: "delA" }],
  "*4": [{ rsId: "rs3892097", position: 1846, alt: "A" }],
  // ... additional alleles
}
```

**Why It Exists:** Star allele definitions are **reference data**. Separating them:
- Makes diplotype resolver logic cleaner
- Enables easy updates when PharmVar releases new alleles
- Provides single source of truth for variant-allele mappings
- Simplifies testing with known allele patterns

---

#### `config/phenotypeMaps.js`
**Purpose:** Diplotype-to-phenotype mapping tables for six genes  
**Content:** CPIC-aligned phenotype assignments with activity scores

**Example Structure:**
```javascript
const CYP2D6_PHENOTYPE_MAP = {
  "*1/*1": { phenotype: "Normal Metabolizer", activityScore: 2.0 },
  "*1/*4": { phenotype: "Intermediate Metabolizer", activityScore: 1.0 },
  "*4/*4": { phenotype: "Poor Metabolizer", activityScore: 0.0 },
  // ... additional mappings
}
```

**Why It Exists:** Phenotype mappings are **CPIC reference data**. Separating them:
- Ensures CPIC compliance (judges can verify against published tables)
- Enables easy updates when CPIC revises guidelines
- Provides single source of truth for phenotype assignments
- Simplifies testing with known diplotype-phenotype pairs

---

#### `config/drugRiskMatrix.js`
**Purpose:** Drug-gene risk classifications based on CPIC guidelines  
**Content:** Risk levels for drug-gene-phenotype combinations

**Example Structure:**
```javascript
const CYP2D6_DRUG_RISKS = {
  "codeine": {
    "Poor Metabolizer": "low_risk",
    "Ultrarapid Metabolizer": "high_risk",
    "Normal Metabolizer": "no_known_interaction"
  },
  // ... additional drugs
}
```

**Why It Exists:** Risk classifications are **clinical evidence**. Separating them:
- Ensures CPIC alignment (judges can verify against guidelines)
- Enables easy addition of new drugs
- Provides single source of truth for risk assignments
- Simplifies testing with known phenotype-risk pairs

---

#### `config/cpicRecommendations.js`
**Purpose:** CPIC-aligned recommendation templates  
**Content:** Clinical recommendations for drug-gene-phenotype combinations

**Example Structure:**
```javascript
const RECOMMENDATIONS = {
  "CYP2D6_codeine_ultrarapid": {
    recommendation: "Avoid codeine use. Choose alternative analgesic not metabolized by CYP2D6 (e.g., morphine, hydromorphone).",
    strength: "Strong",
    cpicGuideline: "CPIC Guideline for CYP2D6 and Codeine (2014)",
    alternatives: ["morphine", "hydromorphone", "oxycodone"]
  },
  // ... additional recommendations
}
```

**Why It Exists:** Recommendations are **guideline-based templates**. Separating them:
- Ensures CPIC compliance (judges can verify against published guidelines)
- Enables easy updates when guidelines are revised
- Provides single source of truth for clinical recommendations
- Simplifies testing with known interaction-recommendation pairs
- **Critical for evaluation** - judges will assess recommendation quality

---

### Backend Test Data

#### `test-data/sample_wildtype.vcf`
**Purpose:** Sample VCF with no variants (all *1/*1 diplotypes)  
**Use Case:** Tests default diplotype assignment and "no_known_interaction" handling

---

#### `test-data/sample_cyp2d6_poor.vcf`
**Purpose:** Sample VCF with CYP2D6 *4/*4 (poor metabolizer)  
**Use Case:** Tests high-risk interaction detection (e.g., codeine, tamoxifen)

---

#### `test-data/sample_cyp2d6_ultra.vcf`
**Purpose:** Sample VCF with CYP2D6 gene duplication (ultrarapid metabolizer)  
**Use Case:** Tests ultrarapid metabolizer handling and high-risk codeine interaction

---

#### `test-data/sample_mixed.vcf`
**Purpose:** Sample VCF with variants across multiple genes  
**Use Case:** Tests complete pipeline with diverse diplotypes and risk levels

---

#### `test-data/sample_malformed.vcf`
**Purpose:** Sample VCF with parsing errors  
**Use Case:** Tests error handling and graceful degradation

**Why Test Data Exists:** Sample VCF files are **essential for demo preparation**. They enable:
- End-to-end testing before submission
- Demo rehearsal with known results
- Verification of color coding and risk visualization
- **Demo confidence** - knowing exactly what results will appear

---

## API Structure

### Request Flow

```
Client Request
    ↓
CORS Middleware (validate origin)
    ↓
Route Handler (routes/api.js)
    ↓
Controller (uploadController, analysisController, resultsController)
    ↓
Service (processingPipeline.js)
    ↓
Modules (vcfParser → geneFilter → diplotypeResolver → ... → llmExplanationService)
    ↓
Validator (jsonSchemaValidator.js)
    ↓
Controller (format response)
    ↓
Error Handler Middleware (if error occurs)
    ↓
Client Response
```

---

### API Endpoint Details

#### `POST /api/upload`

**Purpose:** Upload VCF file  
**Request:**
- Method: POST
- Content-Type: multipart/form-data
- Body: `vcfFile` (File)

**Response (Success - 200):**
```json
{
  "uploadId": "uuid-v4",
  "filename": "patient_sample.vcf",
  "size": 1048576
}
```

**Response (Error - 400):**
```json
{
  "error_code": "INVALID_FILE_FORMAT",
  "message": "Invalid file format. Please upload a .vcf or .vcf.gz file."
}
```

**Processing Steps:**
1. Multer middleware validates file extension and size
2. File stored in `uploads/temp/` with unique filename
3. Upload ID generated (UUID v4)
4. Upload confirmation returned

---

#### `POST /api/analyze`

**Purpose:** Start pharmacogenomic analysis  
**Request:**
- Method: POST
- Content-Type: application/json
- Body:
  ```json
  {
    "uploadId": "uuid-v4",
    "patientId": "optional-patient-identifier"
  }
  ```

**Response (Success - 200):**
```json
{
  "analysisId": "uuid-v4",
  "status": "processing",
  "estimatedTime": 25
}
```

**Response (Error - 400):**
```json
{
  "error_code": "INVALID_UPLOAD_ID",
  "message": "Upload ID not found. Please upload a file first."
}
```

**Processing Steps:**
1. Validate upload ID exists
2. Generate analysis ID
3. Start processing pipeline asynchronously
4. Return analysis ID immediately (non-blocking)

---

#### `GET /api/results/:analysisId`

**Purpose:** Retrieve analysis results  
**Request:**
- Method: GET
- URL Parameter: `analysisId` (UUID)

**Response (Success - 200):**
```json
{
  "patient_id": "P12345",
  "analysis_date": "2026-02-19T10:30:00Z",
  "genes": [
    {
      "name": "CYP2D6",
      "diplotype": "*1/*4",
      "phenotype": "Intermediate Metabolizer",
      "activity_score": 1.0
    }
  ],
  "drug_interactions": [
    {
      "gene_name": "CYP2D6",
      "drug_name": "codeine",
      "risk_level": "moderate_risk",
      "phenotype": "Intermediate Metabolizer",
      "cpic_level": "A"
    }
  ],
  "recommendations": [
    {
      "drug_name": "codeine",
      "gene_name": "CYP2D6",
      "recommendation": "Consider 50% dose reduction or alternative analgesic",
      "strength": "Moderate",
      "alternatives": ["tramadol", "hydrocodone"]
    }
  ],
  "explanations": [
    {
      "drug_name": "codeine",
      "gene_name": "CYP2D6",
      "explanation": "Your body processes codeine at a slower rate than average..."
    }
  ]
}
```

**Response (Error - 404):**
```json
{
  "error_code": "ANALYSIS_NOT_FOUND",
  "message": "Analysis not found. Please check the analysis ID."
}
```

**Processing Steps:**
1. Validate analysis ID format
2. Retrieve results from in-memory storage
3. Validate results against JSON schema (critical!)
4. Return validated JSON or 500 error if validation fails

---

### Response Formatting Pipeline

**Critical Rule:** All responses pass through JSON schema validator before being sent to client.

```
Processing Complete
    ↓
Build Result Object
    ↓
JSON Schema Validator (Zod)
    ↓
Validation Success? ──NO──> Return 500 Error (log validation errors)
    ↓ YES
Return 200 with Valid JSON
```

**Why This Matters:** This pipeline **guarantees** that only schema-compliant JSON reaches judges, preventing disqualification.

---

## Configuration Files

### Frontend Configuration

#### `frontend/.env`
**Purpose:** Frontend environment variables (gitignored)  
**Content:**
```
VITE_API_URL=http://localhost:3000
```

**Production:**
```
VITE_API_URL=https://pharmaguard-api.render.com
```

**Why It Exists:** Enables easy switching between development and production backend URLs without code changes.

---

#### `frontend/.env.example`
**Purpose:** Template for environment variables (committed to Git)  
**Content:**
```
VITE_API_URL=http://localhost:3000
```

**Why It Exists:** Provides setup guidance for developers and judges reviewing code.

---

#### `frontend/package.json`
**Purpose:** Frontend dependencies and scripts  
**Key Dependencies:**
- `react`: ^18.2.0
- `react-dom`: ^18.2.0
- `axios`: ^1.6.0
- `tailwindcss`: ^3.4.0
- `vite`: ^5.0.0

**Scripts:**
```json
{
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview"
}
```

**Why It Exists:** Defines all frontend dependencies and build commands for Vercel deployment.

---

#### `frontend/vite.config.js`
**Purpose:** Vite build configuration  
**Content:**
```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173
  }
})
```

**Why It Exists:** Configures Vite development server and build process. Ensures React plugin is enabled.

---

#### `frontend/tailwind.config.js`
**Purpose:** TailwindCSS configuration  
**Content:**
```javascript
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,jsx}"
  ],
  theme: {
    extend: {
      colors: {
        'risk-high': '#ef4444',      // Red
        'risk-moderate': '#f59e0b',  // Yellow
        'risk-low': '#10b981',       // Green
        'risk-none': '#6b7280'       // Gray
      }
    }
  },
  plugins: []
}
```

**Why It Exists:** Configures TailwindCSS to scan all JSX files and defines custom color palette for risk levels. Ensures consistent color coding across all components.

---

### Backend Configuration

#### `backend/.env`
**Purpose:** Backend environment variables (gitignored)  
**Content:**
```
PORT=3000
OPENAI_API_KEY=sk-...
CORS_ORIGIN=http://localhost:5173
MAX_FILE_SIZE=52428800
UPLOAD_DIR=uploads/temp
NODE_ENV=development
```

**Production:**
```
PORT=3000
OPENAI_API_KEY=sk-...
CORS_ORIGIN=https://pharmaguard.vercel.app
MAX_FILE_SIZE=52428800
UPLOAD_DIR=/tmp/uploads
NODE_ENV=production
```

**Why It Exists:** Centralizes all configuration, enabling easy deployment to Render without code changes. Protects API keys from being committed to Git.

---

#### `backend/.env.example`
**Purpose:** Template for environment variables (committed to Git)  
**Content:**
```
PORT=3000
OPENAI_API_KEY=your_openai_api_key_here
CORS_ORIGIN=http://localhost:5173
MAX_FILE_SIZE=52428800
UPLOAD_DIR=uploads/temp
NODE_ENV=development
```

**Why It Exists:** Provides setup guidance for developers and judges. Documents all required environment variables.

---

#### `backend/package.json`
**Purpose:** Backend dependencies and scripts  
**Key Dependencies:**
- `express`: ^4.18.0
- `@gmod/vcf`: ^5.0.0 (VCF parsing)
- `zod`: ^3.22.0 (JSON schema validation)
- `openai`: ^4.20.0 (LLM explanations)
- `multer`: ^1.4.5 (file upload)
- `cors`: ^2.8.5 (CORS middleware)
- `dotenv`: ^16.3.0 (environment variables)
- `uuid`: ^9.0.0 (unique ID generation)

**Scripts:**
```json
{
  "start": "node server.js",
  "dev": "nodemon server.js",
  "test": "jest"
}
```

**Why It Exists:** Defines all backend dependencies and run commands. Critical for Render deployment (uses `npm start`).

---

## Separation of Concerns

### Architectural Principles

This file structure follows strict separation of concerns to ensure:

1. **Modularity**
   - Each module has a single, well-defined responsibility
   - Modules can be developed, tested, and debugged independently
   - Changes to one module don't affect others

2. **Testability**
   - Each module can be unit tested with mock inputs
   - Property-based tests can target specific modules
   - Integration tests can verify module interactions

3. **Maintainability**
   - Code is organized logically by function
   - Easy to locate and fix bugs
   - Easy to add new features (e.g., new drugs, new genes)

4. **Demo Stability**
   - Error handling at every layer prevents cascading failures
   - Fallback mechanisms ensure system never crashes
   - Graceful degradation maintains functionality even when components fail

5. **JSON Schema Compliance**
   - Single validator module ensures consistent validation
   - Validation happens before every response
   - Validation failures return errors rather than invalid JSON
   - **Prevents disqualification**

---

### Data Flow Separation

**Clear Boundaries Between Layers:**

```
Presentation Layer (Frontend)
    ↓ HTTP Requests
API Layer (Routes + Controllers)
    ↓ Function Calls
Service Layer (Processing Pipeline)
    ↓ Function Calls
Business Logic Layer (Core Modules)
    ↓ Data Access
Configuration Layer (Reference Data)
```

**Why This Matters:**
- Frontend never accesses backend modules directly (API contract)
- Controllers never contain business logic (orchestration only)
- Modules never access HTTP requests (pure functions)
- Configuration is separated from logic (easy updates)

---

### Error Handling Separation

**Error Boundaries at Every Layer:**

1. **Frontend:** User-friendly error messages, no technical details
2. **API Layer:** HTTP status codes, structured error responses
3. **Service Layer:** Error logging, graceful degradation
4. **Module Layer:** Specific error types, detailed error messages
5. **Middleware Layer:** Centralized error formatting

**Why This Matters:**
- Errors are caught and handled at appropriate levels
- Users see helpful messages, not stack traces
- Developers see detailed logs for debugging
- **Demo never crashes** - all errors handled gracefully

---

### Validation Separation

**Multiple Validation Layers:**

1. **Client-Side (Frontend):** File extension, file size (immediate feedback)
2. **Upload Middleware:** File validation before storage
3. **Controller Layer:** Request parameter validation
4. **Module Layer:** Data validation (variants, diplotypes, etc.)
5. **Output Layer:** JSON schema validation (final gatekeeper)

**Why This Matters:**
- Invalid data caught early (reduces processing waste)
- Multiple safety nets prevent invalid data from reaching judges
- **Guarantees schema compliance** - final validation before response

---

### Configuration Separation

**Why Configuration Files Are Separated:**

1. **Star Alleles (`config/starAlleles.js`)**
   - Reference data from PharmVar database
   - Changes when new alleles are discovered
   - Judges can verify against published data

2. **Phenotype Maps (`config/phenotypeMaps.js`)**
   - Reference data from CPIC guidelines
   - Changes when CPIC updates guidelines
   - Judges can verify against CPIC tables

3. **Drug Risk Matrix (`config/drugRiskMatrix.js`)**
   - Clinical evidence from CPIC
   - Changes when new evidence emerges
   - Judges can verify against CPIC guidelines

4. **Recommendations (`config/cpicRecommendations.js`)**
   - Template-based clinical guidance
   - Changes when guidelines are revised
   - Judges can verify against published guidelines

**Benefits:**
- Logic separated from data (clean code)
- Easy to update when guidelines change
- Single source of truth for reference data
- Judges can verify CPIC compliance by reviewing config files

---

### Security Separation

**Security Measures at Multiple Layers:**

1. **Upload Layer:** File extension validation, size limits
2. **Storage Layer:** Temporary directory with restricted permissions
3. **Processing Layer:** Path sanitization, input validation
4. **Logging Layer:** PII filtering
5. **Cleanup Layer:** Automatic file deletion

**Why This Matters:**
- Defense in depth (multiple security layers)
- Prevents common attacks (directory traversal, DoS, etc.)
- Protects patient privacy (no PII stored or logged)
- **Meets evaluation criteria** for security and privacy

---

### Alignment with Evaluation Criteria

**How This Architecture Supports Hackathon Success:**

1. **JSON Schema Compliance (Critical)**
   - Single validator module (`jsonSchemaValidator.js`)
   - Validation before every response
   - Validation failures return errors, not invalid JSON
   - **Prevents disqualification**

2. **CPIC Alignment (High Priority)**
   - Configuration files match CPIC guidelines
   - Judges can verify by reviewing config files
   - Easy to update when guidelines change
   - **Demonstrates clinical validity**

3. **Explainable AI (High Priority)**
   - Dedicated LLM service module
   - Fallback explanations ensure reliability
   - Plain-language output for non-technical users
   - **Meets innovation criteria**

4. **Demo Stability (Critical)**
   - Error handling at every layer
   - Graceful degradation (LLM fallbacks, partial results)
   - Comprehensive logging for debugging
   - **Ensures smooth presentation**

5. **Code Quality (Medium Priority)**
   - Clear separation of concerns
   - Well-documented modules
   - Consistent naming conventions
   - **Demonstrates professional development**

6. **Security (Medium Priority)**
   - Multiple validation layers
   - Path sanitization
   - PII filtering
   - **Meets privacy requirements**

---

## Summary

This file structure is designed for **hackathon success** by prioritizing:

1. **Schema Compliance:** Guaranteed through dedicated validator module
2. **Modularity:** Each component has a single, clear responsibility
3. **Testability:** Independent modules enable comprehensive testing
4. **Demo Readiness:** Error handling and fallbacks ensure stability
5. **CPIC Alignment:** Configuration files match published guidelines
6. **Professional Quality:** Clean architecture demonstrates technical competence

**Critical Success Factors:**
- JSON schema validation before every response (prevents disqualification)
- Comprehensive error handling (prevents demo crashes)
- CPIC-aligned configuration (demonstrates clinical validity)
- LLM fallbacks (ensures explanation generation never fails)
- Clear separation of concerns (enables rapid debugging)

This architecture ensures the PharmaGuard system is **judge-ready, demo-stable, and evaluation-compliant**.

---

**End of File Structure Documentation**

