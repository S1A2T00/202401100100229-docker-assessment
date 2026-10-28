# FestPass – Docker & Container Security (CA-2)

A containerized event registration application built using Flask, PostgreSQL, and Nginx. This project demonstrates Docker image creation, multi-container orchestration, network isolation, persistent storage, vulnerability scanning, image versioning, and container image signing.

## 1. Student Details

- **Name:** Satyam Pandey
- **Registration Number (REGNO):** 202401100100229
- **Application Port:** 8029
- **Docker Hub Username:** `s9a5`
- **Docker Hub Repository:** [s9a5/fest-backend](https://hub.docker.com/r/s9a5/fest-backend)

## 2. Project Objectives

The objectives of this practical assessment are to:

- Dockerize a Flask backend application.
- Deploy a three-tier application using Docker Compose.
- Configure separate networks for application communication and database isolation.
- Persist PostgreSQL data using a named Docker volume.
- Scan container images for vulnerabilities using Trivy.
- Improve image security and compare vulnerability reports.
- Publish versioned images to Docker Hub.
- Sign and verify container images using Cosign.

## 3. Technology Stack

| Technology | Purpose |
|---|---|
| Docker | Containerization and image creation |
| Docker Compose | Multi-container orchestration |
| Python and Flask | Backend REST API |
| PostgreSQL | Persistent application database |
| Nginx | Frontend web server and reverse proxy |
| Trivy | Container vulnerability scanning |
| Docker Hub | Container image registry |
| Cosign | Container image signing and verification |
| Git and GitHub | Source control and project submission |

## 4. Application Architecture

The application consists of three services:

1. **Frontend:** Nginx serves the FestPass web interface and forwards API requests to the backend.
2. **Backend:** Flask provides REST API endpoints for health checks and event registrations.
3. **Database:** PostgreSQL stores registration records persistently.

### Network design

- `frontend-backend`: Connects the frontend and backend.
- `backend-db`: Connects the backend and database.

The frontend is not directly connected to the database network.

### Persistent storage

A named Docker volume, `postgres-data`, stores PostgreSQL data so registrations survive container recreation when the volume is retained.

## 5. Project Structure

```text
202401100100229-docker-assessment/
├── README.md
├── compose.yaml
├── .env.example
├── .gitignore
├── cosign.pub
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .dockerignore
├── frontend/
│   ├── index.html
│   └── nginx.conf
├── reports/
│   ├── trivy-before.txt
│   └── trivy-after.txt
└── screenshots/
```

## 6. Application API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/health` | Checks backend and database connectivity |
| GET | `/api/registrations` | Retrieves registered participants |
| POST | `/api/registrations` | Creates a new event registration |

The health endpoint reports the registration number, application version, container hostname, and database connectivity status.

## 7. Setup and Execution

### Prerequisites

Install and configure:

- Docker Desktop
- Git
- Trivy
- Cosign

Ensure Docker Desktop is running before executing Docker commands.

### Configure the environment

Create a local `.env` file from `.env.example` and set a strong PostgreSQL password.

For PowerShell, open the project directory and run:

```powershell
$REGNO = "202401100100229"
$PORT = 8029
$env:REGNO = $REGNO
$env:PORT = "$PORT"
```

Keep `.env` and private keys out of version control.

### Validate and start the application

```powershell
docker compose config
docker compose up -d --build
docker compose ps
```

Open the application in a browser:

http://localhost:8029

### Inspect logs

```powershell
docker compose logs
docker compose logs backend
docker compose logs db
```

### Test the backend

```powershell
curl.exe http://localhost:8029/api/health
curl.exe http://localhost:8029/api/registrations
```

### Stop and restart the application

```powershell
docker compose down
docker compose up -d
```

The named PostgreSQL volume should preserve registrations across this restart. Do not use `docker compose down -v` when you need to retain the database data.

## 8. Docker Image Versioning

The backend images used in this assessment are:

| Image | Description |
|---|---|
| `fest-backend:0.1` | Initial backend image based on Python 3.9 |
| `fest-backend:0.2` | Improved backend image based on Python 3.12 slim |
| `s9a5/fest-backend:1.0.0` | Versioned image published to Docker Hub |
| `s9a5/fest-backend:1.0` | Additional release tag |
| `s9a5/fest-backend:1.0.1` | Unsigned version used to demonstrate verification failure |

### Build the initial image

```powershell
docker build -t fest-backend:0.1 ./backend
```

### Build the improved image

```powershell
docker build -t fest-backend:0.2 ./backend
```

### Build and tag release 1.0.0

The release image must include the intended application version:

```powershell
docker build --build-arg APP_VERSION=1.0.0 -t fest-backend:1.0.0 ./backend

docker tag fest-backend:1.0.0 s9a5/fest-backend:1.0.0
docker tag fest-backend:1.0.0 s9a5/fest-backend:1.0
```

### Publish to Docker Hub

After logging in with the Docker Hub account:

```powershell
docker login
docker push s9a5/fest-backend:1.0.0
docker push s9a5/fest-backend:1.0
```

Docker Hub repository: [https://hub.docker.com/r/s9a5/fest-backend](https://hub.docker.com/r/s9a5/fest-backend)

## 9. Trivy Vulnerability Scanning

Trivy is used to identify vulnerabilities in the initial and improved backend images.

### Scan the initial image

```powershell
trivy image --format table --output reports/trivy-before.txt fest-backend:0.1
```

### Scan the improved image

```powershell
trivy image --format table --output reports/trivy-after.txt fest-backend:0.2
```

### Enforce the critical vulnerability gate

```powershell
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
```

The gate fails when fixable CRITICAL vulnerabilities matching the selected scan configuration remain. Actual results depend on the image contents and the vulnerability database at scan time.

### Before-and-after comparison

Replace the placeholders below with the actual Trivy reports and image sizes from the assessment environment.

| Image | CRITICAL | HIGH | Size |
|---|---:|---:|---:|
| `fest-backend:0.1` (Python 3.9) | Pending scan | Pending scan | Pending measurement |
| `fest-backend:0.2` (Python 3.12 slim) | Pending scan | Pending scan | Pending measurement |

The comparison should document whether vulnerabilities decreased after remediation. Do not assume that the improved image has zero vulnerabilities without verifying the scan output.

## 10. Cosign Image Signing and Verification

Cosign is used to sign the published release and verify its authenticity.

### Generate a key pair

```powershell
cosign generate-key-pair
```

Keep `cosign.key` private. Only `cosign.pub` may be committed to the repository.

### Sign the published release

Use the actual immutable digest returned by Docker Hub or the image push operation.

```powershell
cosign sign --key cosign.key s9a5/fest-backend@sha256:REPLACE_WITH_ACTUAL_DIGEST
```

Replace the placeholder with the complete real digest.

### Verify the signature

```powershell
cosign verify --key cosign.pub s9a5/fest-backend@sha256:REPLACE_WITH_ACTUAL_DIGEST
```

Verification must succeed for the signed release.

### Test the unsigned release

Build and publish `1.0.1` without signing it, then verify it using the same public key:

```powershell
cosign verify --key cosign.pub s9a5/fest-backend:1.0.1
```

Expected result: verification fails if the image has no valid matching signature.

## 11. Release Digest

Record the actual digest associated with the published `1.0.0` release.

**Published image:** `s9a5/fest-backend:1.0.0`

**Digest:** `sha256:REPLACE_WITH_ACTUAL_PUSH_DIGEST`

Do not substitute the digest of another tag or image build. Use the digest of the exact release that was signed and verified.

## 12. AI Tools Used

**AI tool:** ChatGPT / GitHub Copilot, as applicable to the work actually performed.

AI tools assisted with project scaffolding, Docker configuration, command explanations, documentation, and troubleshooting. Commands and application behavior were tested by the student where indicated by the recorded execution evidence.

Update this section to accurately reflect the tools used and the work personally completed.

## 13. Assessment Evidence

Save genuine screenshots of the actual terminal, browser, Docker Hub, Trivy, and Cosign outputs in the `screenshots/` directory.

Suggested evidence:

- `S1-2.png`: Initial Docker image build and image listing.
- `S1-3.png`: Backend container, health endpoint, and logs.
- `S2-2.png`: Docker network isolation verification.
- `S2-3.png`: Running application and registration functionality.
- `S2-4.png`: PostgreSQL persistence after restart.
- `S3-1.png`: Initial Trivy scan.
- `S3-3.png`: Improved image scan and vulnerability comparison.
- `S4-1.png`: Docker Hub image push and digest.
- `S4-2.png`: Docker Hub version tags.
- `S4-3.png`: Application running with release `1.0.0`.
- `S5-1.png`: Cosign key-generation evidence, without exposing the private key.
- `S5-2.png`: Image signing.
- `S5-3.png`: Successful signature verification.
- `S5-4.png`: Failed verification of the unsigned release.

Do not include passwords, access tokens, private signing keys, or other confidential information in screenshots.

## 14. Steps Not Completed

Update this section before submission.

- Vulnerability counts and image sizes: Pending actual Trivy scans and image measurements.
- Published release digest: Pending confirmation from the actual image push output.
- Signing and verification: Mark complete only after the signed release verifies successfully and the unsigned release fails verification.
- Screenshots: Pending genuine evidence capture, where not yet completed.
- Other incomplete tasks: Record any remaining blockers here.

If every assessment requirement has been completed and verified, replace the pending items with the actual results and state that no required steps remain incomplete.

## 15. Final Submission Checklist

- [ ] Backend Docker image builds and runs.
- [ ] All required API endpoints work.
- [ ] Docker Compose starts all three services.
- [ ] Frontend cannot directly access the database network.
- [ ] PostgreSQL registrations persist across container restarts.
- [ ] Initial and improved Trivy reports are saved.
- [ ] Vulnerability comparison contains actual scan results.
- [ ] Docker Hub release tags are published.
- [ ] Actual `1.0.0` digest is recorded.
- [ ] Release signature verifies successfully.
- [ ] Unsigned release verification fails.
- [ ] No secrets or private keys are committed.
- [ ] Genuine screenshots are included.
- [ ] README accurately describes completed and incomplete work.
- [ ] Final GitHub repository URL is ready for submission.
