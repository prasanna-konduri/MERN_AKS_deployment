
# Deploying a MERN Application on Azure Kubernetes Service (AKS)

This guide provides a step-by-step approach to deploying a MERN (MongoDB, Express, React, Node.js) application on Azure Kubernetes Service (AKS), adhering to industry best practices for security and optimization.

## Prerequisites

- **Azure Account**: Ensure you have an active Azure subscription.
- **Azure CLI**: Installed and configured. [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- **Kubernetes CLI (kubectl)**: Installed. [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- **Docker**: Installed on your local machine. [Install Docker](https://docs.docker.com/get-docker/)
- **Git**: Installed. [Install Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- **Azure Key Vault**: For managing secrets. [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/)

## Step 1. Clone the Repository

```bash
git clone https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices.git
cd SampleMERNwithMicroservices
```

## Step 2. Set Up Environment Variables

Create `.env` files for each backend service with the following content:

**helloService/.env**:
```
PORT=3001
```

**profileService/.env**:
```
PORT=3002
MONGO_URL="your_mongo_connection_string"
```

Replace `"your_mongo_connection_string"` with your actual MongoDB connection string.

## Step 3. Create Dockerfiles

Create `Dockerfile` for each service:

- helloService/Dockerfile
- profileService/Dockerfile
- frontend/Dockerfile:



## Step 4. Build and Push Docker Images

a. Ensure you’re logged into Azure:

```bash
az login
```

b. Ensure Azure CLI is installed and updated:

```bash
az version
az upgrade
```

c. Create Azure Container Registry (ACR):

```bash
az acr create --resource-group <ResourceGroup> --name <ACRName> --sku Basic
```

d. Login to ACR:

```bash
az acr login --name <ACRName>
```

e. Build and Push Images:

Navigate to each service directory and execute:

```bash
docker build -t <ACRName>.azurecr.io/<service-name>:v1 .
docker push <ACRName>.azurecr.io/<service-name>:v1
```

Replace `<service-name>` with `helloservice`, `profileservice`, and `frontend` respectively.

f. Confirm Images Are Pushed to ACR:

```bash
az acr repository list --name <ACRName> --output table
```


## Step 5. Create AKS Cluster

```bash
az aks create --resource-group <ResourceGroup> --name <AKSClusterName> --node-count 3 --enable-managed-identity --generate-ssh-keys
az aks get-credentials --resource-group <ResourceGroup> --name <AKSClusterName>
```

## Step 6. Deploy Application on AKS

### Kubernetes Configuration Files

#### helloService Deployment and Service

**helloservice-deployment.yml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloservice
  namespace: ms
spec:
  replicas: 2
  selector:
    matchLabels:
      app: helloservice
  template:
    metadata:
      labels:
        app: helloservice
    spec:
      containers:
      - name: helloservice
        image: <ACRName>.azurecr.io/helloservice:v1
        ports:
        - containerPort: 3001
```

**helloservice-service.yml**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloservice
  namespace: ms
spec:
  selector:
    app: helloservice
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3001
```

#### profileService Deployment, Service, and Secret

**profileservice-deployment.yml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: profileservice
  namespace: ms
spec:
  replicas: 2
  selector:
    matchLabels:
      app: profileservice
  template:
    metadata:
      labels:
        app: profileservice
    spec:
      containers:
      - name: profileservice
        image: <ACRName>.azurecr.io/profileservice:v1
        ports:
        - containerPort: 3002
        env:
        - name: MONGO_URL
          valueFrom:
            secretKeyRef:
              name: profile-secret
              key: MONGO_URL
```

**profileservice-service.yml**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: profileservice
  namespace: ms
spec:
  selector:
    app: profileservice
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3002
```

**secret.yml**:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: profile-secret
  namespace: ms
type: Opaque
data:
  MONGO_URL: <Base64_Encoded_Mongo_URL>
```

#### frontend Deployment and Service

**frontend-deployment.yml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: ms
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: <ACRName>.azurecr.io/frontend:v1
        ports:
        - containerPort: 80
```

**frontend-service.yml**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: ms
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

### Apply Configurations

```bash
kubectl apply -f helloservice-deployment.yml
kubectl apply -f helloservice-service.yml
kubectl apply -f secret.yml
kubectl apply -f profileservice-deployment.yml
kubectl apply -f profileservice-service.yml
kubectl apply -f frontend-deployment.yml
kubectl apply -f frontend-service.yml
```

## Step 7: Expose the Application

### Install Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

### Create an Ingress Resource

**ingress.yaml**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mern-ingress
spec:
  rules:
  - host: <your-domain>
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

Apply the Ingress configuration:

```bash
kubectl apply -f ingress.yaml
```

## Step 8: Monitor and Troubleshoot

### Check Logs

```bash
kubectl logs <pod-name>
```

### Describe Resources

```bash
kubectl describe pod <pod-name>
```

### View Events

```bash
kubectl get events
```

---

For screenshots and detailed steps, refer to the documentation in the GitHub repository.
