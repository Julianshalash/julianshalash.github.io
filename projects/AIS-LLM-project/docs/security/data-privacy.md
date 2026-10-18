# AIS Data Privacy

## 1. Purpose

This document describes privacy-relevant data handling identified within the AIS application and its supporting services.

It documents:

- What categories of data AIS processes
- Why the data is processed
- Where the data is stored
- Which external services may receive data
- What information may be logged
- How long data is retained where this is known
- How data is deleted
- How access to data is controlled
- Privacy risks and areas requiring confirmation

This document reflects behavior identified from the AIS implementation and available configuration.

It is not a privacy notice, Data Protection Impact Assessment (DPIA), or statement of legal or regulatory compliance.

Where behavior cannot be verified from the implementation, it is explicitly marked:

**To be confirmed by the AIS team.**

---

## 2. Data Categories

AIS may process the following categories of information.

| Category | Examples | Purpose |
|---|---|---|
| Account data | User identifier, email address, authentication-related identifiers | Account management, authentication and access control |
| Chat data | User questions, AIS responses, conversation history, chat metadata and project-related information | Provide conversational functionality and maintain chat continuity |
| Uploaded-document data | Filename, extension, MIME type, processing status, extracted text and OCR output | Document analysis and question answering |
| Original uploaded files | File bytes received during document processing | Enable extraction, OCR and analysis |
| AI context | User request, selected chat history, document content, retrieved evidence, tool results and relevant instructions | Construct context required for model inference |
| AI-generated output | Generated answers, citations and other model-generated content | Provide responses to users |
| Web-search data | Search queries and retrieved result passages | Retrieve external evidence when web search is used |
| Knowledge-base data | Indexed source text, framework information, version information and source metadata | Provide internal sustainability knowledge and evidence |
| Technical and operational data | User identifiers, document/chat IDs, filenames, processing status, error information, token usage and operational metadata | Application operation, troubleshooting and monitoring |
| Security and authentication events | Authentication status, failed operations and access-related events | Application security and operational investigation |
| Feedback / analytics / embeddings | No corresponding application store or integration was identified in the reviewed implementation | Not currently established |

---

## 3. Data Classification

No formal AIS data-classification policy was identified in the reviewed repository.

**Classification to be confirmed by the AIS team.**

Until an approved classification policy is established, the following should be treated as potentially confidential project information:

- User prompts
- Conversation history
- Uploaded documents
- Extracted document text
- OCR output
- Project information
- AI context containing project information
- Retrieved internal knowledge
- Generated responses containing information derived from confidential inputs

The AIS team and the appropriate privacy/cybersecurity stakeholders should establish which organizational data classifications are permitted to be submitted to AIS.

---

## 4. Personal and Sensitive Data Handling

AIS accepts user-generated content and uploaded documents. As a result, users may submit content containing personal, confidential or sensitive business information.

Potential examples include:

- Names and contact information
- Employee or client information
- Project information
- Commercial information
- Sustainability and environmental project data
- Internal company information
- Confidential documents
- Credentials or secrets accidentally included in uploaded content

The following controls and policies require confirmation:

- Whether Personal Identifiable Information (PII) detection is implemented
- Whether sensitive-data detection is implemented
- Whether sensitive information is redacted before being sent to external AI services
- Whether secrets or credentials are detected in uploaded documents or prompts
- Whether sensitive information is automatically removed from logs
- Which data classifications users are allowed to upload
- Which information users must not submit to AIS
- Whether additional controls apply to customer or project-confidential information

**Status: To be confirmed by the AIS team and cybersecurity/privacy stakeholders.**

---

## 5. Purpose of Processing

AIS processes data for the following identified purposes.

### Account and authentication data

Used to:

- Authenticate users
- Associate application activity with authenticated users
- Enforce access controls
- Support account-related operations

### Chat data

Used to:

- Process user questions
- Maintain conversation context
- Present conversation history
- Support chat management

### Uploaded documents and extracted content

Used to:

- Extract document text
- Perform OCR when required
- Analyze document content
- Build context for user questions
- Provide document-based answers

### Knowledge-base content

Used to retrieve internal sustainability references and relevant supporting evidence.

### Web-search queries and results

Used to retrieve external evidence when web search functionality is selected or triggered.

### AI prompts and results

Used to provide contextual information necessary for the configured model to generate AIS responses.

### Operational and security data

