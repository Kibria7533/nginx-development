# CI/CD for NGINX Development Using GitHub Actions

This project uses GitHub Actions for automated CI/CD of different Docker images for various environments (`dev`, `staging`, `prod`).

---

## 📁 Directory Structure

```
nginx-development/
├── .github/
│   └── workflows/
│       ├── CI.dev.yml         # CI workflow for dev branch
│       ├── CI.staging.yml     # CI workflow for staging branch
│       └── CI.master.yml      # CI workflow for production (main branch)
├── Dockerfile                 # General or dev Dockerfile
├── Dockerfile.stage           # Dockerfile for staging
├── Dockerfile.prod            # Dockerfile for production
├── versions.yaml              # Stores image version tags
├── index.html
└── README.md
```

---

## 🔧 GitHub Actions Workflows

### ✅ Common Setup

Each workflow file runs on its respective branch (`dev`, `staging`, or `main`) and does the following:

1. **Extract the branch name**
2. **Check out the repository**
3. **Extract image version from `versions.yaml`**
4. **Build Docker image with the extracted version**
5. **Login to Docker Hub and push the image**

---

## 📜 Sample Workflow (CI.master.yml)

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Extract branch name
        shell: bash
        run: echo "##[set-output name=branch;]$(echo ${GITHUB_REF#refs/heads/})"
        id: extract_branch

      - uses: actions/checkout@v2

      - name: Extract imageAppVersion from values.yaml
        id: get_version
        run: |
          image_version=$(yq eval '.image.prod_tag' ./versions.yaml)
          echo "IMAGE_VERSION=$image_version" >> $GITHUB_ENV

      - name: Docker build
        run: |
          docker build -t ${{ secrets.DOCKER_USER }}/nginx-development-prod:${{ env.IMAGE_VERSION }} -f Dockerfile.prod .

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USER }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Docker push
        run: |
          docker push ${{ secrets.DOCKER_USER }}/nginx-development-prod:${{ env.IMAGE_VERSION }}
```

---

## 🔐 Secrets Used

Make sure you have the following secrets set in your GitHub repo:
- `DOCKER_USER`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password or access token
  kubectl -n app-of-apps create secret docker-registry regcred --docker-username=xxxxxxx --docker-password=xxxxxxxxxxxxxx

---

## 📦 versions.yaml

This file holds image tags for different environments.

```yaml
image:
  dev_tag: "dev-1.0.0"
  stage_tag: "stage-1.0.0"
  prod_tag: "prod-1.0.0"
```

Each workflow pulls the correct tag based on the branch it runs from.

---

## 🚀 How It Works

- When you push to `main`, the `CI.master.yml` workflow runs.
- It reads the `prod_tag` from `versions.yaml`
- Builds and pushes a Docker image to Docker Hub using the tag.
- Staging and dev branches follow the same pattern with different Dockerfiles and tags.

---

## ✅ Next Steps

- Add ArgoCD or Docker Compose to auto-deploy images by tag.
- Add Slack notifications or test stages for each environment.