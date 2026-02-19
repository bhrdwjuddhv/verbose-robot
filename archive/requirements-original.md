# Requirements Document

## Introduction

PharmaGuard is a pharmacogenomic risk prediction system designed for the RIFT 2026 Hackathon. The system analyzes genomic variants from VCF files to predict drug-gene interaction risks, resolve diplotypes and phenotypes for six critical pharmacogenomic genes, and provide CPIC-aligned recommendations with explainable AI-powered insights for healthcare providers and patients.

## Glossary

- **VCF_Parser**: Component that reads and parses Variant Call Format genomic files
- **Diplotype_Resolver**: Component that determines diplotype combinations from variant data
- **Phenotype_Mapper**: Component that maps diplotypes to metabolizer phenotypes
- **Risk_Classifier**: Component that categorizes drug-gene interaction risks
- **Recommendation_Engine**: Component that generates CPIC-aligned clinical recommendations
- **Explanation_Generator**: Component that produces LLM-powered explanations for non-technical users
- **JSON_Validator**: Component that ensures output conforms to hackathon schema requirements
- **File_Upload_Handler**: Component that manages secure VCF file uploads
- **Frontend_UI**: React-based user interface for file upload and results display
- **Backend_API**: Express.js REST API that orchestrates processing pipeline
- **Target_Genes**: The six pharmacogenomic genes (CYP2D6, CYP2C19, CYP2C9, VKORC1, SLCO1B1, DPYD)
- **CPIC**: Clinical Pharmacogenetics Implementation Consortium guidelines
- **Risk_Level**: Classification of drug-gene interaction severity (high_risk, moderate_risk, low_risk, no_known_interaction)
- **Metabolizer_Phenotype**: Functional status of enzyme activity (Poor, Intermediate, Normal, Rapid, Ultrarapid Metabolizer)

## Requirements

### Requirement 1: VCF File Upload and Validation

**User Story:** As a healthcare provider, I want to securely upload VCF genomic files, so that I can analyze patient pharmacogenomic data for drug interaction risks.

#### Acceptance Criteria

1. WHEN a user uploads a file, THE File_Upload_Handler SHALL validate that the file has a .vcf or .vcf.gz extension
2. WHEN a file exceeds 50MB, THE File_Upload_Handler SHALL reject the upload and return a descriptive error message
3. WHEN a file is uploaded, THE File_Upload_Handler SHALL scan for malicious content before processing
4. WHEN a valid VCF file is received, THE Backend_API SHALL return an upload confirmation with a unique processing identifier
5. IF an invalid file format is uploaded, THEN THE File_Upload_Handler SHALL return an error message specifying the expected format

### Requirement 2: VCF Parsing and Variant Extraction

**User Story:** As a system, I want to parse VCF files and extract relevant genomic variants, so that I can identify pharmacogenomic markers for analysis.

#### Acceptance Criteria

1. WHEN a VCF file is processed, THE VCF_Parser SHALL extract all variant records from the file
2. WHEN parsing variants, THE VCF_Parser SHALL filter for variants located in the Target_Genes only
3. WHEN a VCF file contains malformed records, THE VCF_Parser SHALL log the error and continue processing valid records
4. WHEN variant extraction completes, THE VCF_Parser SHALL return a structured list of variants with chromosome, position, reference allele, and alternate allele
5. IF a VCF file contains no variants in Target_Genes, THEN THE VCF_Parser SHALL return an empty variant list with a descriptive message

### Requirement 3: Diplotype Resolution

**User Story:** As a clinician, I want the system to resolve diplotypes from genomic variants, so that I can understand the patient's genetic makeup for drug metabolism genes.

#### Acceptance Criteria

