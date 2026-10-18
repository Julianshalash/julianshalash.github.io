# Data model

## Persistence model

There is no application database or persistent domain model in the repository. The application coordinates files and session state rather than storing structured business records.

| Entity / data | Representation | Location |
|---|---|---|
| User credentials | YAML entries containing usernames and password hashes | `Code/login.yaml` at runtime |
| Browser authentication state | Streamlit session state plus browser `localStorage`/cookie values | Browser and application process |
| Uploaded source document | File bytes, then a blob keyed by the uploaded filename | Application memory/temp storage and Azure `source` container |
| Translated document | File bytes, then a blob keyed by the matching filename | Azure `target` container and local temp storage |
| OCR output | Searchable PDF file | Application host temporary storage |
| Translation job | Azure SDK poller/result in process memory | No application-side durable job record |
| Glossary | Container name and URL construction exist in configuration/orchestration | No active glossary workflow is configured |

## Blob naming and lifecycle

The source and target containers are fixed in application configuration. Uploads use the original filename and set overwrite behavior. This means filenames are not unique identifiers and concurrent jobs with the same name may collide. After the translated blob is downloaded to application-local temporary storage, the application attempts to delete same-name blobs from the source and target containers. This happens before the user's browser download; cleanup success is not durably recorded.

## Recommended future model

If the application grows beyond a single-user prototype, introduce a durable job record with an opaque job ID, owner/tenant ID, original and output object references, selected language pair, processing state, timestamps, retention expiry, and sanitized error status. Use per-job blob paths and explicit ownership checks. Avoid storing document text or secrets in job metadata.