Used for:

- Application operation
- Troubleshooting
- Error investigation
- Document-processing monitoring
- Authentication monitoring
- Security investigation where applicable

---

## 6. Storage Locations

| Location | Data held | Privacy considerations |
|---|---|---|
| SQL Server | Accounts, token records, chats, messages, document metadata and extracted content | Message/document content encryption depends on `ENCRYPTION_KEY`. Application behavior where this key is absent requires security review. |
| Local SQLite FTS5 | Knowledge-base chunks and source/framework/version metadata | Runtime location is configurable. Production persistence, filesystem protection and access controls must be confirmed. |
| Backend process memory | Current upload bytes during extraction, AI/tool context and temporary search information | Memory is process-local. Exposure through crash dumps, debugging or process access should be considered by the security review. |
| Application logs | Operational, authentication, document-processing and error information | Production destination, access restrictions, redaction and retention require confirmation. |
| External LLM provider | Prompt/context required for model inference and generated responses | Provider, region, retention, training settings and contractual processing terms require confirmation. |
| Tavily | Search queries and search-related data when web search is used | Provider-side retention, processing region and data-processing conditions require confirmation. |

No Azure Blob Storage, vector database or separate application analytics store was identified in the reviewed application implementation.

If those services are introduced later, this document must be updated before or as part of the change.

---

## 7. External Data Recipients

AIS integrates with external services that may process data.

| Recipient | Data sent | Purpose |
|---|---|---|
| Configured LLM provider (OpenAI or Anthropic) | User request, system/task instructions, selected conversation/document context, retrieved evidence and tool results required for the request | Generate AIS responses |
| Tavily | Search query generated from the user request | Retrieve web-search evidence |
| Microsoft Entra ID | Authentication protocol and identity-related information when OIDC authentication is enabled | Authenticate users |
| Azure-hosted infrastructure | Application and operational data associated with deployed AIS services | Host and operate AIS, where applicable |

Uploaded documents and page images are processed using backend OCR/parser libraries according to the reviewed implementation. No separate external OCR API was identified.

For each external provider, the following information should be maintained and periodically reviewed:

- Data categories transferred
- Purpose of the transfer
- Whether the transfer is mandatory or conditional
- Data-processing region
- Provider retention policy applicable to AIS
- Model-training/data-reuse configuration where applicable
- Applicable contractual/data-processing agreement
- Security controls
- Service owner

Where these details cannot be determined from the repository, they are **To be confirmed by the AIS team and relevant privacy/cybersecurity stakeholders.**

---

## 8. Data Sent to the LLM

Depending on the user request and AIS workflow, model requests may contain:

- User question
- System and task instructions
- Relevant conversation history
- Extracted document content
- OCR-generated text
- Retrieved knowledge-base passages
- Web-search evidence
- Tool results
- Relevant metadata required to process the request

AIS should provide only the context required to complete the requested task.

The following require confirmation:

- Whether sensitive-data filtering occurs before model submission
- Whether PII redaction occurs before model submission
- Maximum document/context content sent per request
- Provider-side request/response retention
- Provider-side model-training/data-reuse settings
- Processing location/data residency
- Whether provider-side abuse/security monitoring retains submitted content

**To be confirmed by the AIS team and relevant privacy/cybersecurity stakeholders.**

---

## 9. User Data Logged

Repository logging includes some or all of the following:

- User identifiers associated with WebSocket, upload, chat and authentication events
- Filenames and client-declared MIME types during uploads
- Document and chat identifiers
- File byte counts
- Extraction methods
- Extracted character counts
- Processing status
- Failure and exception information
- Evidence passage counts/types
- Token usage
- Model stop reason
- Tool failures
- Authentication events

### WebSocket Payload Logging Risk

The reviewed implementation may log a prefix of received WebSocket payload text, up to 200 characters, at debug level.

Depending on the message, this may expose:

- User prompts
- Personal information
- Confidential project information
- Sensitive business information
- Other user-supplied content

The following must therefore be confirmed before production approval:

- Whether debug logging is enabled in production
- Whether prompt/message content needs to be logged
- Whether metadata-only logging can be used instead
- Whether sensitive fields are redacted
- Where these logs are stored
- Who can access them
- How long the logs are retained
- How logs are securely deleted

A comprehensive prompt/response logging repository was not identified in the reviewed application path.

---

## 10. Data Retention

