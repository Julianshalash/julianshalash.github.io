Purpose: Summarize security controls visible in code and the gaps requiring attention before production use.

Status: In active development. Scope 3 and commercial features are not yet implemented.

## Authentication and authorization

- ✅ Implemented: bearer JWT authentication through FastAPI `HTTPBearer`; the dependency validates signature/expiry and checks an in-memory session-activity expiry.
- ✅ Implemented: access and refresh token creation; refresh-token records are stored in SQL Server and logout revokes a token record.
- ✅ Implemented: password hashing utilities use bcrypt; password reset flows and role checks exist.
- ✅ Implemented: role hierarchy includes Developer, Alpin Admin, Alpin Manager, Admin, Manager and User.
- 🚧 In progress: permissions and tenant ownership are not applied consistently across every route. Some handlers enforce minimum role; others only require a valid token or use repository filters.
- **TODO: confirm** session storage behavior across workers, token revocation enforcement for access tokens, and company ownership enforcement for every operation.

## User/company separation

Several query paths filter projects/users by the authenticated user’s company and/or project membership, and report routes validate project accessibility. This is not a complete row-level authorization layer: implementation varies by endpoint, and global comparison/admin routes intentionally expose broader sets based on role.

**TODO: confirm** a complete endpoint-by-endpoint authorization matrix and add automated cross-company access tests before presenting this as tenant isolation.

## Secrets and configuration

Observed configuration variable **names only**:

`JWT_SECRET`, `JWT_ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `PASSWORD_RESET_EXPIRE_HOURS`, `REPORT_ENCRYPTION_KEY`, `REPORT_TOKEN_SECRET`, `OPENAI_API_KEY`, `AZURE_STORAGE_CONNECTION_STRING`, `AZURE_STORAGE_CONTAINER_NAME`, `SMTP_PASSWORD`, `FRONTEND_RESET_PASSWORD_URL`, and `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_DB`, `REDIS_SSL`, `REDIS_SOCKET_CONNECT_TIMEOUT`, `REDIS_SOCKET_TIMEOUT`, `REDIS_RETRY_ON_TIMEOUT`.

🚧 Critical configuration gap: SQL Server and MongoDB clients are constructed with connection settings in source. The JWT module has a development fallback when `JWT_SECRET` is absent, and some secrets are loaded from ignored/local `.env` files. Move all credentials to a managed secret source, reject weak/missing values at startup, and rotate any credential that has been committed historically.

No values, hostnames, database credentials, API keys or internal URLs are included in this documentation.

## Data protection

- ✅ Report artifact service encrypts generated files using Fernet and validates encrypted-file SHA-256 hashes before decryption; signed download tokens have a limited validity period.
- 🚧 Transport security: assistant calls use HTTPS; API/database production TLS is not configured or documented in repository code. **TODO: confirm** HTTPS termination and SQL Server/MongoDB TLS.
- 🚧 At-rest protections for SQL Server, MongoDB, Blob Storage and backups depend on deployment configuration; no evidence of platform encryption settings or key rotation is included.
- 🚧 Local files and BSON/SQL backup artifacts are present in the repository. Treat them as potentially sensitive data, remove them from public distribution where appropriate, and review history/artifact handling.

## API security

- ✅ Pydantic models and template-specific validators enforce selected input shapes, field constraints and workbook structure.
- ✅ SlowAPI rate limiting is registered; explicit limits exist for login, assistant, report and some other routes, with a role-based limiter in dashboard/report usage.
- 🚧 CORS allows specific development origins in code; production origins need explicit configuration and review.
- 🚧 Rate limits and cache configuration do not establish a production-wide distributed limiter; local fallback behavior is process-local.
- 🚧 Upload limits/content-type scanning and broader schema validation are **TODO: confirm**.

## AI-specific risks

The active assistant is authenticated and uses fixed templates, but user-supplied company/project context is included in prompts. The report generator sends structured report data to an external LLM. Prompt injection controls, sensitive-field minimization, provider retention settings and output-review governance need formal design.

## Docker and Azure

The intended deployment target is Azure for the Next.js frontend, FastAPI backend, SQL Server and MongoDB; this deployment is 📋 Planned and no Azure hosting or database manifests are present. Azure Blob Storage is integrated for template downloads.

📋 Planned security controls for Azure include managed identity where supported, managed secret storage, private database networking, HTTPS/TLS, encryption-at-rest verification, least-privilege resource access, logging/monitoring, backups and recovery testing. **TODO: confirm** the selected compute and database products, authentication mechanisms, network controls and compliance/data-residency requirements. Docker image hardening, non-root runtime and image scanning are also planned because no container definitions are present.

## Risk register

| Risk | Current mitigation | Gap / planned improvement |
|---|---|---|
| Credentials embedded in source/config | Some secrets are read from environment or local `.env` files. | Move DB credentials and all secrets into managed secret storage; remove leaked values from public history; rotate them. |
| Weak JWT fallback or missing secret | JWT signing configured via environment where available. | Fail closed on missing/weak secrets; rotate keys and define lifecycle. |
| Cross-company data exposure | Some repositories filter by company/project and report handlers check project access. | Enforce consistent authorization at service/repository boundaries; test tenant isolation for all routes. |
| Token/session invalidation gaps | Refresh records can be revoked; session inactivity is checked. | Validate access-token revocation semantics and cross-process session state. |
| Sensitive AI prompt data | Assistant key stays server-side; prompt templates constrain supported fields. | Minimize payloads, document provider retention, defend against prompt injection and review generated claims. |
| Unencrypted or unmanaged database/blob data | Report files are encrypted before local persistence. | Verify managed at-rest encryption, TLS, key rotation, backups and access policies for every store. |
| Unsafe uploads | Excel extension, workbook structure and numeric checks are present. | Add explicit size limits, malware/content checks and broader parser abuse tests. |
| Rate-limit bypass or inconsistent throttling | SlowAPI limits and Redis-capable limiter/cache components exist. | Verify distributed configuration and fail-safe behavior under multiple instances. |
| Development CORS allowlist in production | Explicit origins are listed in middleware. | Externalize per-environment allowlists and verify no debug/test origin remains. |
| Data in repository artifacts | No public documentation reproduces backup contents. | Review SQL/BSON exports, logs and generated files; exclude real records from public repo. |
