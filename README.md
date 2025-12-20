# Docker Build, Push and Deploy Action

A GitHub composite action that builds Docker images, pushes them to GitHub Container Registry (GHCR), and optionally triggers Render deployments.

## Features

- Build Docker images with custom platforms
- Push to GitHub Container Registry
- Support for multiple tags
- Optional Render deployment integration
- Configurable deployment branches

## Usage

### Basic Usage

```yaml
- name: Deploy Docker Image
  uses: zakloum/deploy-action@v1
  with:
    image-name: ghcr.io/${{ github.repository }}
    ghcr-username: ${{ secrets.GHCR_USERNAME }}
    ghcr-token: ${{ secrets.GHCR_TOKEN }}
```

### With Render Deployment

```yaml
- name: Deploy Docker Image
  uses: zakloum/deploy-action@v1
  with:
    image-name: ghcr.io/${{ github.repository }}
    ghcr-username: ${{ secrets.GHCR_USERNAME }}
    ghcr-token: ${{ secrets.GHCR_TOKEN }}
    enable-render-deployment: 'true'
    render-deploy-hook: ${{ secrets.RENDER_DEPLOY_HOOK }}
    deploy-on-branch: 'master'
```

### Complete Example

```yaml
name: Deploy Application

on:
  push:
    branches: [ master, develop ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build application
        run: mvn clean package -DskipTests

      - name: Deploy Docker Image
        uses: zakloum/deploy-action@v1
        with:
          image-name: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          ghcr-username: ${{ secrets.GHCR_USERNAME }}
          ghcr-token: ${{ secrets.GHCR_TOKEN }}
          dockerfile: 'Dockerfile.ci'
          platform: 'linux/amd64'
          enable-render-deployment: 'true'
          render-deploy-hook: ${{ secrets.RENDER_DEPLOY_HOOK }}
          deploy-on-branch: 'master'
          additional-tags: 'staging,v1.0.0'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `registry` | Container registry | No | `ghcr.io` |
| `image-name` | Docker image name | Yes | - |
| `ghcr-username` | GitHub Container Registry username | Yes | - |
| `ghcr-token` | GitHub Container Registry token | Yes | - |
| `dockerfile` | Path to Dockerfile | No | `Dockerfile.ci` |
| `docker-context` | Docker build context path | No | `.` |
| `platform` | Target platform for Docker build | No | `linux/amd64` |
| `enable-render-deployment` | Enable Render deployment trigger | No | `false` |
| `render-deploy-hook` | Render deployment hook URL | No | - |
| `deploy-on-branch` | Branch name to trigger deployment | No | `master` |
| `additional-tags` | Additional Docker tags (comma-separated) | No | - |

## Outputs

| Output | Description |
|--------|-------------|
| `image-url` | Full URL of the pushed Docker image |
| `deployment-status` | Status of Render deployment (success, failed, skipped) |

## Requirements

### Secrets

You need to set up the following secrets in your repository:

- `GHCR_USERNAME`: Your GitHub username
- `GHCR_TOKEN`: GitHub Personal Access Token with `write:packages` permission
- `RENDER_DEPLOY_HOOK`: (Optional) Render deployment hook URL

### Permissions

The workflow using this action needs the following permissions:

```yaml
permissions:
  contents: read
  packages: write
```

## How It Works

1. **Login**: Authenticates with GitHub Container Registry
2. **Metadata**: Extracts Docker image metadata and generates tags
3. **Build & Push**: Builds the Docker image and pushes it with multiple tags:
   - `latest` (always)
   - Branch name (if on deployment branch)
   - Additional custom tags (if specified)
4. **Deploy**: (Optional) Triggers Render deployment if enabled and on the specified branch

## License

MIT