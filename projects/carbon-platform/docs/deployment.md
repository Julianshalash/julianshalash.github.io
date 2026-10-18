Purpose: Describe the deployment and runtime setup that is verifiable from repository files.

Status: In active development. Scope 3 and commercial features are not yet implemented.

## Docker and Compose

📋 **Not currently available.** No `Dockerfile`, `docker-compose.yml`, Compose manifest or container entrypoint is tracked. There is no repository-supported Docker command. Add container packaging only after externalizing connection settings and documenting health checks, persistent report storage and secret injection.

## Environment configuration

Names referenced in the application include:

| Variable names | Use observed |
|---|---|
| `JWT_SECRET`, `JWT_ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `PASSWORD_RESET_EXPIRE_HOURS` | JWT and reset-token behavior |
| `OPENAI_API_KEY` | Assistant and report LLM calls |
| `REPORT_ENCRYPTION_KEY`, `REPORT_TOKEN_SECRET` | Encrypt report artifacts and sign download tokens |
| `AZURE_STORAGE_CONNECTION_STRING`, `AZURE_STORAGE_CONTAINER_NAME` | Azure Blob template downloads |
| `SMTP_PASSWORD`, `FRONTEND_RESET_PASSWORD_URL` | Email and reset flow configuration |
| `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_DB`, `REDIS_SSL`, `REDIS_SOCKET_CONNECT_TIMEOUT`, `REDIS_SOCKET_TIMEOUT`, `REDIS_RETRY_ON_TIMEOUT` | Optional Redis cache settings |

No values are documented here. SQL Server and MongoDB connection configuration is not consistently environment-based in current source. **TODO: confirm** production environment variable contract and required/optional status.

## Local development

Frontend package scripts include `dev`, `build`, `start`, and `lint`. Example:

```powershell
cd Frontend
npm install
npm run dev
```

The backend FastAPI entry point is `Backend/app/main.py`, but the repository does not include a backend dependency manifest or a documented service initialization procedure. **TODO: confirm** Python version, dependencies, SQL schema setup, Mongo database setup and API launch command.

## Intended Azure deployment

The intended hosting target is **Microsoft Azure for the complete application stack**:

| Workload | Intended target | Status |
|---|---|---|
| Next.js frontend | Hosted on Azure | 📋 Planned |
| FastAPI backend | Hosted on Azure | 📋 Planned |
| SQL Server relational database | Azure-hosted SQL Server service | 📋 Planned |
| MongoDB document database | Azure-hosted MongoDB-compatible service | 📋 Planned |
| Template files | Azure Blob Storage | ✅ Application integration implemented; account/container provisioning and production configuration are not present here |

The repository does not currently define or deploy this topology. The specific Azure products are deliberately left open rather than guessed:

- Frontend and backend compute: **TODO: confirm** Azure App Service, Azure Container Apps, or another Azure hosting option; choose based on runtime, background report jobs and operational needs.
- SQL Server: **TODO: confirm** Azure SQL Database versus Azure SQL Managed Instance and required compatibility with the existing SQL queries/schema.
- MongoDB: **TODO: confirm** the selected Azure-hosted option and compatibility with the PyMongo operations in use (for example, an Azure Cosmos DB API for MongoDB offering versus a MongoDB service hosted on Azure).
- Network topology, environments, regions, identity, private access and resiliency: **TODO: confirm**.

Until these choices and infrastructure are implemented, describe Azure as the intended target—not the current deployment. Azure Blob Storage is the only Azure service integration directly evidenced by application code.

## Azure security and operations (planned)

📋 Plan to use managed identities where supported, Azure Key Vault or equivalent secret management, private networking for database access, HTTPS/TLS, managed encryption at rest, restricted storage access, centralized logs/metrics, backup and restore policies, and separate deployment environments. These are recommendations, not implemented controls. **TODO: confirm** the chosen services support the required features and define retention, recovery objectives and data residency.

## CI/CD

No GitHub Actions workflows or other CI/CD configuration was found in tracked files. 📋 An automated build/test/deployment pipeline targeting Azure is planned or **TODO: confirm** if hosted elsewhere.

## Operational checklist before production

- Externalize and rotate all secrets; remove in-source database settings.
- Configure HTTPS, database TLS, CORS origins, network access and managed identity.
- Define persistent shared report storage and a durable background-job strategy.
- Add startup/readiness checks, structured monitoring, backup/restore and retention controls.
- Select Azure compute and database products, then add infrastructure/deployment definitions and automated security/build/test checks.
