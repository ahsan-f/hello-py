# hello-py
This GitHub Actions workflow will build a Docker image from your Dockerfile and push it to the GitHub Container Registry (GHCR). It will be triggered on pushes to the main branch.

Here is the workflow file (.github/workflows/docker-build.yml):

YAML

name: Docker Image CI

on:
  push:
    branches: [ main ]
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  build_and_push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write # Needed to push to GHCR

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Log in to the GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Define image name and tags
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=sha,prefix=,suffix=,format=short
            type=branch,enable=${{ github.ref == 'refs/heads/main' }},priority=700,event=push
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
Workflow Explanation
1. Structure and Trigger
name: Docker Image CI: The name displayed for the workflow in GitHub Actions.

on: push: branches: [ main ]: The workflow will run automatically whenever code is pushed to the main branch.

workflow_dispatch: This allows you to manually trigger the workflow from the "Actions" tab in your GitHub repository.

2. Job: build_and_push
runs-on: ubuntu-latest: Specifies the runner environment.

permissions: Explicitly grants the necessary permissions:

contents: read: To read the code from the repository.

packages: write: Crucial for pushing the built Docker image to the GitHub Container Registry (GHCR).

3. Steps
Checkout repository: Downloads your repository's code to the runner.

Log in to the GitHub Container Registry: Uses the docker/login-action to authenticate with GHCR using your GitHub username (github.actor) and the automatically provided GITHUB_TOKEN secret for the password.

Set up Docker Buildx: Sets up the Docker Buildx environment, which is required for efficient, modern, and multi-platform Docker builds.

Define image name and tags: Uses the docker/metadata-action to automatically generate appropriate tags for the image.

The image name will be ghcr.io/<YOUR_USERNAME>/<YOUR_REPOSITORY_NAME>.

It tags the image with the short commit SHA.

For pushes to main, it also applies the branch name tag and the latest tag.

Build and push Docker image: The core step using docker/build-push-action.

context: .: Uses the current directory as the build context.

file: ./Dockerfile: Specifies your Dockerfile location.

push: true: Tells the action to push the resulting image to the registry.

tags: ${{ steps.meta.outputs.tags }}: Applies the dynamically generated tags.

Prerequisites
Ensure your Dockerfile and hello.py are in the root directory of your GitHub repository.

Commit this workflow file to your repository under .github/workflows/docker-build.yml.
