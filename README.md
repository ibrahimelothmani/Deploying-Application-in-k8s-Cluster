# Deploying MongoDB Application in Kubernetes Cluster

A complete Kubernetes deployment project demonstrating how to deploy MongoDB database and Mongo Express web interface with proper configuration management using ConfigMaps and Secrets.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Components](#components)
- [Prerequisites](#prerequisites)
- [Configuration Details](#configuration-details)
- [Deployment Instructions](#deployment-instructions)
- [Accessing the Application](#accessing-the-application)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Cleanup](#cleanup)
- [Security Considerations](#security-considerations)

## 🎯 Overview

This project demonstrates a production-ready Kubernetes deployment of:

- **MongoDB Database**: A NoSQL database running in a containerized environment
- **Mongo Express**: A web-based MongoDB administration interface

The deployment showcases Kubernetes best practices including:

- ✅ Secret management for sensitive data
- ✅ ConfigMaps for configuration management
- ✅ Service discovery and internal networking
- ✅ External access via LoadBalancer and NodePort
- ✅ Environment variable injection
- ✅ Pod orchestration and management

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Kubernetes Cluster                     │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │       Mongo Express (Port 8081)                │    │
│  │  - Web UI for MongoDB Management               │    │
│  │  - Accessible via NodePort 30007               │    │
│  │  - LoadBalancer Service                        │    │
│  └─────────────────┬──────────────────────────────┘    │
│                    │                                     │
│                    │ (Connects via mongodb-service)     │
│                    ▼                                     │
│  ┌────────────────────────────────────────────────┐    │
│  │       MongoDB Database (Port 27017)            │    │
│  │  - NoSQL Database                              │    │
│  │  - Internal Service                            │    │
│  │  - Credentials from Secret                     │    │
│  └────────────────────────────────────────────────┘    │
│                                                          │
│  Configuration Resources:                               │
│  ├─ mongodb-secret (credentials)                        │
│  └─ mongodb-config (database URL)                       │
└─────────────────────────────────────────────────────────┘
```

## 📦 Components

### 1. MongoDB Deployment & Service

**File**: [`mongo-deployment.yaml`](mongo-deployment.yaml)

- **Deployment**: Runs MongoDB container with root credentials from Secret
- **Service**: Internal ClusterIP service (`mongodb-service`) for inter-pod communication
- **Port**: 27017 (standard MongoDB port)
- **Replicas**: 1 instance

### 2. Mongo Express Deployment & Service

**File**: [`mongo-express.yaml`](mongo-express.yaml)

- **Deployment**: Web-based MongoDB admin interface
- **Service**: LoadBalancer type with NodePort for external access
- **Port**: 8081 (Mongo Express default port)
- **NodePort**: 30007 (accessible from outside the cluster)
- **Replicas**: 1 instance

### 3. MongoDB Secret

**File**: [`mongodb-secret.yaml`](mongodb-secret.yaml)

Stores **base64-encoded** credentials:

- `mongo-root-username`: bW9uZ28tZGItdXNlcm5hbWU= (decodes to: `mongo-db-username`)
- `mongo-root-password`: bW9uZ28tZGItcGFzc3dvcmQ= (decodes to: `mongo-db-password`)

> ⚠️ **Security Note**: These are example credentials and should be changed in production!

### 4. MongoDB ConfigMap

**File**: [`mongodb-config.yaml`](mongodb-config.yaml)

Stores non-sensitive configuration:

- `database_url`: mongodb-service (hostname of the MongoDB service)

## ✅ Prerequisites

Before deploying this application, ensure you have:

- **Kubernetes Cluster**: Minikube, Kind, or any cloud Kubernetes cluster (GKE, EKS, AKS)
- **kubectl**: Kubernetes command-line tool installed and configured
- **Access Permissions**: Ability to create deployments, services, secrets, and configmaps

### Verify your setup:

```bash
# Check kubectl is installed and connected
kubectl version --short

# Check cluster status
kubectl cluster-info

# Verify you can create resources
kubectl auth can-i create deployments
```

## ⚙️ Configuration Details

### Environment Variables

**MongoDB Container**:

- `MONGO_INITDB_ROOT_USERNAME` ← mongodb-secret/mongo-root-username
- `MONGO_INITDB_ROOT_PASSWORD` ← mongodb-secret/mongo-root-password

**Mongo Express Container**:

- `ME_CONFIG_MONGODB_ADMINISTRATOR` ← mongodb-secret/mongo-root-username
- `ME_CONFIG_MONGODB_PASSWORD` ← mongodb-secret/mongo-root-password
- `ME_CONFIG_MONGODB_SERVER` ← mongodb-config/database_url

### Service Types

1. **MongoDB Service**: ClusterIP (default) - Internal only
2. **Mongo Express Service**: LoadBalancer with NodePort - External access

## 🚀 Deployment Instructions

Deploy the resources in the correct order to ensure dependencies are met:

### Step 1: Create the Secret (First Priority)

```bash
kubectl apply -f mongodb-secret.yaml
```

The Secret must be created first because both MongoDB and Mongo Express deployments reference it.

### Step 2: Create the ConfigMap

```bash
kubectl apply -f mongodb-config.yaml
```

The ConfigMap stores the MongoDB service hostname for Mongo Express to connect.

### Step 3: Deploy MongoDB

```bash
kubectl apply -f mongo-deployment.yaml
```

This creates:

- MongoDB Deployment (1 replica)
- MongoDB Service (mongodb-service)

### Step 4: Deploy Mongo Express

```bash
kubectl apply -f mongo-express.yaml
```

This creates:

- Mongo Express Deployment (1 replica)
- Mongo Express Service (LoadBalancer with NodePort 30007)

### Alternative: Deploy All at Once

```bash
# Deploy in order
kubectl apply -f mongodb-secret.yaml
kubectl apply -f mongodb-config.yaml
kubectl apply -f mongo-deployment.yaml
kubectl apply -f mongo-express.yaml
```

## 🌐 Accessing the Application

### For Minikube

1. Get the Minikube IP:

```bash
minikube ip
```

2. Access Mongo Express:

```
http://<minikube-ip>:30007
```

Or use the Minikube service command:

```bash
minikube service mongo-express
```

### For Cloud Kubernetes (GKE/EKS/AKS)

1. Get the LoadBalancer external IP:

```bash
kubectl get service mongo-express
```

2. Access Mongo Express:

```
http://<EXTERNAL-IP>:8081
```

### For Local Kubernetes (Kind, K3s)

Use port forwarding:

```bash
kubectl port-forward service/mongo-express 8081:8081
```

Then access: `http://localhost:8081`

## 🔍 Verification

### Check All Resources

```bash
# View all resources
kubectl get all

# Check specific resources
kubectl get deployments
kubectl get services
kubectl get pods
kubectl get configmaps
kubectl get secrets
```

### Check Pod Status

```bash
# Get detailed pod information
kubectl get pods -o wide

# Check pod logs
kubectl logs -f deployment/mongodb-deployment
kubectl logs -f deployment/mongo-express
```

### Check Service Endpoints

```bash
# View service details
kubectl describe service mongodb-service
kubectl describe service mongo-express

# Check endpoints
kubectl get endpoints
```

### Expected Output

All pods should be in `Running` status:

```
NAME                                   READY   STATUS    RESTARTS   AGE
mongodb-deployment-xxxxxxxxxx-xxxxx    1/1     Running   0          2m
mongo-express-xxxxxxxxxx-xxxxx         1/1     Running   0          1m
```

## 🔧 Troubleshooting

### Pod Not Starting

**Check pod status**:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

**Common issues**:

- Image pull errors: Check internet connectivity
- Secret not found: Ensure mongodb-secret is created first
- ConfigMap not found: Ensure mongodb-config is created

### Mongo Express Cannot Connect to MongoDB

**Check**:

1. MongoDB pod is running:

   ```bash
   kubectl get pods -l app=mongodb
   ```

2. MongoDB service exists:

   ```bash
   kubectl get service mongodb-service
   ```

3. Credentials are correct in Secret:

   ```bash
   kubectl get secret mongodb-secret -o yaml
   ```

4. ConfigMap has correct database URL:
   ```bash
   kubectl get configmap mongodb-config -o yaml
   ```

### Cannot Access Mongo Express

**For Minikube**:

```bash
# Check if minikube tunnel is needed
minikube service list
minikube service mongo-express --url
```

**For Cloud Kubernetes**:

```bash
# Check LoadBalancer status
kubectl get service mongo-express
# Wait until EXTERNAL-IP is assigned (not <pending>)
```

### Debug Service Selector

Check if service selectors match pod labels:

```bash
# MongoDB Service - check if selector matches
kubectl get service mongodb-service -o yaml | grep -A 2 selector
kubectl get pods -l app=mongodb -o wide
```

## 🧹 Cleanup

Remove all resources created by this project:

```bash
# Delete deployments and services
kubectl delete -f mongo-express.yaml
kubectl delete -f mongo-deployment.yaml
kubectl delete -f mongodb-config.yaml
kubectl delete -f mongodb-secret.yaml

# Or delete all at once
kubectl delete -f .

# Verify cleanup
kubectl get all
```

## 🔒 Security Considerations

### 🚨 Important Security Notes

1. **Change Default Credentials**: The provided credentials are examples only

   ```bash
   # Create new base64-encoded credentials
   echo -n 'your-username' | base64
   echo -n 'your-password' | base64
   ```

2. **Use Kubernetes Secrets Encryption**: Enable encryption at rest for Secrets
3. **Network Policies**: Implement network policies to restrict traffic between pods

4. **RBAC**: Use Role-Based Access Control to limit who can access secrets

5. **Don't Commit Secrets**: Never commit `mongodb-secret.yaml` with real credentials to version control

   - Add to `.gitignore`
   - Use sealed-secrets or external secret management (Vault, AWS Secrets Manager)

6. **Use TLS/SSL**: In production, configure MongoDB with TLS encryption

7. **Limit External Access**: Consider using Ingress with authentication instead of LoadBalancer

### Production Recommendations

- Use **PersistentVolumes** for MongoDB data persistence
- Implement **MongoDB ReplicaSets** for high availability
- Set up **Resource Limits** and **Requests** for pods
- Configure **Health Checks** (liveness and readiness probes)
- Use **Horizontal Pod Autoscaling** for Mongo Express
- Implement **Backup Strategy** for MongoDB data
- Use **Ingress Controller** instead of NodePort for production traffic

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

---

**Project Structure**:

```
.
├── README.md                    # This file
├── mongodb-secret.yaml          # Secret with MongoDB credentials
├── mongodb-config.yaml          # ConfigMap with database URL
├── mongo-deployment.yaml        # MongoDB deployment and service
└── mongo-express.yaml           # Mongo Express deployment and service
```

**Author**: Ibrahim El Othmani  
**Last Updated**: December 2025
