**PII Data Masking using SSIS & KingswaySoft Productivity pack**

Personally Identifiable Information (PII) refers to any data that can be used to identify an individual — such as name, address, email, phone number, SSN, customer IDs, or financial account details. Protecting PII is critical for security, privacy, regulatory compliance (GDPR, HIPAA, CPRA, etc.), and to prevent misuse of sensitive information. In analytics, testing, and data-sharing environments, masking PII helps ensure that real identities are never exposed while still preserving data usability.

This repository demonstrates a practical approach to implementing PII data masking using SQL Server Integration Services (SSIS) along with the KingswaySoft Data Anonymizer component. The goal of this repo is to help data engineers and ETL developers understand how to:

  - Identify PII fields in source systems
  - Apply consistent and repeatable masking rules
  - Use KingswaySoft anonymization functions in SSIS
  - Preserve data integrity for downstream reporting & testing
  - Build reusable masking patterns for enterprise ETL pipelines

The examples included in this repo show end-to-end workflows covering configuration, transformation logic, and best-practice patterns — making it easier for teams to adopt a secure and standardized approach to PII masking across environments.

**Tools needed:**
Microsoft SQL Srver Integration Service (SSIS)
MySQL database
Kingswaysoft productivity pack for Data anonymization - https://www.kingswaysoft.com/solutions/developer-productivity/data-generation-anonymization/data-anonymizer-component