1. WHEN variants for a Target_Gene are identified, THE Diplotype_Resolver SHALL determine the diplotype combination using star allele nomenclature
2. WHEN resolving diplotypes, THE Diplotype_Resolver SHALL process all six Target_Genes (CYP2D6, CYP2C19, CYP2C9, VKORC1, SLCO1B1, DPYD)
3. WHEN insufficient variant data exists for a gene, THE Diplotype_Resolver SHALL assign a default diplotype of *1/*1 (wild-type)
4. WHEN diplotype resolution completes, THE Diplotype_Resolver SHALL return diplotypes in standard notation (e.g., *1/*2, *2/*3)
5. IF conflicting variant data prevents diplotype resolution, THEN THE Diplotype_Resolver SHALL flag the gene as "unresolved" with an explanation

### Requirement 4: Phenotype Mapping

**User Story:** As a healthcare provider, I want diplotypes mapped to metabolizer phenotypes, so that I can understand the functional impact on drug metabolism.

#### Acceptance Criteria

1. WHEN a diplotype is resolved, THE Phenotype_Mapper SHALL map it to the corresponding Metabolizer_Phenotype
2. WHEN mapping phenotypes, THE Phenotype_Mapper SHALL use CPIC-aligned phenotype classifications
3. WHEN a diplotype has no established phenotype mapping, THE Phenotype_Mapper SHALL assign "Unknown Metabolizer" status
4. THE Phenotype_Mapper SHALL support all standard phenotype categories (Poor, Intermediate, Normal, Rapid, Ultrarapid Metabolizer)
5. WHEN phenotype mapping completes, THE Phenotype_Mapper SHALL return phenotype labels for all six Target_Genes

### Requirement 5: Drug-Gene Interaction Risk Classification

**User Story:** As a clinician, I want drug-gene interactions classified by risk level, so that I can prioritize clinical interventions for high-risk combinations.

#### Acceptance Criteria

1. WHEN a phenotype is determined, THE Risk_Classifier SHALL evaluate drug-gene interaction risks for relevant medications
2. WHEN classifying risks, THE Risk_Classifier SHALL assign exactly one Risk_Level per drug-gene pair
3. THE Risk_Classifier SHALL use CPIC guidelines to determine Risk_Level assignments
4. WHEN a drug has no known interaction with a gene, THE Risk_Classifier SHALL assign "no_known_interaction" status
5. WHEN risk classification completes, THE Risk_Classifier SHALL return a list of drug-gene pairs with assigned Risk_Level values

### Requirement 6: CPIC-Aligned Recommendation Generation

**User Story:** As a healthcare provider, I want CPIC-aligned clinical recommendations, so that I can make evidence-based prescribing decisions.

#### Acceptance Criteria

1. WHEN a high_risk interaction is identified, THE Recommendation_Engine SHALL generate an actionable clinical recommendation
2. WHEN generating recommendations, THE Recommendation_Engine SHALL align with current CPIC guideline versions
3. WHEN a moderate_risk interaction exists, THE Recommendation_Engine SHALL provide dosing adjustment guidance
4. WHEN a low_risk or no_known_interaction exists, THE Recommendation_Engine SHALL provide standard prescribing information
5. THE Recommendation_Engine SHALL include recommendation strength levels (Strong, Moderate, Optional) based on CPIC evidence ratings

### Requirement 7: Explainable AI-Powered Explanations

**User Story:** As a patient or non-technical user, I want plain-language explanations of my pharmacogenomic results, so that I can understand the implications without medical expertise.

#### Acceptance Criteria

1. WHEN results are generated, THE Explanation_Generator SHALL produce a plain-language summary for each drug-gene interaction
2. WHEN generating explanations, THE Explanation_Generator SHALL use an LLM (OpenAI or Gemini) to create non-technical descriptions
3. WHEN explaining high_risk interactions, THE Explanation_Generator SHALL clearly communicate the potential consequences and recommended actions
4. THE Explanation_Generator SHALL avoid medical jargon and use analogies or simple terms
5. WHEN LLM API calls fail, THE Explanation_Generator SHALL fall back to template-based explanations

### Requirement 8: JSON Schema Compliance and Output Validation

**User Story:** As a hackathon judge, I want results in the exact specified JSON schema, so that I can evaluate submissions consistently and fairly.

#### Acceptance Criteria

1. WHEN results are finalized, THE JSON_Validator SHALL validate the output against the hackathon-specified schema
2. THE Backend_API SHALL output JSON containing all required fields: patient_id, genes, drug_interactions, recommendations, and explanations
3. WHEN outputting drug interactions, THE Backend_API SHALL include gene_name, drug_name, risk_level, and phenotype for each interaction
4. THE JSON_Validator SHALL use Zod schema validation to ensure type correctness and required field presence
5. IF validation fails, THEN THE Backend_API SHALL return a 500 error with validation details rather than invalid JSON

### Requirement 9: Frontend User Interface and Visualization

**User Story:** As a user, I want an intuitive interface with color-coded risk visualization, so that I can quickly identify critical drug-gene interactions.

#### Acceptance Criteria

1. WHEN the application loads, THE Frontend_UI SHALL display a file upload interface with clear instructions
2. THE Frontend_UI SHALL provide a drug input field for users to specify medications of interest
3. WHEN entering drugs, THE Frontend_UI SHALL support multiple drug inputs via comma-separated text or multi-select dropdown
4. WHEN a user enters drug names, THE Frontend_UI SHALL validate drug names against the supported drug list
5. WHEN displaying results, THE Frontend_UI SHALL use color coding: Red for high_risk, Yellow for moderate_risk, Green for low_risk, Gray for no_known_interaction
6. WHEN results are loading, THE Frontend_UI SHALL display a progress indicator with processing status
7. THE Frontend_UI SHALL organize results by gene with expandable sections for detailed information
8. WHEN a user clicks on a drug-gene interaction, THE Frontend_UI SHALL display the full explanation and recommendation
9. IF no drugs are specified, THE Frontend_UI SHALL analyze all supported drugs by default

### Requirement 10: Error Handling and Robustness

**User Story:** As a system administrator, I want comprehensive error handling, so that the system remains stable and provides helpful feedback when issues occur.

#### Acceptance Criteria

1. WHEN any component encounters an error, THE Backend_API SHALL log the error with timestamp, component name, and error details
2. WHEN a processing error occurs, THE Backend_API SHALL return a structured error response with an error code and user-friendly message
3. IF the VCF_Parser fails, THEN THE Backend_API SHALL return a 400 error indicating the specific parsing issue
4. IF the LLM API is unavailable, THEN THE Explanation_Generator SHALL use fallback explanations without failing the entire request
5. WHEN file upload fails, THE File_Upload_Handler SHALL clean up any partially uploaded files

### Requirement 11: API Endpoint Structure

**User Story:** As a frontend developer, I want well-defined API endpoints, so that I can integrate the backend services reliably.

#### Acceptance Criteria

1. THE Backend_API SHALL expose a POST /api/upload endpoint for VCF file uploads
2. THE Backend_API SHALL expose a POST /api/analyze endpoint for processing uploaded VCF files
3. THE Backend_API SHALL expose a GET /api/results/:id endpoint for retrieving analysis results
4. WHEN an endpoint receives a request, THE Backend_API SHALL validate request parameters and return 400 for invalid inputs
5. THE Backend_API SHALL return appropriate HTTP status codes (200, 400, 500) for all responses

### Requirement 12: Security and Data Protection

**User Story:** As a healthcare provider, I want patient genomic data protected, so that I can comply with privacy regulations and maintain patient trust.

#### Acceptance Criteria

1. WHEN files are uploaded, THE File_Upload_Handler SHALL store them in a secure temporary directory with restricted permissions
2. WHEN processing completes, THE Backend_API SHALL delete uploaded VCF files within 1 hour
3. THE Backend_API SHALL implement file size limits to prevent denial-of-service attacks
4. THE Backend_API SHALL sanitize all file paths to prevent directory traversal attacks
5. THE Backend_API SHALL not log or store patient identifiable information from VCF files

### Requirement 13: Deployment and Environment Configuration

**User Story:** As a developer, I want the system deployable to Vercel and Render, so that I can demonstrate the application for hackathon judges.

#### Acceptance Criteria

1. THE Frontend_UI SHALL be deployable to Vercel with a single command
2. THE Backend_API SHALL be deployable to Render with environment variable configuration
3. WHEN deployed, THE Frontend_UI SHALL connect to the Backend_API using environment-configured URLs
4. THE Backend_API SHALL support CORS configuration for the deployed frontend domain
5. WHERE environment variables are required, THE system SHALL provide clear documentation for configuration

### Requirement 14: Performance and Responsiveness

**User Story:** As a demo presenter, I want fast processing times, so that I can showcase the system effectively during the hackathon presentation.

#### Acceptance Criteria

1. WHEN a VCF file under 10MB is uploaded, THE system SHALL complete analysis within 30 seconds
2. WHEN the Frontend_UI makes API requests, THE Backend_API SHALL respond within 5 seconds for non-LLM operations
3. WHEN generating LLM explanations, THE Explanation_Generator SHALL implement a 10-second timeout per explanation
4. THE Frontend_UI SHALL remain responsive during file upload and processing
5. WHEN multiple requests are received, THE Backend_API SHALL handle them concurrently without blocking

### Requirement 15: Demo Readiness and Documentation

**User Story:** As a hackathon participant, I want clear documentation and a demo-ready system, so that I can present confidently to judges.

#### Acceptance Criteria

1. THE system SHALL include a README with setup instructions, API documentation, and demo workflow
2. THE system SHALL include sample VCF files for demonstration purposes
3. WHEN demonstrating, THE Frontend_UI SHALL provide a smooth user experience without requiring technical knowledge
4. THE system SHALL include inline code comments explaining complex pharmacogenomic logic
5. THE system SHALL include a deployment guide with step-by-step instructions for Vercel and Render
