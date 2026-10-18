# AIS Security Overview

## 1. Purpose and Scope

This overview summarizes controls visible in the repository for the Next.js frontend, FastAPI backend, SQL Server application data, document processing, AI/retrieval integrations, and documented container deployment. It is not a penetration-test report or confirmation of production configuration.

## 2. System Overview

The browser uses the Next.js frontend, which communicates with authenticated FastAPI REST and WebSocket endpoints. The backend checks ownership for chat/document access, stores application records in SQL Server, processes uploaded bytes in-process, and may call a configured LLM, local SQLite FTS5 knowledge base, and Tavily. Microsoft Entra ID is used when OIDC SSO is enabled.

See [system architecture](../architecture/system-architecture.md) and [data flow](../architecture/data-flow.md).

## 3. Architecture and Trust Boundaries

The principal boundaries are user/browser to frontend, frontend to backend, backend to SQL Server, backend to local document parsers, backend to LLM provider, backend to Tavily, and backend to Microsoft Entra ID for OIDC. The application has no verified durable blob-storage integration. Administrative access is a separate privileged boundary; its full role model and deployment access controls are **To be confirmed by the AIS team**.

## 4. Authentication

- Local email/password registration/login is implemented.
- Microsoft OIDC SSO with PKCE/state is supported when configured.
- JWT access and refresh tokens are placed in HttpOnly cookies; secure and SameSite settings are configurable.
- Protected REST routes use the current-user dependency. Token expiry and database revocation checks are implemented, though the dependency logs a database revocation-check failure and continues.
- State-changing routes use CSRF validation.
- WebSocket authentication uses a short-lived ticket; the handler also accepts a JWT cookie fallback. That fallback validates JWT signature/expiry but does not perform the same SQL revocation check.
- MFA is not established by the reviewed application code. Identity-provider MFA policy is **To be confirmed by the AIS team**.
- Configuration code defaults the SSO development bypass on and has a fallback JWT secret. Production must explicitly disable bypass and configure strong secrets; verify deployed settings.

## 5. Authorization

- Authenticated users access their own chats/documents through user ownership filters and chat membership checks.
- Chat creation, rename, and deletion require authentication and CSRF validation.
- Document upload and retrieval require authentication and validate document/chat ownership.
- A separate admin route exists; exact role assignment and least-privilege administration are **To be confirmed by the AIS team**.
- Registered AI tools are calculators and document search, with a separate legacy ContactOut search route. There is no general shell or code execution tool in the active registry.

## 6. User and Tenant Separation

Data access is scoped by user identifiers (`user_id`/`user_uid`) in chat and document queries. Conversation IDs are public UUIDs but are resolved together with the requesting user. No organization/company tenant identifier, per-tenant knowledge-base filter, per-user Tavily-cache partition, or shared-cache isolation mechanism was verified. The product’s intended tenant model is **To be confirmed by the AIS team**.

## 7. Secrets and Credentials

The backend reads secrets/settings from environment variables and a local `chatbot.env` file for development. The deployment guide describes injecting production settings as Azure App Service application settings. No Azure Key Vault integration was verified in this repository.

Secret categories include model-provider API credentials, Tavily credentials, SQL Server credentials, JWT signing key, AES encryption key, and OIDC client credentials. Rotation ownership, production access policy, and Key Vault adoption are **To be confirmed by the AIS team**. A source audit found a hard-coded third-party credential in the legacy ContactOut integration; do not copy it into documentation, and revoke/rotate it if active. See the existing [security review](../../SECURITY-REVIEW.md).

## 8. File-Upload Security

- Supported formats are listed in `backend/document_analysis.py`.
- The upload route enforces a configurable size limit (default 200 MB).
- `validate_file_bytes` checks magic bytes for recognized types when its optional file-type library is available; DOCX package structure receives additional validation. Some text types are checked for printable content.
- Content-type from the client is logged but is not the authoritative format validation.
- No antivirus/malware scanning, isolated parser sandbox, or durable original-file retention was verified.
- New upload bytes are passed to an in-process background task; no raw bytes are passed to the SQL save call.
- User ownership checks protect document records and extracted text.

## 9. API Security