The reviewed application currently identifies the following retention behavior:

- `CHAT_HISTORY_DAYS` defaults to 90 days.
- A daily cleanup process permanently removes soft-deleted chats after seven days.
- The cleanup process also removes non-deleted chats whose last update/create timestamp is older than the configured chat-history threshold.
- Associated message and document records are deleted as part of this process.
- Original upload retention is not established because the current upload implementation does not persist raw upload bytes.

The following retention areas require confirmation:

- Knowledge-base source retention
- SQLite knowledge-index retention
- SQL Server backup retention
- Application-log retention
- Security-log retention
- External LLM-provider retention
- Tavily retention
- Authentication-related retention
- Account data retention

Retention requirements should ultimately be based on approved organizational, legal, contractual and operational requirements rather than only application defaults.

---

## 11. Data Deletion

### Chat deletion

Users can soft-delete chats through the available chat APIs.

The scheduled cleanup process subsequently permanently removes associated chats, messages and document records according to the configured retention behavior.

### Document deletion

No separate user-facing document-only deletion route was identified in the reviewed document router.

Whether document-level deletion must be provided independently from chat deletion is **To be confirmed by the AIS team**.

### Other deletion workflows

The following require confirmation:

- User-account deletion
- Administrative deletion
- Knowledge-base source removal
- Knowledge-index removal
- Application-log deletion
- Security-log deletion
- External-provider deletion
- Backup expiration
- Restoration of previously deleted information from backups
- Evidence/proof of deletion where required

No embedding deletion workflow is currently applicable because no embedding/vector store was identified in the reviewed implementation.

---

## 12. Access Control and Data Isolation

Protected endpoints use authenticated user identity.

Reviewed chat and document retrieval/mutation paths apply ownership or chat-membership checks.

The privacy and security review should verify access control across:

- User accounts
- Chats
- Messages
- Uploaded documents
- Extracted document content
- Project data
- Knowledge-base content
- Administrative functionality
- Application logs

### User and Tenant Isolation

The following scenarios require explicit verification:

- User A cannot access User B's chats.
- User A cannot retrieve User B's documents.
- Guessing or modifying a chat/document identifier cannot bypass authorization.
- Organization A cannot retrieve Organization B's content where multi-tenant separation is required.
- Knowledge-base retrieval respects the intended visibility model.
- Administrative access is restricted to authorized roles.
- Temporary processing data cannot be accessed by another user.
- AI context does not unintentionally contain another user's information.

Organization/tenant separation requirements, administrative role assignment and knowledge-base visibility are **To be confirmed by the AIS team and cybersecurity team.**

---

## 13. Data Residency and Processing Locations

The repository alone does not establish all geographic locations where AIS data may be stored or processed.

The AIS team should document the production processing/storage region for:

- Application hosting
- SQL Server
- Application logs
- Backups
- LLM provider requests
- Tavily requests
- Microsoft Entra ID authentication data
- Knowledge-base storage

Cross-region or cross-border data transfers should be identified where applicable.

**Production data residency and applicable processing requirements are To be confirmed by the AIS team and privacy/cybersecurity stakeholders.**

---

## 14. Backup and Recovery Privacy Considerations

Database and infrastructure backup behavior is not fully established from the reviewed application code.

The production review should establish:

- Which AIS data stores are backed up
- Backup storage location
- Backup encryption
- Backup retention
- Access to backups
- Restore permissions
- Restore auditing
- Whether deleted user information remains in backups
- When deleted information expires from backups
- Whether restored backups can reintroduce previously deleted records

**To be confirmed by the AIS team and infrastructure/cybersecurity teams.**

---

## 15. User Privacy Rights and Data Requests

The repository does not establish the complete organizational process for privacy-related user requests.

The responsible business/privacy stakeholders should determine applicable processes covering:

- Access to personal data
- Correction of personal data
- Deletion requests
- Account deletion
- Conversation deletion
- Uploaded-document deletion
- Export of user information
- Questions concerning how information is processed

The exact rights and obligations depend on applicable organizational, contractual and legal requirements.

**Process ownership and implementation are To be confirmed.**

---

## 16. Privacy Incident Handling

Privacy-related incidents may include scenarios such as:

- Unauthorized access to another user's chat
- Unauthorized access to uploaded documents
- Sensitive information appearing in application logs
- Sensitive information being unintentionally returned in an AI response
- Incorrect access controls exposing project information
- Accidental transmission of information to an unintended external provider
- Unexpected retention of deleted information

