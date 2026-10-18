# AIS

AIS is a sustainability assistant for building and infrastructure questions. The application combines a Next.js frontend, FastAPI backend, document extraction, a local searchable knowledge base, and configurable language-model and web-search integrations.

## High-Level Architecture

- **Frontend:** Next.js application.
- **Backend:** FastAPI REST and WebSocket APIs.
- **AI:** OpenAI or Anthropic model integration, selected through backend configuration.
- **Retrieval:** SQLite FTS5 keyword search over the local knowledge base; optional Tavily web research.
- **Documents:** Native text extraction and selective OCR using RapidOCR, with Tesseract available as a configured fallback.
- **Application data:** SQL Server stores accounts, chats, messages, and document metadata/extracted text.
- **Deployment:** Docker and Azure App Service sidecar deployment procedures are documented in this repository; this documentation does not confirm live deployment status.

## Documentation

- [System Architecture](docs/architecture/system-architecture.md)
- [Data Flow](docs/architecture/data-flow.md)
- [AI Architecture](docs/architecture/ai-architecture.md)
- [Security Overview](docs/security/security-overview.md)
- [Threat Model](docs/security/threat-model.md)
- [Data Privacy](docs/security/data-privacy.md)
- [Agent Capabilities](docs/ai-governance/agent-capabilities.md)
- [AI Readiness](docs/ai-governance/ai-readiness.md)

