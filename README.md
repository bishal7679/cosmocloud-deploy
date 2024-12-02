# 🌟 Cosmocloud Deploy Helm Chart
Welcome to the Cosmocloud Deploy Helm Chart! This Helm chart helps you deploy three essential components—Backend, Frontend, and Redis Database—together in a Kubernetes cluster. Below, you'll find detailed instructions, explanations for each file, and a walkthrough of the deployment process.

## 📄 Chart Structure
The cosmocloud-deploy chart is structured as follows:

```bash
cosmocloud-deploy/
├── templates/               # Contains all Kubernetes manifests
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── redis-deployment.yaml
│   ├── redis-service.yaml
│   ├── configmap.yaml
│   └── NOTES.txt         
├── Chart.yaml               # Metadata about this Helm chart
├── values.yaml              # Default configuration values

```
