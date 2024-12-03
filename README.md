# 🌟 Cosmocloud: Unified Application Deployment with Helm Chart
Welcome to the Cosmocloud Deploy Helm Chart! This Helm chart helps you deploy three essential components — Backend, Frontend, and Redis Database together in a Kubernetes cluster. Below, you'll find detailed instructions, explanations for each file, and a walkthrough of the deployment process.

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

## 📝 Files Explained
  
  ### 1. 📜 `Chart.yaml`
  This file defines metadata about the Helm chart. Here’s a breakdown of its key fields:

 - `apiVersion`: Specifies the Helm chart API version (`v2` for Helm 3).
 - `name`: Name of the chart (`cosmocloud-deploy`).
 - `description`: Brief summary of the chart.
 - `version`: Version of the Helm chart.
 - `appVersion`: Version of the application(s) being deployed.

    Purpose: Helps Helm understand and manage the chart versioning.

  ### 2. 🎛️ `values.yaml`
  This file contains default configurations for the chart. It allows you to:

  - Customize settings like image names, ports, and replicas.
  - Define ConfigMap values for environment variables.

   ***Key Configurations:***

```yaml
replicaCount: 1                          # Number of replicas for each Deployment
backend:                                 # Backend configuration
  image:
    repository: shreybatra/sample-backend
    pullPolicy: IfNotPresent            # Image pull policy when container runtime pulling image from registry
  service:
    type: ClusterIP
    port: 8000

frontend:                               # Frontend configuration
  image:
    repository: shreybatra/sample-frontend
    pullPolicy: IfNotPresent
  service:
    type: NodePort                     # using NodePort as service type to expose it to user in order to access the application
    port: 5173
    nodePort: 31000

redis:                                 # Redis configuration
  image:
    repository: redis
    pullPolicy: IfNotPresent
  service:
    type: ClusterIP
    port: 6379
```
Purpose: Simplifies customization without modifying templates directly.

  ### 3. 🗂️ `templates/configmap.yaml`
  Defines a ConfigMap used to pass environment variables to the applications. It references values from values.yaml.

  ***Key Snippet:***

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-env-configmap
  namespace: default
data:
  REDIS_URI: redis://redis-svc:6379
  BACKEND_URL: http://backend-svc:8000
```
Purpose: Centralized Configuration Management
The ConfigMap holds environment-specific details (`REDIS_URI` and `BACKEND_URL`) that are critical for the application's operation. By defining them in a ConfigMap:
 - Configurations are decoupled from the application code.
 - It becomes easier to manage and update configurations without modifying the application or its container image.

***Key-Value Pairs in data:***

`REDIS_URI:` Points to the Redis service (`redis-svc`) running within the cluster on the default port `6379`.
  - Enables the backend to connect to Redis for caching, session storage, or other database like tasks.
    
`BACKEND_URL:` Points to the backend service (`backend-svc`) running within the cluster on port `8000`.
  - Enables the frontend to send API requests to the backend.
---
  ### 4. ⚙️ `templates/backend-deployment.yaml`
  Creates the Backend Deployment with:

  👉 Container image: `shreybatra/sample-backend`.
  
  👉 Environment variables: `REDIS_URI` from the ConfigMap.
  
  👉 Port: Exposes the container on port `8000`.

```yaml

env:
    - name: REDIS_URI
      valueFrom:
         configMapKeyRef:
            name: app-env-configmap
            key: REDIS_URI
```
Purpose: Ensures the backend can connect to Redis using the environment variable.

  ### 5. 🌐 `templates/backend-service.yaml`
  Exposes the Backend as a ClusterIP service:

   - Service name: `backend-svc`.
   - Port: `8000`.
Key Snippet:

```yaml

spec:
  type: ClusterIP
  selector:              # using selector `app: backend` to connect the svc with the specific backend pod
    app: backend
  ports:
    - protocol: TCP
      port: {{ .Values.backend.service.port }}
```
Purpose: Provides internal networking for the backend service.

  ### 6. ⚙️ `templates/frontend-deployment.yaml`
  Creates the Frontend Deployment with:

   - Container image: `shreybatra/sample-frontend`.
   - Environment variables: `BACKEND_URL` from the ConfigMap.
   - Port: Exposes the container on port `5175`.
     
Key Snippet:

```yaml

env:
    - name: BACKEND_URL
      valueFrom:
          configMapKeyRef:
              name: app-env-configmap
              key: BACKEND_URL
```
Purpose: Connects the frontend to the backend service.

  ### 7. 🌐 `templates/frontend-service.yaml`
  Exposes the Frontend as a NodePort service:

   - Service name: `frontend-svc`.
   - Port: 5175.
   - NodePort: 31000.
Key Snippet:

```yaml

spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: {{ .Values.frontend.service.port }}
      nodePort: {{ .Values.frontend.service.nodePort }}
```
Purpose: Makes the frontend accessible from outside the cluster as using svc type NodePort.

  ### 8. ⚙️ `templates/redis-deployment.yaml`
  Creates the Redis Deployment with:

   - Container image: redis.
   - Port: Exposes the container on port 6379.
  Purpose: Deploys the Redis database for backend data storage.

  ### 9. 🌐 `templates/redis-service.yaml`
  Exposes Redis as a ClusterIP service:

   - Service name: redis-svc.
   - Port: 6379.
  Purpose: Enables backend to connect to Redis internally.

---

## 🚀 How to Use
   ### 1. Install Helm
  Ensure Helm is installed on your system:

```bash
helm version
```
   ### 2. Package the Chart

```bash
helm package cosmocloud-deploy
```
   ### 3. Deploy the Chart
   Install the chart into your Kubernetes cluster:

```bash
helm install cosmocloud cosmocloud-deploy
```
OUTPUT:
```bash
NAME: cosmocloud
LAST DEPLOYED: Mon Dec  2 20:09:19 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing cosmocloud-deploy.

Your release is named cosmocloud.

To learn more about the release, try:

  $ helm status cosmocloud
  $ helm get all cosmocloud
```
   ### 4. Verify Resources
   Check if all resources were deployed successfully:

```bash
kubectl get all -n default
```

   ### 5. Access Services
   - Frontend: Use the NodePort (e.g., `http://<NodeIP>:31000` or `localhost:31000` NOTE:- [use port forwarding to access the nodeport on localhost]).
     Refer this post if you are using kind cluster for config.yaml [Link](https://stackoverflow.com/questions/62432961/how-to-use-nodeport-with-kind)

## 🛠️ Customization
Modify `values.yaml` to adjust configurations such as image versions, ports, and environment variables.

## 📚 Troubleshooting
 - Error: `ImagePullBackOff`
    - Ensure the specified images are accessible from your cluster.
 - Error: `CrashLoopBackOff`
    - Check logs with `kubectl logs <pod-name>`.
      
### 🎉 Congratulations!
You’ve successfully deployed your Cosmocloud applications using a Helm chart! Happy Helm-ing! 🚀
