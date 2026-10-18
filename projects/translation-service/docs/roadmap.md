# Roadmap

This roadmap separates currently visible behavior from engineering work that would be needed for a reliable production product. Priorities are indicative, not committed delivery dates.

## Near term — make the prototype runnable and safe to evaluate

- Correct the Docker entry point to `Alpin_app.py` and configure Linux Tesseract with required language packs.
- Declare all imported runtime dependencies in `requirements.txt`.
- Remove local credential and secret files from build contexts; add secret-safe configuration examples.
- Replace browser-side authentication persistence with a supported server-validated session or identity provider.
- Use unique per-job blob paths to prevent name collisions and verify cleanup.
- Add automated tests for file validation, language direction, upload/download orchestration, OCR behavior, and cleanup.

## Medium term — reliability and operational readiness

- Add durable translation-job tracking, timeout handling, idempotency, retries, and user-safe error states.
- Add content-based file validation, malware scanning, and parser/resource limits.
- Replace file-size-ratio validation with format-aware integrity checks and a measured translation-quality review process.
- Add structured, secret-safe logs, metrics, health checks, and service dashboards.
- Define and test retention, deletion, backup, and incident-response policies.
- Add deployment infrastructure and a CI/CD pipeline only after the target Azure hosting model is selected.

## Longer term — product scope decisions

- Support user-selected source/target languages and validate language detection confidence.
- Evaluate glossary support and terminology workflows.
- Improve layout fidelity and accessibility across document formats.
- Determine whether a REST API, multi-tenant model, or commercial product workflows are required.

## Current product scope

The current codebase focuses on document translation and OCR. Product scope beyond these workflows has not been defined in the repository.
