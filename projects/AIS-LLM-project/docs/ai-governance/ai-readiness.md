# AIS AI Readiness

## Purpose

This checklist summarizes AI governance capabilities evident in the repository and open controls that require product, privacy, security, or operational confirmation. It is not a certification or claim of regulatory compliance.

## Readiness Assessment

| Area | Repository evidence | Status | Follow-up |
|---|---|---|---|
| Defined intended use | Sustainability question answering and building/infrastructure document review are described in backend prompts and frontend/product text. | Partial | AIS team to approve intended-use statement, supported users, and exclusions. |
| Model/provider inventory | OpenAI and Anthropic integrations are available through backend configuration. | Partial | Record deployed provider/model versions, region, retention/training terms, and approval owner. |
| Data inventory | SQL Server, SQLite FTS5, process memory, logs, Tavily, and model request data are visible in code. | Partial | Confirm production data locations, backups, retention, and provider-side handling. |
| Human oversight | Responses are presented to users; no approval gate for calculator/search tools was found. Professional validation is recommended in prompts/tool notes. | Partial | Define review requirements for high-impact or regulated use. |
| Agent boundaries | Registered calculators/document search, optional web search, bounded tool loop, and per-tool timeout. | Partial | Test tool authorization and define policy for future write-capable tools. |
| Prompt-injection resilience | Prompts mark uploaded/retrieved content as untrusted. | Partial | Add direct/indirect prompt-injection evaluations and document residual risk. |
| Retrieval provenance | Local FTS5 source metadata and citation markers are present; Tavily results include URLs/source labels. | Partial | Define source approval, versioning, ingestion review, citation quality tests. |
| Document/OCR quality | Native-first per-page extraction, selection heuristics, OCR backends and warnings exist. | Partial | Maintain representative OCR regression corpus; track confidence/quality and page-level failures. |
| Output safety | Source rendering behavior was flagged in existing security review. | Fail | Sanitize/disable unsafe HTML and test browser output handling. |
| Privacy and transparency | Context can be sent to third-party model/search providers. | Partial | Confirm provider contracts, retention, notice/consent, region, and data-classification rules. |
| Security testing | Source-level controls exist; this documentation task did not execute a security test suite. | Not Tested | Run threat-model scenarios and preserve evidence. |
| Monitoring and incident response | Logs and container health checks exist; comprehensive AI monitoring/incident procedures not verified. | Partial | Define model abuse, privacy incident, provider outage, and AI-quality escalation procedures. |
| Change management | Runtime model/provider settings are configurable. Formal evaluation/release gates not found. | To be confirmed | Establish model/version approval, test thresholds, rollback and change log. |
| User feedback and appeals | No dedicated feedback or appeal store/workflow was identified. | To be confirmed | Confirm whether user correction/complaint mechanisms exist outside this repository. |

## Minimum Operational Readiness Actions

1. Confirm the production model provider, model identifiers, provider data-processing terms, and allowed data classes.
2. Disable development SSO bypass and require production signing/encryption secrets.
3. Rotate the legacy ContactOut credential if active and remove source-level secret handling.
4. Address raw HTML rendering and test injection-resistant output.
5. Perform user-isolation, upload/parser, prompt-injection, retrieval, and resource-limit tests.
6. Confirm retention, deletion, backups, logging access, and external-provider deletion processes.
7. Maintain a versioned evaluation set covering sustainability answers, source/citation quality, multilingual behavior, and OCR edge cases.
8. Define human review/escalation for safety-critical, contractual, regulatory, or engineering decisions.

## Evidence and Ownership

No named AI governance, privacy, security, data-owner, or model-risk owner was verifiable from the codebase. **Owner assignments and approval authority are to be confirmed by the AIS team.** Existing source documentation includes [security review](../../SECURITY-REVIEW.md), [system architecture](../../SYSTEM_ARCHITECTURE.md), and [Azure deployment guidance](../../AZURE_DEPLOYMENT_GUIDE.md).

