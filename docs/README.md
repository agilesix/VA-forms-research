# VA Forms Research Documentation

This directory contains research and analysis of VA forms implementation on VA.gov.

## Structure

### `/forms/`
Individual form analysis documents, including implementation details, code references, and integration patterns.

### `/diagrams/`
Mermaid diagrams and other visual representations of form processing flows.

## Available Research

### Form 21-2680
- **[Implementation Overview](forms/21-2680-implementation-overview.md)** - Comprehensive analysis of how Form 21-2680 (Examination for Housebound Status or Permanent Need for Regular Aid and Attendance) is implemented
- **[Processing Flow Diagram](diagrams/21-2680-flow.mermaid)** - Sequence diagram showing the end-to-end upload and processing flow

## Contributing

When adding new form research:

1. Create a new document in `/forms/` following the naming pattern: `{form-number}-implementation-overview.md`
2. Add any related diagrams to `/diagrams/` with the pattern: `{form-number}-{diagram-type}.mermaid`
3. Update this README with links to the new research
4. Include key file references from the vets-website and vets-api repositories
5. Document the complete end-to-end flow including frontend touchpoints and backend integration

## Research Focus Areas

- Frontend implementation patterns
- Backend API integration (Simple Forms API, Lighthouse Benefits Intake)
- Document processing and validation
- Email notifications and confirmations
- S3 archival and audit trails
- Integration with existing VA.gov applications (e.g., pensions, disability)