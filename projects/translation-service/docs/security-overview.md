# Security overview

## Scope and current state

This overview describes behavior visible in the repository; it is not a penetration test or an Azure configuration audit. The application uses Azure AI Document Translation and Azure Blob Storage, but the repository contains no Azure deployment configuration, network policy, managed identity setup, or infrastructure-as-code. No real credentials, connection strings, IP addresses, or internal service URLs are included here.

## Authentication and authorization

- The Streamlit UI has a username/password form. Password verification uses Passlib's bcrypt context against password hashes loaded from `login.yaml`.
- Password changes update the credentials file. There is no external identity provider, MFA, role model, or documented account lifecycle.
- Authentication persistence reads and writes a username in browser `localStorage` and a JavaScript-managed cookie. The cookie is not `HttpOnly`, and the code restores `authenticated` based on the presence of browser-held identity data rather than validating a server-side session. This is not suitable for production authentication.
- No authorization checks by tenant, role, or per-document ownership are present.

## Data protection

- Translator credentials and Storage access values are read by configuration code from environment variables and a dotenv file. Deployment secrets should be injected at runtime and excluded from source control and container layers.
- The application uses Azure SDKs and constructs Blob container URLs with a SAS token. SAS scope, permission, and expiry are not enforced by application code and must be set in Azure.
- The repository does not configure or verify HTTPS-only transport, encryption-at-rest settings, private endpoints, customer-managed keys, or storage lifecycle policies. Verify these directly in the Azure resources before handling sensitive documents.
- Temporary files and translated outputs are written to local disk. Cleanup is attempted but not comprehensively guaranteed or auditable.

## API and input security

- The application exposes a Streamlit UI, not a separately defined REST API. Streamlit itself handles browser requests.
- File extension and size checks are present, but content-based type validation, malware scanning, archive-bomb protection, and comprehensive parser hardening are not implemented.
- Blob names are based on user-provided filenames and use overwrite behavior. This introduces collision and cross-request interference risk.
- Output validation checks only file-size ratio; it does not confirm safe content, structural integrity, or translation correctness.
- Logging and error messages can include exception details; production logging should avoid document content and secret values.

## Docker and Azure security

- The Dockerfile uses a slim Python base image and exposes Streamlit port `8501`; no non-root user, health check, image scanning, or runtime hardening is configured.
- The Dockerfile copies the full build context, and no `.dockerignore` is present in the inspected project tree. Sensitive local configuration could be copied into an image if included in the Docker build context.
- The Docker command points to a missing application file, and the OCR implementation uses a Windows-specific executable path. The checked-in container is not a verified deployment artifact.
- Azure hosting, network boundaries, managed identity, Key Vault, private access, and resource-level access policies are not represented in the repository.

## Risk register

| Risk | Current mitigation | Gap / planned improvement |
|---|---|---|
| Browser-side authentication can be forged or replayed | Password hashes are checked at login | Replace localStorage and JavaScript cookie trust with server-validated, secure, HttpOnly sessions or a supported identity provider; add MFA and authorization. |
| Secrets may be exposed in source or images | Configuration is read from named environment variables | Remove secrets from tracked dotenv files; add `.dockerignore`; inject secrets at runtime from a secret manager; rotate any exposed credentials. |
| Broad or long-lived SAS access | Blob access requires a SAS token | Issue short-lived, least-privilege SAS or use managed identity; scope access to per-job blobs and verify expiry/permissions. |
| Sensitive document data persists after processing | Cleanup of matching source and target blobs is attempted after the translated file is downloaded to application-local storage, before the browser download | Deletion is best effort, is not tied to the user's browser download, and is not durably verified; define retention guarantees, verify cleanup outcomes, and add lifecycle policies and auditable deletion. |
| Uploads may contain malicious or malformed files | Supported extensions and a size limit are checked | Validate content types, scan files, harden parsers, isolate processing, and add per-format resource limits. |
| Same-name documents may overwrite each other | The upload path has a fixed source/target container structure | Use opaque per-request blob names, user/job isolation, and concurrency-safe cleanup. |
| Network/data encryption settings are unverified | Azure SDK clients are used | Enforce and verify HTTPS, secure Azure resource configuration, private networking where required, and encryption-at-rest policies. |
| Container image may be insecure or nonfunctional | A slim Python base image is used | Fix the entry point and OCR dependencies; run as non-root, pin/scan base images, add health checks, and avoid embedding credentials. |
| Translation output is treated as valid based on size only | File-size ratio check exists | Add structural, format-specific, and human-quality checks; avoid presenting the size heuristic as translation validation. |

## Public documentation hygiene

Only environment-variable names are listed in this documentation. Replace values with placeholders such as `<API_KEY>` and `<STORAGE_SAS_TOKEN>` in examples, and do not publish real account URLs, credentials, user records, or customer documents.
