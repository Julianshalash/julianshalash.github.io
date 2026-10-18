Purpose: Distinguish implemented product areas from development work visible in source and repository configuration.

Status: In active development. Scope 3 and commercial features are not yet implemented.

| Feature | Status | Notes |
|---|---|---|
| User, company and project workflows | ✅ Implemented | SQL Server CRUD and role-aware routes exist; audit authorization consistency. |
| Workbook templates and data import | ✅ Implemented | Excel parsing, validation, pending upload preview/confirmation and template version/checksum metadata exist. |
| Scope 1 and Scope 2 tracking/results | ✅ Implemented | Inputs/results and dashboards are present; full calculation/factor methodology needs confirmation. |
| Dashboard analytics | ✅ Implemented | Company/project/category/scope/sector endpoints and frontend charts. |
| AI-assisted form suggestions | ✅ Implemented | Direct OpenAI completion with preset prompts. |
| AI-generated methodology reports | ✅ Implemented | Background jobs, structured context, LLM section generation, validation and encrypted local file output. |
| PDF report output | 🚧 In progress | PDF builder exists, but report service comments say PDF generation is disabled; Word output is active. |
| Scope 3 emissions | 📋 Planned | Do not treat report-schema fields as a working calculator or importer. |
| Pricing, plans, billing, subscriptions and commercial multi-tenancy | 📋 Planned | No commercial implementation found. |
| Docker packaging | 📋 Planned | No Dockerfile or Compose manifest. |
| Azure hosting for web apps and databases | 📋 Planned | Target is to host the Next.js frontend, FastAPI API, SQL Server and MongoDB on Azure. Compute/database products, network topology and IaC are TODO: confirm; Blob template integration exists. |
| CI/CD | 📋 Planned | No GitHub Actions workflow found in tracked project files. |
| Centralized secrets / external DB configuration | 🚧 In progress | SQL Server/MongoDB connection values are hardcoded in application code; move to secret-backed configuration. |
| SQL schema/migrations and backend dependency manifest | 🚧 In progress | No checked-in relational DDL/migrations or backend requirements manifest found; local bootstrap is unclear. |
| Complete tenant authorization and security tests | 🚧 In progress | Authorization is route-specific; establish and test a complete policy. |
| Distributed report jobs, storage and caching | 📋 Planned | Current implementation includes background tasks/local files and process-local cache fallback. |
| AI quality evaluation and monitoring | 📋 Planned | Tests cover selected report defects; no systematic model evaluation/monitoring plan is implemented. |
