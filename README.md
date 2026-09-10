# MLOps CD Demo

A Flask-based ML prediction API with a complete **Continuous Delivery** pipeline using GitHub Actions, Docker, and GHCR.

## Project Structure

```
mlops-cd-demo/
├── app.py                  # Flask API (home, health, predict)
├── tests/
│   └── test_app.py         # Pytest test suite
├── Dockerfile              # Container image definition
├── compose.yaml            # Docker Compose for local dev
├── requirements.txt        # Python dependencies
├── pyproject.toml          # Pytest configuration
├── VERSION                 # Semantic version file
└── .github/
    └── workflows/
        └── cd.yml          # CD pipeline definition
```

## API Endpoints

| Method | Endpoint    | Description                     |
|--------|-------------|---------------------------------|
| GET    | `/`         | Service info                    |
| GET    | `/health`   | Health check with model version |
| POST   | `/predict`  | ML prediction (dummy: value×2)  |

### Example Usage

```bash
# Health check
curl http://localhost:5000/health

# Prediction
curl -X POST http://localhost:5000/predict \
  -H "Content-Type: application/json" \
  -d '{"value": 5}'
# Response: {"input": 5.0, "prediction": 10.0, "model_version": "1.0"}
```

## Local Development

### Run with Python

```bash
python -m venv .venv
source .venv/Scripts/activate   # Windows (Git Bash)
pip install -r requirements.txt
python app.py
```

### Run with Docker

```bash
docker build -t mlops-cd-demo:local .
docker run --rm -p 5000:5000 mlops-cd-demo:local
```

### Run with Docker Compose

```bash
docker compose up --build
```

### Run Tests

```bash
pytest -v
```

## CD Pipeline

The GitHub Actions workflow (`.github/workflows/cd.yml`) triggers on version tags (`v*.*.*`) and runs:

```
git tag v1.0.0 → push tag
        │
        ▼
   ┌─────────┐
   │  test    │  Run pytest
   └────┬─────┘
        │
        ▼
   ┌─────────┐
   │  build   │  Build & push Docker image to GHCR
   └────┬─────┘
        │
        ▼
   ┌──────────────┐
   │deploy-staging │  SSH deploy + smoke test (curl /health)
   └────┬──────────┘
        │
        ▼
   ┌───────────────────┐
   │deploy-production   │  Manual approval gate → SSH deploy
   └───────────────────┘
```

### Releasing a New Version

```bash
git add .
git commit -m "Your changes"
git push
git tag v1.0.1
git push origin v1.0.1
```

### Required GitHub Secrets

| Secret             | Description                    |
|--------------------|--------------------------------|
| `STAGING_HOST`     | Staging server IP address      |
| `STAGING_USER`     | Staging SSH username           |
| `STAGING_SSH_KEY`  | Staging SSH private key        |
| `PRODUCTION_HOST`  | Production server IP address   |
| `PRODUCTION_USER`  | Production SSH username        |
| `PRODUCTION_SSH_KEY`| Production SSH private key    |

> **Note:** `GITHUB_TOKEN` is provided automatically by GitHub Actions for GHCR authentication.

### GitHub Environments

- **staging** — auto-deploys after build
- **production** — requires manual approval (configure under *Settings → Environments → production → Required reviewers*)

## Tech Stack

- **Python 3.12** / Flask 3.1.2
- **Docker** / GHCR (GitHub Container Registry)
- **GitHub Actions** for CI/CD
- **pytest** for testing
