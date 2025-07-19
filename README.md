# Portage Action

A GitHub Action to mirror container images between registries. Copy container images from one registry to another with
support for batch operations.

## Features

- 🚀 Mirror single or multiple container images
- 📁 Support for JSON input or file-based configuration
- 📊 Detailed output with image digests, sizes, and status

## Usage

### Basic Example

```yaml
- name: Mirror container images
  uses: monkescience/portage-action@v0.2.0
  with:
    images: |
      [
        {
          "source": "docker.io/nginx:latest",
          "destination": "ghcr.io/myorg/nginx:latest",
          "architecture": "linux/amd64"
        }
      ]
```

### Multiple Images

```yaml
- name: Mirror multiple images
  uses: monkescience/portage-action@v0.2.0
  with:
    images: |
      [
        {
          "source": "docker.io/nginx:1.21",
          "destination": "ghcr.io/myorg/nginx:1.21",
          "architecture": "linux/amd64"
        },
        {
          "source": "docker.io/redis:alpine",
          "destination": "ghcr.io/myorg/redis:alpine",
          "architecture": "linux/arm64"
        }
      ]
```

### Using a Configuration File

```yaml
- name: Mirror images from file
  uses: monkescience/portage-action@v0.2.0
  with:
    images-file: .github/images.json
```

Example `images.json`:

```json
[
  {
    "source": "docker.io/nginx:latest",
    "destination": "ghcr.io/myorg/nginx:latest",
    "architecture": "linux/amd64"
  },
  {
    "source": "docker.io/postgres:13",
    "destination": "ghcr.io/myorg/postgres:13",
    "architecture": "linux/arm64"
  }
]
```

### Complete Workflow Example

```yaml
name: Mirror Images
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  workflow_dispatch:

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Mirror images
        id: mirror
        uses: monkescience/portage-action@v0.2.0
        with:
          images: |
            [
              {
                "source": "docker.io/nginx:latest",
                "destination": "ghcr.io/${{ github.repository }}/nginx:latest",
                "architecture": "linux/amd64"
              }
            ]

      - name: Show results
        run: |
          echo "Successfully mirrored: ${{ steps.mirror.outputs.success-count }}/${{ steps.mirror.outputs.total-count }} images"
          echo "Results: ${{ steps.mirror.outputs.results }}"
```

## Inputs

| Input         | Description                                              | Required | Default |
|---------------|----------------------------------------------------------|----------|---------|
| `images`      | JSON array of images to mirror                           | No*      |         |
| `images-file` | Path to a file containing JSON array of images to mirror | No*      |         |

*Either `images` or `images-file` must be provided, but not both.

## Outputs

| Output          | Description                                                                                                          |
|-----------------|----------------------------------------------------------------------------------------------------------------------|
| `results`       | JSON array containing results for each mirrored image with digest, size, source, destination, and status information |
| `success-count` | Number of images successfully mirrored                                                                               |
| `total-count`   | Total number of images processed                                                                                     |

### Example Output

```json
[
  {
    "source": "docker.io/nginx:latest",
    "destination": "ghcr.io/myorg/nginx:latest",
    "digest": "sha256:abc123...",
    "size": "142MB",
    "status": "success"
  }
]
```

## Prerequisites

- The runner must have Docker installed (available by default on `ubuntu-latest`)
- Authentication to source and destination registries must be configured using `docker/login-action` or similar
- Appropriate permissions to pull from source and push to destination registries

## Authentication

Before using this action, ensure you're authenticated to both source and destination registries:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

- name: Login to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

## Error Handling

- If any image fails to mirror, the action will continue processing remaining images
- Failed images will be reported in the results with error details
- The action will fail overall if any images fail to mirror
- Detailed logs are provided for troubleshooting
