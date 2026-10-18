# Deployment

## Verified deployment assets

The repository includes a Dockerfile based on `python:3.11-slim`, installs `Code/requirements.txt`, sets Streamlit to listen on `0.0.0.0:8501`, and exposes port `8501`. Azure Blob Storage and Azure AI Document Translation SDK clients are used by the application.

The repository does **not** include Azure hosting resources or deployment configuration: no Bicep/Terraform, compose file, Azure Container Apps/App Service/AKS manifests, registry configuration, CI/CD workflow, managed identity, Key Vault integration, or operational monitoring setup was found. Azure service usage in application code does not establish that the app is deployed on Azure.

## Current container blockers

The checked-in Dockerfile is not a working runtime recipe as written:

1. It starts `setec_app.py`, which is absent. The Streamlit entry point is `Alpin_app.py`.
2. The requirements omit imported packages `pytesseract` and `deep-translator`.
3. The OCR module hard-codes a Windows Tesseract executable path, while the image is Linux-based, and the image does not install Tesseract or OCR language data.
4. The application expects runtime `login.yaml` and Azure configuration; secrets and account data should not be copied into an image.

For these reasons, the commands below illustrate the intended container workflow only after the Dockerfile and dependencies are corrected; they are not a claim that the current image can start successfully.

```powershell
cd Code
docker build -t alpin-document-translation .
docker run --rm -p 8501:8501 `
  -e TRANSLATOR_ENDPOINT `
  -e TRANSLATOR_KEY `
  -e TRANSLATOR_REGION `
  -e STORAGE_ACCOUNT_URL `
  -e SAS_TOKEN `
  alpin-document-translation
```

The `-e NAME` form passes each variable from the current PowerShell environment; set those variables through a protected secret mechanism before running the command. Provide any required authentication configuration through a protected runtime mount. Do not put actual values in shell history, source control, image layers, or public documentation. The app reads a dotenv file named `Transaltion.env` relative to its working directory; environment variables can also be injected at runtime. Validate the actual configuration flow before deployment.

## Local development

From the repository root:

```powershell
cd Code
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip install pytesseract deep-translator
streamlit run Alpin_app.py
```

Set the five configuration variable names documented in the root README. The OCR path additionally requires an installed Tesseract executable and language data. The application reads `login.yaml` from the current working directory; provide an appropriately secured runtime file.

## Production deployment requirements

Before using the application with sensitive documents:

- Select and document an Azure hosting target and create repeatable infrastructure configuration.
- Prefer managed identity for service-to-service access where supported; otherwise use short-lived, least-privilege secrets in a secret manager.
- Restrict storage/network access, verify TLS and encryption settings, and configure blob lifecycle/retention.
- Fix authentication and ensure authorization isolates each user's files and jobs.
- Fix per-job blob naming, cleanup verification, logging, monitoring, resource limits, and health checks.
- Build and scan the image in CI; run as a non-root user and ensure image layers contain no credentials.
- Validate Tesseract and every supported file type in the target operating system.
