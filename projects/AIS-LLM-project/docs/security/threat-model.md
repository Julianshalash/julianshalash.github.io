# AIS Threat Model

## 1. Scope

This threat model covers the browser/Next.js frontend, FastAPI APIs and WebSocket, authentication, SQL Server application records, document extraction/OCR, SQLite FTS5 knowledge base, LLM and Tavily calls, legacy ContactOut integration, and documented Docker/Azure deployment. Actual production infrastructure and configuration are **To be confirmed by the AIS team**.

## 2. Assets

- User identities, credentials, sessions, JWTs, and OIDC state.
- Uploaded bytes during processing, filenames, extracted document text, and OCR results.
- Chat history, project information, and assistant responses.
- SQL Server account, token, chat, message, and document records.
- Knowledge-base content, source metadata, and local index file.
- LLM context and outputs, search queries/results, provider credentials, database credentials, encryption/signing keys.
- Application, authentication, and document-processing logs.

No embeddings/vector store was identified.

## 3. Actors

**Legitimate actors:** unauthenticated visitors, registered/local users, Microsoft Entra ID users when SSO is enabled, application administrators, and operators who configure/deploy the service.

**Potential threat actors:** malicious or compromised users; a user attempting cross-account access; an attacker submitting crafted files or prompts; malicious content authors whose content is retrieved; compromised third-party providers or credentials; and external network attackers. These are threat-model actors, not claims of observed incidents.

## 4. Entry Points

- Local login/registration, password reset, OIDC start/callback, token refresh, CSRF and WebSocket-ticket endpoints.
- Chat WebSocket messages and document IDs.
- Document upload and document/raw retrieval endpoints.
- Chat and admin APIs.
- Knowledge-base ingestion scripts/files.
- Tavily results and their page content/URLs.
- LLM responses and registered tool-call arguments.
- Container configuration and environment secrets.

## 5. Trust Boundaries

