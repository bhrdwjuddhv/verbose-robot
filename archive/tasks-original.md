# Implementation Plan: PharmaGuard - Pharmacogenomic Risk Prediction System

## Overview

This implementation plan breaks down the PharmaGuard system into discrete coding tasks following the pipeline architecture: file upload → VCF parsing → diplotype resolution → phenotype mapping → risk classification → recommendation generation → explanation generation → JSON validation → frontend display. Each task builds incrementally, with property-based tests integrated throughout to catch errors early.

The implementation uses JavaScript (no TypeScript) with React/Vite for frontend and Node.js/Express for backend, optimized for hackathon rapid development while maintaining production-ready code quality.

## Tasks

- [ ] 1. Initialize project structure and dependencies
  - Create frontend project with Vite + React + TailwindCSS
  - Create backend project with Express.js
  - Install dependencies: @gmod/vcf, zod, multer, cors, axios, fast-check
  - Set up basic folder structure for both projects
  - Configure environment variables for both frontend and backend
  - _Requirements: 13.1, 13.2, 13.3_

- [ ] 2. Implement file upload handler with validation
  - [ ] 2.1 Create FileUploadHandler with multer configuration
  - [ ] 2.2 Implement file extension validation (.vcf, .vcf.gz)
  - [ ] 2.3 Implement file size limit (50MB)
  - [ ] 2.4 Create POST /api/upload endpoint
  - [ ] 2.5 Write property test for file extension validation (Property 1)
  - [ ] 2.6 Write property test for unique upload identifiers (Property 2)
  - [ ] 2.7 Write property test for invalid file format errors (Property 3)
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 11.1, 12.1_

- [ ] 3. Implement VCF parser
  - [ ] 3.1 Create VCFParser class using @gmod/vcf
  - [ ] 3.2 Implement parseVariants() method
  - [ ] 3.3 Implement filterTargetGenes() for six target genes
  - [ ] 3.4 Handle malformed VCF records gracefully
  - [ ] 3.5 Write property test for variant extraction (Property 4)
  - [ ] 3.6 Write property test for target gene filtering (Property 5)
  - [ ] 3.7 Write property test for malformed record handling (Property 6)
  - [ ] 3.8 Write property test for variant structure (Property 7)
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [ ] 4. Implement diplotype resolver
  - [ ] 4.1 Create DiplotypeResolver class
  - [ ] 4.2 Define star allele definitions for six target genes
  - [ ] 4.3 Implement resolveDiplotypes() method
  - [ ] 4.4 Implement matchStarAllele() for variant matching
  - [ ] 4.5 Write property test for all target genes resolution (Property 8)
  - [ ] 4.6 Write property test for wild-type defaults (Property 9)
  - [ ] 4.7 Write property test for star allele notation (Property 10)
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5_

- [ ] 5. Implement phenotype mapper
  - [ ] 5.1 Create PhenotypeMapper class
  - [ ] 5.2 Define diplotype-to-phenotype mapping tables for six genes
  - [ ] 5.3 Implement mapPhenotypes() method
  - [ ] 5.4 Implement getPhenotypeForDiplotype() method
  - [ ] 5.5 Write property test for diplotype-to-phenotype mapping (Property 11)
  - [ ] 5.6 Write property test for unknown diplotype handling (Property 12)
  - [ ] 5.7 Write property test for all target genes phenotypes (Property 13)
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

- [ ] 6. Implement risk classifier
  - [ ] 6.1 Create RiskClassifier class
  - [ ] 6.2 Define drug-gene risk matrices for target genes
  - [ ] 6.3 Implement classifyRisks() method
  - [ ] 6.4 Implement getRiskForDrugGene() method
  - [ ] 6.5 Write property test for unique risk levels per drug-gene pair (Property 14)
  - [ ] 6.6 Write property test for unknown interaction handling (Property 15)
  - [ ] 6.7 Write property test for risk level presence (Property 16)
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

- [ ] 7. Implement recommendation engine
  - [ ] 7.1 Create RecommendationEngine class
  - [ ] 7.2 Define CPIC-aligned recommendation templates
  - [ ] 7.3 Implement generateRecommendations() method
  - [ ] 7.4 Implement getRecommendationForInteraction() method
  - [ ] 7.5 Write property test for high-risk actionable recommendations (Property 17)
  - [ ] 7.6 Write property test for moderate-risk dosing guidance (Property 18)
  - [ ] 7.7 Write property test for all risk levels recommendations (Property 19)
  - [ ] 7.8 Write property test for recommendation strength levels (Property 20)
  - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 8. Implement explanation generator with LLM
  - [ ] 8.1 Create ExplanationGenerator class
  - [ ] 8.2 Implement LLM integration (OpenAI or Gemini)
  - [ ] 8.3 Implement generateExplanations() method
  - [ ] 8.4 Implement getFallbackExplanation() method
  - [ ] 8.5 Write property test for explanation generation (Property 21)
  - [ ] 8.6 Write property test for LLM failure fallback (Property 22)
  - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

