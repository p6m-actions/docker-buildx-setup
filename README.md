# Docker Buildx Setup

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/docker-buildx-setup?style=flat-square&label=Latest%20Release&color=blue)

## Description

A GitHub Action that sets up Docker Buildx for multi-platform container builds. This action is a simple wrapper around the official [docker/setup-buildx-action@v2](https://github.com/docker/setup-buildx-action) to make it easier to use in your workflows.

## Usage

```yaml
- name: Set up Docker Buildx
  uses: p6m-actions/docker-buildx-setup@v1
```

## Outputs

| Name | Description |
|------|-------------|
| `name` | The name of the buildx instance |
| `driver` | The driver being used |
| `endpoint` | The builder endpoint |

## Examples

### Basic usage

```yaml
steps:
  - name: Checkout Code
    uses: actions/checkout@v3

  - name: Set up Docker Buildx
    id: buildx
    uses: p6m-actions/docker-buildx-setup@v1

  - name: Login to Docker Registry
    uses: p6m-actions/docker-repository-login@v1
    with:
      registry: your-registry.example.com
      username: ${{ secrets.REGISTRY_USERNAME }}
      password: ${{ secrets.REGISTRY_PASSWORD }}

  - name: Build and push Docker image
    uses: docker/build-push-action@v4
    with:
      context: .
      push: true
      platforms: linux/amd64,linux/arm64
      tags: your-registry.example.com/your-image:latest
```

### Multi-platform build example

```yaml
steps:
  - name: Checkout Code
    uses: actions/checkout@v3

  - name: Set up Docker Buildx
    id: buildx
    uses: p6m-actions/docker-buildx-setup@v1

  - name: Build and push Docker image
    env:
      PLATFORMS: linux/amd64,linux/arm64
    run: |
      docker buildx build \
        --platform $PLATFORMS \
        --tag your-registry.example.com/your-image:latest \
        --push \
        .
```

### Using with docker-buildx-build-publish

This action pairs seamlessly with [docker-buildx-build-publish](https://github.com/p6m-actions/docker-buildx-build-publish) for advanced build workflows:

```yaml
steps:
  - name: Checkout Code
    uses: actions/checkout@v3

  - name: Set up Docker Buildx
    id: buildx
    uses: p6m-actions/docker-buildx-setup@v1

  - name: Build and push Docker image
    uses: p6m-actions/docker-buildx-build-publish@v1
    with:
      skip-setup: true
      builder-name: ${{ steps.buildx.outputs.name }}
      image: your-registry.example.com/your-image
      tags: |
        latest
        v1.0.0
      platforms: linux/amd64,linux/arm64
      registry: your-registry.example.com
      username: ${{ secrets.REGISTRY_USERNAME }}
      password: ${{ secrets.REGISTRY_PASSWORD }}
```

### Advanced builder configuration

Access builder details for custom workflows:

```yaml
steps:
  - name: Set up Docker Buildx
    id: buildx
    uses: p6m-actions/docker-buildx-setup@v1

  - name: Show builder information
    run: |
      echo "Builder name: ${{ steps.buildx.outputs.name }}"
      echo "Driver: ${{ steps.buildx.outputs.driver }}"
      echo "Endpoint: ${{ steps.buildx.outputs.endpoint }}"

  - name: Custom build with builder details
    run: |
      docker buildx build \
        --builder ${{ steps.buildx.outputs.name }} \
        --platform linux/amd64,linux/arm64 \
        --tag your-image:latest \
        --push \
        .
```