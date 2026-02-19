# Requirements Document - PharmaGuard (6-Hour Hackathon)

## Introduction

PharmaGuard is a pharmacogenomic risk prediction system built for a 6-hour hackathon. The system analyzes VCF files to predict drug-gene interaction risks for 6 supported drugs across 6 critical genes, providing CPIC-aligned recommendations with GPT-4 powered explanations.

## Glossary

- **Target_Genes**: CYP2D6, CYP2C19, CYP2C9, SLCO1B1, TPMT, DPYD
- **Supported_Drugs**: CODEINE, WARFARIN, CLOPIDOGREL, SIMVASTATIN, AZATHIOPRINE, FLUOROURACIL
- **Risk_Labels**: Safe (Green), Adjust (Orange), Toxic/Ineffective (Red), Unknown (Purple)

## Functional Requirements

### FR1: VCF File Upload (Max 5MB)
**User Story:** As a user, I want to upload VCF files up to 5MB

**Acceptance Criteria:**
1. System SHALL accept .vcf and .vcf.gz files
2. System SHALL reject files exceeding 5MB
3. System SHALL provide drag-and-drop upload interface
4. System SHALL validate file format before processing

### FR2: Drug Selection
**User Story:** As a user, I want to select which drugs to analyze

**Acceptance Criteria:**
1. System SHALL display 6 supported drugs as selection chips
2. System SHALL allow multiple drug selection
3. System SHALL default to analyzing all 6 drugs if none selected
4. Supported drugs: CODEINE, WARFARIN, CLOPIDOGREL, SIMVASTATIN, AZATHIOPRINE, FLUOROURACIL

### FR3: Variant Extraction
**User Story:** As a system, I want to extract variants from 6 target genes

**Acceptance Criteria:**
1. System SHALL parse VCF using Node.js readline (streaming)
2. System SHALL extract variants for: CYP2D6, CYP2C19, CYP2C9, SLCO1B1, TPMT, DPYD
3. System SHALL handle malformed records gracefully
4. System SHALL return structured variant data

### FR4: Diplotype Resolution
**User Story:** As a system, I want to resolve star allele diplotypes

**Acceptance Criteria:**
1. System SHALL match variants to star allele patterns
2. System SHALL assign *1/*1 (wild-type) as default
3. System SHALL return diplotype notation (e.g., *1/*4)
4. System SHALL calculate confidence score

### FR5: Phenotype Mapping
**User Story:** As a system, I want to map diplotypes to phenotypes

**Acceptance Criteria:**
1. System SHALL use CPIC phenotype mappings
2. System SHALL support: Poor, Intermediate, Normal, Rapid, Ultrarapid Metabolizer
3. System SHALL assign "Unknown" for unmapped diplotypes

### FR6: Risk Classification
**User Story:** As a clinician, I want to see color-coded risk levels

**Acceptance Criteria:**
1. System SHALL classify risks as: Safe (Green), Adjust (Orange), Toxic/Ineffective (Red), Unknown (Purple)
2. System SHALL follow CPIC dosing guidelines
3. System SHALL display risk cards with color coding
4. System SHALL show diplotype + phenotype + confidence

### FR7: Clinical Recommendations
**User Story:** As a clinician, I want CPIC-aligned recommendations

**Acceptance Criteria:**
1. System SHALL provide dosing guidance for each drug-gene pair
2. System SHALL reference CPIC guidelines
3. System SHALL suggest alternatives for high-risk interactions

### FR8: AI Explanations
**User Story:** As a user, I want plain-language explanations

**Acceptance Criteria:**
1. System SHALL use OpenAI GPT-4 for explanations
2. System SHALL generate single structured prompt per analysis
3. System SHALL provide fallback explanations if API fails
4. Explanations SHALL be <150 words

### FR9: 3D Visualization
**User Story:** As a user, I want to visualize variants in 3D

**Acceptance Criteria:**
1. System SHALL use React Three Fiber
2. System SHALL display variants as simple spheres
3. System SHALL support rotate + zoom controls
4. System SHALL color-code by risk level

### FR10: Clinical Chat
**User Story:** As a user, I want to ask clarifying questions

**Acceptance Criteria:**
1. System SHALL provide simple question input
2. System SHALL use GPT-4 for clarifications
3. System SHALL maintain context of current analysis

## Non-Functional Requirements

### NFR1: Performance
- VCF parsing SHALL complete within 10 seconds for 5MB files
- Total analysis SHALL complete within 30 seconds

### NFR2: Output Format
System SHALL output JSON with structure:
```json
{
  "patient_id": "string",
  "drug": "string",
  "risk_label": "Safe|Adjust|Toxic|Unknown",
  "confidence": "number (0-1)",
  "diplotype": "string",
  "phenotype": "string",
  "variants": ["array"],
  "clinical_recommendation": "string",
  "llm_explanation": "string"
}
```

### NFR3: UI Theme
- Dark medical theme
- Cyan primary color (#00BCD4)
- Material-UI components
- Risk-based color palette

### NFR4: Deployment
- Frontend: Vercel
- Backend: Render
- Database: MongoDB Atlas
- AI: OpenAI GPT-4

## Out of Scope (6-Hour Constraint)
- Multi-agent systems
- WebSockets/real-time updates
- Complex authentication
- Result persistence beyond session
- Advanced 3D visualizations
- Multiple LLM providers