See [system architecture](../architecture/system-architecture.md#6-trust-boundaries) and [data flow](../architecture/data-flow.md#8-data-flow-diagram). Important boundaries include browser-to-backend, backend-to-database, parser handling of untrusted uploads, and backend-to-LLM/search/identity providers.

## 6. STRIDE Analysis

| STRIDE category | Example AIS threat | Verified controls and gaps | Risk |
|---|---|---|---|
| Spoofing | Forged/replayed session or identity; unsafe SSO bypass default. | JWT signature/expiry validation, OIDC PKCE/state, cookie auth; development bypass defaults on in config; WS cookie fallback skips DB revocation. | High |
| Tampering | Malformed uploads, altered prompts/retrieval records, or forged tool arguments. | Size/content validation, Pydantic models, parameterized SQL in reviewed paths; no malware sandbox or comprehensive knowledge-base integrity gate verified. | Medium/High |
| Repudiation | User disputes a chat, upload, or administrative action. | Some operations log user/chat/document IDs; complete immutable audit trail and log retention are not verified. | Medium |
| Information disclosure | Cross-user access, prompt leakage, exposed credentials, logs, provider sharing. | Ownership checks and conditional content encryption; ContactOut secret is in source, shared state risk and raw HTML issue are documented, provider/privacy settings unconfirmed. | High |
| Denial of service | Large/malformed files, repeated model calls, oversized context, search abuse. | Upload size limit, selected route rate limits, page cap, bounded tools and timeouts; in-process rate limiting and background tasks may not be robust across workers. | Medium/High |
| Elevation of privilege | Unauthorized admin action or prompt-induced tool invocation. | Authenticated endpoints and registered tool allowlist; admin role design and adversarial tool authorization tests need confirmation. | High |

## 7. AI-Specific Threats

- **Direct prompt injection:** a user attempts to override intended behavior or extract hidden context.
- **Indirect prompt injection:** uploaded documents or web passages contain instructions aimed at the model.
- **Sensitive-data disclosure:** chat/document context is sent to a configured external model; model output may expose context.
- **Excessive agency:** tool loop is bounded, but unexpected model tool selection still needs authorization tests.
- **Knowledge-base poisoning:** source ingestion/provenance controls are not established.
- **Embedding/vector weaknesses:** no vector or embedding system was found in this repository; **Not Applicable to current indexed implementation**, subject to confirmation of external infrastructure.
- **Citation manipulation:** content can contain misleading sources; citation markers map to retrieval order and do not validate claim support.
- **Resource/token consumption:** prompts, document context, retrieval and repeated turns can consume resources; tool rounds and selected limits mitigate some paths.
- **Unsafe output processing:** existing security review identifies assistant raw-HTML rendering and possible external requests.
- **Supply-chain risk:** parser/OCR/provider libraries and Docker base images are dependencies; automated dependency governance is **To be confirmed by the AIS team**.

## 8. Threat Register

Likelihood and impact are qualitative engineering assessments, not measured incident probabilities.

| ID | Threat and attack scenario | Affected component | Likelihood | Impact | Risk | Existing mitigation | Recommended mitigation | Owner | Status / evidence |
|---|---|---|---|---|---|---|---|---|---|
| TM-01 | Unsafe development SSO bypass is enabled in a deployed configuration and an attacker obtains a token for the bypass identity. | Auth/config | Medium | Critical | High | Deployment guide documents production override. | Fail startup if bypass enabled outside development; remove signing-key fallback. | Platform/security | Open; `backend/config.py`, `SECURITY-REVIEW.md` |
| TM-02 | Source-exposed ContactOut credential is abused. | ContactOut integration | Medium | High | High | Explicit legacy `bd:` entry path. | Revoke/rotate if active; remove source secret; use managed configuration. | Integration/security | Open; `backend/contactout_search.py`, `SECURITY-REVIEW.md` |
| TM-03 | Shared ContactOut follow-up state allows one user's search context to influence another user's result. | ContactOut state | Medium | High | High | No adequate isolation verified. | Bind state to user/chat; authorize retrieval; add isolation tests. | Backend | Open; `SECURITY-REVIEW.md` |
| TM-04 | Model/document content renders as unsafe HTML or causes an external browser request. | Frontend renderer | Medium | High | High | Security headers provide partial defense. | Sanitize/disable raw HTML; block remote images; apply restrictive CSP. | Frontend/security | Open; `SECURITY-REVIEW.md` |
| TM-05 | A user accesses another user's chat/document by changing an identifier. | API/database | Medium | High | High | Ownership predicates and chat membership checks. | Automated IDOR tests for all endpoints and legacy IDs. | Backend/security | Not tested end-to-end |
| TM-06 | Crafted PDF/Office/image input exploits parser/OCR library or exhausts resources. | Upload/parser | Medium | High | High | Size cap, content checks, PDF page cap, image width cap. | Malware scanning, process isolation, CPU/memory/time limits, dependency updates. | Platform/backend | Partial; no sandbox verified |
| TM-07 | Uploaded text or a web passage injects instructions and elicits disclosure or inappropriate tool use. | LLM/RAG | High | High | High | Context labelled untrusted; tool allowlist and bounded rounds. | Adversarial evaluations, minimize context, enforce authorization outside the model, output checks. | AI/security | Not tested |
| TM-08 | LLM/search provider retains sensitive prompts or queries beyond AIS expectations. | External services | Medium | High | High | Provider configurable; no verified data-processing settings. | Confirm provider contracts, retention/training settings, region, and approved data classes. | Privacy/procurement | To be confirmed |
| TM-09 | Revocation check fails open on REST or differs on WebSocket cookie fallback. | Auth | Medium | High | High | JWT signature and expiry are checked. | Fail closed where revocation is required; align REST/WS checks. | Backend/security | Open; documented in `SYSTEM_ARCHITECTURE.md` |
| TM-10 | Multi-worker process-local ticket/SSO state causes auth failure or inconsistent behavior. | Auth deployment | Medium | Medium | Medium | Short-lived one-time tickets/state. | Shared short-TTL store or verified sticky routing; test across workers/restarts. | Platform | Open; documented in `SYSTEM_ARCHITECTURE.md` |
| TM-11 | Retrieval index is poisoned or stale, producing incorrect sustainability guidance. | SQLite FTS5 | Medium | High | High | Framework/version/source metadata; source citations. | Controlled ingestion, provenance checks, version review, integrity and release process. | Knowledge owner | Controls to be confirmed |
| TM-12 | Repeated uploads/search/model requests consume backend/provider resources. | API/LLM/OCR | Medium | Medium | Medium | Upload limit, OCR cap, tool rounds/timeouts, selected auth rate limits. | Verify route-wide distributed limits and monitor cost/queue saturation. | Platform | Partial; tests not run |

## 9. Security Test Scenarios

These are proposed safe tests only; they were not executed as part of this documentation task.

1. Call each protected API without a cookie and with expired/revoked tokens.
2. Use two test accounts to request, rename, delete, and attach each other's chat/document IDs.
3. Submit valid/invalid signatures, mismatched extensions, malformed Office archives, traversal-like filenames, empty files, and files just below/above the configured limit.
4. Upload a document containing benign prompt-injection test text and verify it cannot cause access to another user's content or unregistered actions.
5. Return benign injection-like text from a controlled test search fixture and verify source instructions are treated as data.
6. Attempt direct tool invocation with unknown tool names, malformed arguments, excessive calls, and timeout conditions.
7. Scan repository bundles, build artifacts, and logs for credentials, tokens, prompt content, and unnecessary personal data.
8. Test model output containing HTML/script-like markup and remote image links in a browser with network capture.
9. Run load tests on upload/OCR, chat, web search and tool calls across multiple workers; verify per-user limits and resource bounds.
10. Request retrieval of restricted knowledge-base content and verify the index policy matches intended user/tenant access.