- [ ] 9. Implement JSON validator
  - [ ] 9.1 Create JSONValidator class with Zod schema
  - [ ] 9.2 Define complete output schema matching hackathon requirements
  - [ ] 9.3 Implement validate() method
  - [ ] 9.4 Write property test for output validation (Property 23)
  - [ ] 9.5 Write property test for required top-level fields (Property 24)
  - [ ] 9.6 Write property test for drug interaction fields (Property 25)
  - [ ] 9.7 Write property test for validation failure handling (Property 26)
  - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_

- [ ] 10. Implement backend API endpoints
  - [ ] 10.1 Create POST /api/analyze endpoint
  - [ ] 10.2 Create GET /api/results/:id endpoint
  - [ ] 10.3 Implement processing pipeline orchestration
  - [ ] 10.4 Implement error handling middleware
  - [ ] 10.5 Write property test for invalid request parameters (Property 34)
  - [ ] 10.6 Write property test for appropriate status codes (Property 35)
  - _Requirements: 11.2, 11.3, 11.4, 11.5_

- [ ] 11. Implement error handling and logging
  - [ ] 11.1 Create centralized error logging utility
  - [ ] 11.2 Implement structured error responses
  - [ ] 11.3 Implement file cleanup on upload failure
  - [ ] 11.4 Write property test for error logging (Property 30)
  - [ ] 11.5 Write property test for structured error responses (Property 31)
  - [ ] 11.6 Write property test for LLM failure resilience (Property 32)
  - [ ] 11.7 Write property test for upload cleanup (Property 33)
  - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

- [ ] 12. Implement security measures
  - [ ] 12.1 Implement file path sanitization
  - [ ] 12.2 Implement PII filtering in logs
  - [ ] 12.3 Configure CORS with origin whitelist
  - [ ] 12.4 Implement file cleanup after processing
  - [ ] 12.5 Write property test for path sanitization (Property 36)
  - [ ] 12.6 Write property test for PII filtering (Property 37)
  - _Requirements: 12.1, 12.2, 12.3, 12.4, 12.5_

- [ ] 13. Implement frontend file upload component
  - [ ] 13.1 Create FileUploadComponent with file selection
  - [ ] 13.2 Implement file validation on client side
  - [ ] 13.3 Implement upload progress indicator
  - [ ] 13.4 Implement error message display
  - [ ] 13.5 Style with TailwindCSS
  - _Requirements: 9.1, 9.3_

- [ ] 14. Implement frontend results display component
  - [ ] 14.1 Create ResultsDisplayComponent
  - [ ] 14.2 Implement color-coded risk level display
  - [ ] 14.3 Implement gene grouping and expandable sections
  - [ ] 14.4 Implement interaction detail modal
  - [ ] 14.5 Write property test for risk level colors (Property 27)
  - [ ] 14.6 Write property test for gene grouping (Property 28)
  - [ ] 14.7 Write property test for interaction details (Property 29)
  - _Requirements: 9.2, 9.4, 9.5_

- [ ] 15. Implement frontend-backend integration
  - [ ] 15.1 Configure Axios with API base URL
  - [ ] 15.2 Implement API service methods
  - [ ] 15.3 Implement loading states and error handling
  - [ ] 15.4 Test end-to-end flow with sample VCF files
  - _Requirements: 14.1, 14.2, 14.4_

- [ ] 16. Create sample VCF files and test data
  - [ ] 16.1 Create sample_wildtype.vcf
  - [ ] 16.2 Create sample_cyp2d6_poor.vcf
  - [ ] 16.3 Create sample_cyp2d6_ultra.vcf
  - [ ] 16.4 Create sample_mixed.vcf
  - [ ] 16.5 Create sample_malformed.vcf
  - _Requirements: 15.2_

- [ ] 17. Deploy and configure production environment
  - [ ] 17.1 Configure Vercel deployment for frontend
  - [ ] 17.2 Configure Render deployment for backend
  - [ ] 17.3 Set up environment variables in production
  - [ ] 17.4 Test deployed application end-to-end
  - [ ] 17.5 Verify CORS configuration
  - _Requirements: 13.1, 13.2, 13.3, 13.4, 13.5_

- [ ] 18. Create documentation and demo materials
  - [ ] 18.1 Write comprehensive README with setup instructions
  - [ ] 18.2 Document API endpoints
  - [ ] 18.3 Create deployment guide
  - [ ] 18.4 Add inline code comments
  - [ ] 18.5 Prepare demo workflow script
  - _Requirements: 15.1, 15.2, 15.3, 15.4, 15.5_