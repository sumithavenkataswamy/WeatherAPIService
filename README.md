# WeatherAPIService

A lightweight ASP.NET Core Web API that provides weather forecasts and demonstrates end-to-end CI/CD pipeline using GitHub Actions, GitHub Container Registry (GHCR), and Azure Container Apps.

---

## ✨ Features
- ASP.NET Core Weather API
- Dockerized application
- GitHub Actions for automated build, publish, and deployment
- Multi-environment deployments (Dev, Test, Prod)
- Secrets managed via GitHub Environments

---

## 🌐 Project Repository
GitHub: [WeatherAPIService](https://github.com/sumithavenkataswamy/WeatherAPIService)

---

## 📅 CI/CD Workflow Summary

This project uses GitHub Actions to:

1. **Build and publish** Docker images to GitHub Container Registry
2. **Deploy** the container image to Azure Container Apps in Dev, Test, and Prod environments

### Branch Strategy:
- `develop` → triggers **Dev** deployment
- `main` → triggers **Test** and then **Prod** deployment

---

## ✨ Setup Instructions

### 1. Create Azure Resources

#### a. Azure Container App
```bash
az group create --name my-resource-group --location eastus
az containerapp environment create \
  --name my-containerapp-env \
  --resource-group my-resource-group \
  --location eastus
az containerapp create \
  --name weather-service-app \
  --resource-group my-resource-group \
  --environment my-containerapp-env \
  --image ghcr.io/<your-username>/weather-service-api:latest \
  --target-port 80 \
  --ingress 'external' \
  --registry-server ghcr.io \
  --query properties.configuration.ingress.fqdn
```

#### b. Create Service Principal for GitHub Actions
```bash
az ad sp create-for-rbac --name "github-container-deployer" --role contributor \
  --scopes /subscriptions/<subscription-id> --sdk-auth
```
Copy the JSON output securely. It will be used as `AZURE_CREDENTIALS` secret.


---

### 2. Configure GitHub Secrets for Each Environment
Go to **GitHub > Settings > Environments** and create `dev`, `test`, `prod` environments.

Add the following secrets in each:
- `AZURE_CREDENTIALS` - JSON output from service principal creation
- `AZURE_SUBSCRIPTION` - Your Azure subscription ID
- `CONTAINER_APP` - The Azure Container App name
- `RESOURCE_GROUP` - Azure Resource Group name
- `GHCR_USERNAME` - Your GitHub username (for `ghcr.io`)
- `TOKEN` - GitHub Personal Access Token with `write:packages` and `read:packages`

---

## 💡 GitHub Actions Workflow Explained

### File: `.github/workflows/build-and-deploy.yml`

#### Build & Publish Job:
- **Checkout** the code
- Generate unique `BUILD_VERSION`
- **Login** to GHCR
- **Build** and **push** Docker image to GHCR with tags:
  - `latest`
  - `<BUILD_VERSION>`

#### Deploy Dev:
- Triggered on `develop` branch
- Logs into Azure
- Updates Container App using image from GHCR

#### Deploy Test:
- Triggered on `main` branch
- Same process as Dev but deployed to test environment

#### Deploy Prod:
- Triggered after Test deployment completes
- Same process as above

---

## ✅ Example Build & Push Step
```yaml
- name: Build Docker image
  run: |
    docker build \
      -t ghcr.io/${{ github.actor }}/weather-service-api:${{ env.BUILD_VERSION }} \
      -t ghcr.io/${{ github.actor }}/weather-service-api:latest \
      .

- name: Push Docker images
  run: |
    docker push ghcr.io/${{ github.actor }}/weather-service-api:${{ env.BUILD_VERSION }}
    docker push ghcr.io/${{ github.actor }}/weather-service-api:latest
```

---

## 🎓 Useful Azure CLI Commands

**Update Container App manually (if needed):**
```bash
az containerapp update \
  --name <your-container-app-name> \
  --resource-group <your-resource-group> \
  --image ghcr.io/<your-username>/weather-service-api:<version> \
  --set registries="ghcr.io=<username>=<token>"
```

---

## 📈 Deployment Flow Chart
```
[push to develop] -> build-and-publish -> deploy-dev

[push to main] -> build-and-publish -> deploy-test -> deploy-prod
```

---

## 🚀 Ready to Use
Once everything is configured, every push to `develop` or `main` will automatically:
- Build and push Docker image
- Deploy to appropriate Azure Container App environment

---

## ❤️ Contributing
PRs and suggestions welcome!
