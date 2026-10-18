Purpose: Assess data governance and operational readiness for responsible AI use based on the code currently present.

Status: In active development. Scope 3 and commercial features are not yet implemented.

## Current maturity

| Dimension | Status | Evidence and gap |
|---|---|---|
| Structured context | 🚧 In progress | Reports combine request fields with database-derived category/totals; completeness and source authority vary. |
| Provenance | 🚧 In progress | Activity records can carry emission-factor source/reference and template version/checksum; not all result paths guarantee populated provenance. |
| Data validation | ✅ Implemented | Typed request models, workbook structural checks, numeric checks and report validators exist for selected paths. |
| Scope coverage | 🚧 In progress | Scope 1/2 paths exist. Scope 3 is not an implemented calculation capability. |
| Model evaluation | 🚧 In progress | Selected report defects/validation are tested. No versioned benchmark, quality threshold or systematic hallucination evaluation is evident. |
| Privacy governance | 🚧 In progress | API keys are kept server-side, but prompts may include company/report data. Provider retention and data minimization policies are TODO. |
| Monitoring | 📋 Planned | No explicit LLM cost, quality, safety or drift monitoring pipeline found. |
| Human review | 📋 Planned | No end-to-end approval/sign-off control for AI-generated facts is established by code. |

## Data quality and governance readiness

The following are readiness requirements and recommendations, not claims that every control is already implemented:

Before using AI output in business or public reporting:

- Ensure company, project, reporting period, activity, units and calculation basis are complete and internally consistent.
- Preserve factor source, geography, reporting year, units, version, and any user override alongside result data.
- Reject or clearly flag missing values, stale factors, inconsistent period boundaries, proxy inputs and unverified spreadsheet totals.
- Keep identity/access checks attached to every query and report job; test that one company cannot read another company’s data.
- Establish retention, deletion, backup and data-subject workflows across SQL Server, MongoDB, report files, logs and any external model provider.
- Scope 3 remains 📋 Planned; AI-generated Scope 3 narratives or values must not be accepted as calculated results.

## Responsible AI controls and gaps

✅ Implemented: fixed suggestion templates, server-side API key usage, report prompts emphasizing supplied evidence, selected placeholder/totals validators, and a report-generation rate limit.

🚧 In progress: structured prompt/context separation is limited; user-provided fields can affect prompt content. No comprehensive prompt-injection defense or red-team suite is evidenced.

📋 Planned: minimize sensitive context, publish AI-use disclosure, require reviewer approval for consequential outputs, improve output provenance and traceability, evaluate across representative domains and monitor model/provider changes.