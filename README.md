# Develop Deployment Workflow

This GitHub Actions workflow automates the deployment process by performing unit tests, code scanning, building and pushing Docker images, scanning images for vulnerabilities, and deploying to a Kubernetes cluster via ArgoCD.

## Workflow Inputs

The workflow supports configurable inputs to enable or disable specific features.

### General Inputs
| Name                    | Type    | Default     | Description |
|-------------------------|---------|-------------|-------------|
| `golang`                | boolean | `false`     | Enable Go language support |
| `go_version`            | string  | `1.23`      | Go version to use |
| `nodejs`                | boolean | `false`     | Enable Node.js support |
| `node_version`          | string  | `18.20.2`   | Node.js version to use |
| `unittest`              | boolean | `true`      | Enable unit testing |
| `image_scan`            | boolean | `true`      | Enable Docker image scanning |
| `image_scan_scanners`   | string  | `vuln,secret,misconfig` | Scanners to use for image scanning |
| `image_scan_severity`   | string  | `UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL` | Severity levels to scan for |
| `image_scan_vuln-type`  | string  | `library`   | Type of vulnerabilities to scan |
| `code_scan_scanners`    | string  | `vuln,secret,misconfig` | Scanners to use for code scanning |
| `code_scan_vuln-type`   | string  | `library`   | Type of vulnerabilities to scan |
| `code_scan_severity`    | string  | `UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL` | Severity levels to scan for |

### Secrets
| Name    | Required | Description |
|---------|----------|-------------|
| `cd_pat` | true    | Personal access token for GitOps repository push |

## Workflow Permissions

This workflow requires the following permissions:
- `id-token: write`
- `contents: write`
- `actions: read`
- `security-events: write`
- `issues: write`
- `checks: write`
- `packages: write`

## Jobs

### 1. Node.js Unit Tests (`NodeUnittest`)
Runs unit tests for a Node.js project if enabled.
- Checks out the repository
- Sets up the specified Node.js version
- Installs dependencies
- Runs tests with `npm run test`

### 2. Go Unit Tests (`GOUnitTest`)
Runs unit tests for a Go project if enabled.
- Checks out the repository
- Sets up the specified Go version
- Runs tests and checks test coverage

### 3. Code Scanning (`CodeScan`)
Scans the codebase using Trivy for vulnerabilities, secrets, and misconfigurations.
- Checks out the repository
- Runs Trivy scans with configurable severity levels
- Uploads scan results as an artifact

### 4. Build and Push Docker Image (`DockerBuild`)
Builds and pushes a Docker image to GitHub Container Registry (GHCR).
- Checks out the repository
- Logs in to GHCR
- Builds the Docker image
- Runs Trivy scans on the image
- Uploads scan results as an artifact
- Pushes the image to GHCR

### 5. Deploy to Kubernetes (`Deploy`)
Deploys the built image to a Kubernetes cluster using ArgoCD.
- Checks out the GitOps repository
- Updates the image tag in `values.yaml`
- Commits and pushes the changes

## Usage

To use this workflow, call it from another workflow:

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/develop-deployment.yml
    with:
      golang: true
      go_version: "1.23"
      nodejs: true
      node_version: "18.20.2"
      unittest: true
      image_scan: true
    secrets:
      cd_pat: ${{ secrets.CD_PAT }}
```

