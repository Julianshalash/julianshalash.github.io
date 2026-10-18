# AI readiness

## Current maturity

The repository has a working-in-code integration pattern for document translation, but it is an early prototype rather than a governed AI product. Translation is provided by Azure AI Document Translation; OCR and language detection are local utility steps. There is no model training, model registry, custom inference service, production monitoring, or evaluation pipeline.

| Area | Current state | Gap |
|---|---|---|
| Data quality | Extracts document text for language detection; supports several office formats and PDF | No quality scoring, extraction confidence threshold, language-detection confidence handling, or content validation. |
| Translation evaluation | Performs a file-size-ratio check | No semantic quality evaluation, reference set, human review workflow, or per-language metrics. |
| OCR quality | Tesseract OCR can produce searchable PDF output | OCR confidence is not surfaced as a quality gate; Linux runtime dependencies are missing from the container. |
| Data governance | Uses Azure Blob Storage source/target containers and attempts cleanup | No documented classification, ownership, consent, retention schedule, deletion assurance, or audit record. |
| Privacy | Configuration is environment-backed | Browser-side authentication is weak; local files and cloud resources lack a verified deployment security posture. |
| Responsible AI | No specific safeguards or user-facing translation caveats are implemented | Add human review for high-impact documents, disclose service/data boundaries, address bias and language coverage, and provide correction paths. |
| Monitoring | Basic UI progress and error messages | No production metrics, structured event trail, alerting, service-level objectives, or quality monitoring. |

## Data quality and governance recommendations

- Establish a supported-format and language matrix; test representative documents including tables, embedded images, RTL text, scanned pages, and mixed-language content.
- Measure OCR and translation quality separately. Use human-reviewed samples and track results by language and file type.
- Define data classification, allowed document types, user notices, Azure processing region, retention, deletion, and incident procedures before production use.
- Keep credentials out of code and images. Restrict storage access to short-lived per-job permissions and verify deletion.
- Ensure model/service changes and configuration changes can be traced, reviewed, and rolled back.

## Responsible AI considerations

Machine translation can alter meaning, formatting, names, numbers, or legal/technical terminology. Do not treat output as authoritative without human review where correctness has material consequences. Language detection maps every non-English language to English, which can produce an unexpected target. Filenames with non-ASCII characters may be sent to an external translation service separately from document translation.

The product should disclose translation limitations, processing services and locations, and the retention behavior. Provide a user path to inspect, correct, and report poor outputs. Avoid using the tool as the sole decision-maker in high-impact contexts.

## Monitoring plan

If the system is prepared for production, monitor at least:

- Upload volume, file type/size, rejection rate, and processing duration.
- Translation job completion, failure categories, retries, and service throttling.
- OCR page counts, processing duration, Tesseract failures, and measured OCR confidence.
- Download completion and blob/local cleanup success.
- Human quality-review outcomes by source/target language and document format.
- Authentication failures and authorization-denial events, without logging passwords, keys, SAS tokens, or document content.

Set explicit retention windows for operational metadata and restrict access to logs. Monitoring and alerting are not configured in the current repository.