Privacy incidents should be handled through the organization's approved cybersecurity/privacy incident-management process.

The following require confirmation:

- Incident owner
- Escalation process
- Investigation responsibilities
- Evidence preservation requirements
- Notification responsibilities
- Remediation tracking
- Post-incident review

---

## 17. Privacy Control Ownership

Privacy responsibilities should be formally assigned.

Suggested ownership areas requiring organizational confirmation include:

| Area | Owner | Reviewer | Status |
|---|---|---|---|
| User authentication data | To be confirmed | Cybersecurity | Open |
| Chat/document privacy | To be confirmed | Privacy/Cybersecurity | Open |
| LLM data processing | To be confirmed | Privacy/Cybersecurity | Open |
| Web-search data processing | To be confirmed | Privacy/Cybersecurity | Open |
| Database protection | To be confirmed | Cybersecurity | Open |
| Logging and redaction | To be confirmed | Cybersecurity | Open |
| Retention/deletion | To be confirmed | Privacy | Open |
| Backup privacy | To be confirmed | Cybersecurity/Infrastructure | Open |
| Privacy incidents | To be confirmed | Privacy/Cybersecurity | Open |

This table represents required ownership areas and does not assign organizational responsibility.

---

## 18. Privacy Risks and Open Questions

The following items remain open or require formal validation:

1. Establish the AIS data-classification policy and identify which information users are permitted to submit.

2. Confirm whether personal or sensitive information requires detection, redaction or prevention controls.

3. Confirm approved LLM and search providers.

4. Confirm LLM-provider:
   - Processing region
   - Data retention
   - Model-training/data-reuse configuration
   - Applicable contractual data-processing terms

5. Confirm Tavily:
   - Processing region
   - Data retention
   - Applicable contractual data-processing terms

6. Review production use of `ENCRYPTION_KEY` and confirm that sensitive application data cannot unintentionally fall back to plaintext storage.

7. Confirm SQL Server:
   - Transport encryption
   - Storage encryption
   - Backup encryption
   - Backup retention

8. Confirm production log:
   - Destination
   - Access restrictions
   - Retention
   - Redaction
   - Deletion

9. Review WebSocket debug logging because payload prefixes may contain user or confidential information.

10. Confirm whether original uploaded files should be retained. Current implementation does not establish durable raw-file storage.

11. Confirm organization and tenant-isolation requirements.

12. Confirm knowledge-base visibility and whether knowledge content is global, role-restricted or organization-specific.

13. Confirm administrative role assignment and access policy.

14. Confirm data-residency requirements across Azure, configured LLM providers, web-search services and authentication providers.

15. Establish privacy-related user request and deletion processes.

16. Confirm deletion propagation to backups and external providers where applicable.

17. Establish privacy incident ownership and escalation processes.

---

## 19. Privacy Evidence Register

Privacy controls should not be considered verified solely because they are documented.

Evidence should be recorded for each material requirement.

| ID | Requirement | Evidence Required | Owner | Status |
|---|---|---|---|---|
| PRIV-01 | Approved data classification established | Approved classification policy | To be confirmed | Open |
| PRIV-02 | User/chat access isolation verified | Access-control test evidence | To be confirmed | Open |
| PRIV-03 | Document isolation verified | Authorization test evidence | To be confirmed | Open |
| PRIV-04 | Production encryption configuration verified | Configuration/security review | To be confirmed | Open |
| PRIV-05 | LLM data-processing settings approved | Provider/configuration evidence | To be confirmed | Open |
| PRIV-06 | Search-provider processing approved | Provider/configuration evidence | To be confirmed | Open |
| PRIV-07 | Production log content reviewed | Logging/configuration review | To be confirmed | Open |
| PRIV-08 | Log retention approved | Retention configuration/policy | To be confirmed | Open |
| PRIV-09 | Chat/document deletion verified | Functional test evidence | To be confirmed | Open |
| PRIV-10 | Backup/deletion behavior documented | Infrastructure evidence | To be confirmed | Open |
| PRIV-11 | Data residency documented | Deployment/provider evidence | To be confirmed | Open |
| PRIV-12 | Administrative access reviewed | Role/access evidence | To be confirmed | Open |

Additional requirements should be added as the implementation and privacy review progress.