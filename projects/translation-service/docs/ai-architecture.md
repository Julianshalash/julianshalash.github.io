# AI and language-processing architecture

## What the application uses

The translation workflow delegates document translation to **Azure AI Document Translation** through the Azure Python SDK. The application does not host or train a translation model and does not call a custom LLM endpoint.

Local processing includes:

- `langdetect` for source-language detection from extracted text.
- Tesseract OCR through `pytesseract` for scanned PDF/image text recognition.
- Python document libraries to extract content and apply limited formatting/direction changes.
- `deep-translator`/Google Translate in the filename helper to translate non-ASCII filenames to English when needed. This is separate from document-content translation and sends the filename text to that external translation service.

## Translation flow and chunking

The application uploads a document to Blob Storage, then submits source and target container URLs to Azure AI Document Translation. It waits for the SDK poller to complete and downloads the matching output blob.

There is no application-level chunking, segmentation, batching, prompt construction, or chunk retry logic in this repository. The code submits a document-level translation job and leaves internal document processing to the Azure service; its internal chunking behavior is not visible or configured here. The application sets the translation category to `general`. Glossary-related objects and arguments are present in the code path, but glossary application is commented out.

## Language direction

The application extracts text and detects its language from a sample of up to 1,000 characters. English maps to Arabic. Any other detected language maps to English. A failed extraction in the Streamlit translation path defaults to English. This is a fixed bilingual workflow rather than arbitrary source/target language selection.

## OCR flow

The OCR module renders PDF pages using PyMuPDF and runs Tesseract with a configured set of language codes. It creates an output PDF with a page image and an invisible text layer. OCR is performed locally by the application host; it is not an Azure OCR API call. The current code hard-codes a Windows Tesseract executable path, and the Docker image neither installs Tesseract nor configures that path for Linux.

## Model/data boundaries

| Operation | Runs where | Data sent to external service |
|---|---|---|
| Text extraction for language detection | Application process | None by this operation alone |
| OCR | Application host using Tesseract | None to Azure OCR; the OCR output may subsequently be sent for document translation |
| Document translation | Azure AI Document Translation | The document is made available to the service through Azure Blob Storage |
| Filename translation | `deep-translator` helper | Non-ASCII filename base text may be sent to the configured external translator |

The application should disclose these processing boundaries to users and avoid processing confidential content until service settings, contracts, access controls, and retention requirements have been reviewed.
