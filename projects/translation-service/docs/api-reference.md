# Application interface reference

## Is this a REST API?

No. The current application is a **Streamlit web application**, not a FastAPI service. It does not define public REST routes, OpenAPI schemas, or a separate API server. It is accurate to describe the user-facing surface as a Streamlit UI and the cloud integration as calls made through Azure SDKs. Do not advertise it as a REST API unless one is added.

## User-facing actions

| Screen/action | Behavior |
|---|---|
| Login | Checks a username and password against configured password hashes in `login.yaml`. |
| Upload document | Accepts PDF, DOCX, XLSX, PPTX, PNG, JPG, and JPEG at the Streamlit control. Translation validation permits PDF, DOCX, XLSX, and PPTX; images use the OCR path to produce a PDF. |
| Translate | Detects a source language and applies the fixed English-to-Arabic / other-to-English mapping, submits the document to Azure Document Translation, and displays a download control. |
| OCR | For PDF/image uploads, attempts to create a searchable PDF locally with Tesseract. |
| Download | Returns output through Streamlit's download button. PDF translation may include a best-effort DOCX conversion in a ZIP archive. |
| Change password / logout | Updates the local credentials configuration or clears browser-side authentication state. This is prototype behavior, not production identity management. |

## Integration operations

The application makes SDK-based calls rather than exposing these as its own endpoints:

- Azure Blob Storage: upload source blob; poll for and download target blob; attempt cleanup of source and target blobs.
- Azure AI Document Translation: submit a document translation job and wait for its result.

Service endpoints and credentials are configured outside this reference. Never publish real endpoints, keys, SAS values, or account identifiers.

## Adding a REST API

A separate API would be appropriate only if there is a need for integrations, independent frontend/backend deployment, or stable machine-to-machine contracts. It would require explicit authentication and authorization, request/response schemas, job lifecycle endpoints, upload limits, idempotency, error contracts, and OpenAPI documentation. For the current browser-first prototype, Streamlit is an accurate and sufficient description.
