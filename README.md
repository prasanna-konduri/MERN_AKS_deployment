
# Deploying a MERN Application on Azure Kubernetes Service (AKS)

A step-by-step approach to deploying a MERN (MongoDB, Express, React, Node.js) application on Azure Kubernetes Service (AKS) is as follows, adhering to industry best practices for security and optimization.

## Prerequisites

- **Azure Account**: Ensure you have an active Azure subscription.
- **Azure CLI**: [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- **Kubernetes CLI (kubectl)**: [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- **Docker**:  [Install Docker](https://docs.docker.com/get-docker/)
- **Git**: [Install Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
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

### Kubernetes Configuration Files.
- Create a deployment. yaml and service.yaml files for each service

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