| Control | Repository evidence and limitation |
|---|---|
| Authentication and authorization | Current-user dependency and ownership checks on protected routes. |
| CSRF | `X-CSRF-Token` checks on state-changing routes. |
| Input validation | Pydantic request models, upload size/content checks, parameterized SQL usage in reviewed paths. |
| CORS | Configured origin allowlist with credentials; production origins must be verified. |
| Rate limiting | In-process sliding-window helper is used by selected auth routes; document upload has optional limiter wiring that may be absent. Limits and deployment-wide behavior need verification. |
| Security headers | HTTP middleware applies configured security headers. |
| Error handling | API errors are returned by framework; logs can include exception details. Avoid exposing sensitive data in production error handling. |
| API docs | FastAPI `/docs` is used by the backend container health check; public exposure policy is **To be confirmed by the AIS team**. |
| Admin APIs | Admin router exists; authorization configuration requires deployment review. |

## 10. Data Protection

- Message and extracted-document content are encrypted by the application with AES-GCM only when a valid `ENCRYPTION_KEY` is configured. On missing/invalid key or encryption failure, the helper can return plaintext. This is not unconditional encryption at rest.
- The example database settings request SQL Server connection encryption; production certificate validation and server-side encryption are **To be confirmed by the AIS team**.
- HTTPS and secure cookies are required for production deployment; actual TLS termination/configuration must be verified.
- Backup encryption, backup retention, key backup/rotation, and deletion from backups are **To be confirmed by the AIS team**.
- User data may be sent to external AI/search providers as described in [data privacy](data-privacy.md).

## 11. Logging and Monitoring

Application logs may contain user identifiers, filenames, MIME types, chat/document IDs, processing statuses, error messages, truncated WebSocket input (up to the first 200 characters), retrieval counts, and token-use metadata. Authentication flows log selected user identifiers and status events. No general prompt/response audit store was identified, but provider-side logging is outside this repository.

Log destination, retention, access controls, redaction, alerting, and security-event review process are **To be confirmed by the AIS team**. Existing deployment documentation describes container logs and health checks, not a fully integrated application monitoring/alerting service.

## 12. AI and LLM Security

The AI layer labels documents and retrieved evidence as untrusted and includes tool-round and per-tool time limits. These are defense-in-depth controls, not complete prompt-injection prevention. The security review identifies raw HTML rendering concerns in the frontend and external request risk from assistant output. Prompt injection, sensitive-data disclosure, retrieval poisoning, and output handling need dedicated security testing.

## 13. RAG and Knowledge-Base Security

The knowledge base is a local SQLite FTS5 lexical index with source/framework/version metadata. It is not a vector store. Uploaded-document search is created from documents already selected for a user/chat and is scoped to that in-memory context. Knowledge-base ingestion authorization, source licensing/validation, shared-index access policy, integrity checks, and poisoning controls are **To be confirmed by the AIS team**.

## 14. Agent and Tool Security

The registered tool set contains deterministic sustainability calculators and an uploaded-document search tool, with bounded rounds and tool timeout. Tool outputs are returned to the LLM. A legacy `bd:` ContactOut route can make an external lookup. No write-capable business action or human-approval workflow was identified in the active calculator/document tools. See [agent capabilities](../ai-governance/agent-capabilities.md).

## 15. External Services

| Service | Purpose and data shared | Authentication/failure impact |
|---|---|---|
| OpenAI or Anthropic | Receives prompts/context and returns model output; selected by backend configuration. | Provider credential from environment; failure interrupts or degrades AI responses. Provider-side retention/training terms: **To be confirmed by the AIS team**. |
| Tavily | Receives search query and returns web passages/URLs when search is enabled and triggered. | API credential from environment; failure removes optional live evidence. |
| Microsoft Entra ID | OIDC authentication when real SSO is enabled; receives standard sign-in request and returns identity tokens. | OIDC client configuration; failure prevents that SSO path. |
| ContactOut | Legacy professional search route (`bd:`); query-derived professional search parameters may be sent. | Credential is currently hard-coded in source; see existing security review. Status/necessity and data terms are **To be confirmed by the AIS team**. |
| Azure App Service | A deployment target is documented; runtime data processing depends on actual deployment. | Production configuration and service status are **To be confirmed by the AIS team**. |

OCR runs through local backend libraries; no external OCR service was identified.

## 16. Known Risks and Gaps

| Risk | Affected component / impact | Existing control | Recommended action | Owner | Status |
|---|---|---|---|---|---|
| SSO bypass enabled by configuration default and fallback JWT secret | Auth; possible unauthorized identity issuance if unsafe defaults reach deployment. | Example deployment settings instruct disabling bypass and configuring a key. | Fail closed on unsafe production configuration; verify deployed settings. | AIS platform/security | Open; see `SECURITY-REVIEW.md`. |
| Hard-coded ContactOut credential | Third-party integration; credential disclosure/abuse. | `bd:` legacy route is explicit. | Revoke/rotate if active; load replacement from managed secret storage; assess removal. | AIS integration owner | Open; see `SECURITY-REVIEW.md`. |
| Shared module-level ContactOut follow-up state | Contact search; cross-user result/context leakage risk. | No effective per-user isolation verified. | Scope state to authenticated user/chat and authorize follow-up. | AIS application owner | Open; see `SECURITY-REVIEW.md`. |
| Raw assistant HTML rendering | Frontend; output-driven external requests or unsafe markup. | Referrer/security headers provide partial defense. | Sanitize or avoid raw HTML; restrict remote images and CSP. | AIS frontend owner | Open; see `SECURITY-REVIEW.md`. |
| Conditional plaintext fallback for content encryption | Database content confidentiality depends on correct key configuration and runtime. | AES-GCM available when key is valid. | Enforce key presence in production; test key backup/rotation and migration. | AIS platform/security | Open; configuration-dependent. |
| Per-process WebSocket/OIDC state with multiple workers | Auth flows can fail across workers/restarts. | Short TTL one-time tickets/state. | Use shared short-TTL storage or ensure routing affinity; test. | AIS platform owner | Open; documented in `SYSTEM_ARCHITECTURE.md`. |
| In-process document background tasks | Extraction can remain pending if process stops. | Failed extraction status is recorded for caught errors. | Use durable queue/recovery for production reliability. | AIS backend owner | Open; documented in `SYSTEM_ARCHITECTURE.md`. |

## 17. Security Controls

**Implemented in source:** authenticated routes, per-user ownership filters, HttpOnly cookies, CSRF checks, configurable CORS, security headers, selected route rate limits, file size/content checks, conditional AES-GCM content encryption, bounded tool loop, and user-scoped chat/document reads.

**Not established as deployed or complete:** secure production configuration, unconditional encryption at rest, comprehensive prompt/data redaction, malware scanning, centralized monitoring/alerting, durable file storage, shared worker state, backup controls, or formal security testing. See [threat model](threat-model.md).

## 18. Security Testing and Evidence

Statuses below reflect evidence available from source inspection, not execution of security tests. No test is marked Pass without test evidence.

| Control ID | Security requirement | Test method | Expected result | Current status | Evidence location | Risk | Production blocker | Owner | Remediation action |
|---|---|---|---|---|---|---|---|---|---|
| SEC-01 | Unauthenticated REST requests denied | Automated route tests for missing/expired cookies | 401/denial on protected routes | Not Tested | `backend/dependencies.py` | High | Yes for protected data | Backend | Add/execute auth tests |
| SEC-02 | Cross-user chat/document access denied | Two-user IDOR integration tests | Other user's records return not found/denied | Partial | `backend/ais_db.py`; `backend/routers/documents.py` | High | Yes | Backend/security | Test every read/mutation route |
| SEC-03 | Unsafe production auth defaults blocked | Startup/configuration test with missing production values | Startup refuses unsafe configuration | Fail | `backend/config.py`; `SECURITY-REVIEW.md` | High | Yes | Platform/security | Add production validation |
| SEC-04 | Upload types and size validated | Invalid signature, extension, malformed archive, boundary-size tests | Reject invalid/oversized input safely | Partial | `backend/security.py`; `backend/routers/documents.py` | Medium | Review before production | Backend | Test parsers and configure upload limits |
| SEC-05 | Stored message/document confidentiality | Verify encryption with and without key, including error behavior | Production never silently stores plaintext | Partial | `backend/encryption.py`; `backend/ais_db.py` | High | Yes if data classified confidential | Platform/security | Enforce key and document backup/rotation |
| SEC-06 | Assistant output rendered safely | XSS and remote-image rendering tests | Sanitized content; no unapproved network loads | Fail | `frontend/components/chat/Message.tsx`; `SECURITY-REVIEW.md` | High | Yes | Frontend/security | Sanitize or disable raw HTML |
| SEC-07 | Prompt injection does not cause unauthorized action/data disclosure | Safe direct/indirect injection tests | Untrusted content cannot bypass authorization/tool scope | Not Tested | `backend/LLM.py` | High | Review before production | AI/security | Add adversarial evaluation |
| SEC-08 | API and auth rate limits are effective across instances | Load tests through deployed topology | Limits cannot be bypassed by worker/instance selection | Partial | `backend/security.py`; `backend/app.py` | Medium | To be confirmed | Platform | Verify shared limiter and route coverage |
| SEC-09 | Secret scanning/rotation | Repository and deployed configuration audit | No active secrets in source/client bundle/logs | Fail | `backend/contactout_search.py`; `SECURITY-REVIEW.md` | High | Yes | Security | Revoke/rotate credential; move to secret store |

