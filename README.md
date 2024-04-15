# Helm Demo - Managed Kubernetes Cluster

## Overview

In this project, I will deploy a managed Kubernetes cluster on Linode Kubernetes Engine (LKE) and demonstrate various tasks such as deploying MongoDB using Helm, configuring data persistence with Linode's cloud storage, deploying a UI client Mongo-Express, and setting up Nginx Ingress.

### What We Will Build/Deploy:

- Deploy MongoDB using Helm with 3 replicas using StatefulSet.
- Configure data persistence with Linode's cloud storage.
- Deploy UI client Mongo-Express.
- Configure Nginx Ingress.

## Step-by-Step Guide

### 1. Create Kubernetes Cluster on LKE

<img src="https://i.imgur.com/4nB2ZyJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

<img src="https://i.imgur.com/tidP5vi.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

<img src="https://i.imgur.com/1cEdbZS.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

- Download the `kubeconfig.yaml` file necessary to access the cluster from your local machine.
- Set the kubeconfig as an environment variable:

```bash
chmod 400 test-kubeconfig.yaml
export KUBECONFIG=test-kubeconfig.yaml
```

<img src="https://i.imgur.com/wC0a83G.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


---

### 2. Deploy MongoDB StatefulSet

- Install Helm:

```
brew install helm
```

- Add the Bitnami Helm chart repository:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

- Search for the MongoDB Helm chart:

```
helm search repo bitnami/mongodb
```

- Create a values file to override parameters:

```
touch helm-mongodb.yaml
``

- Edit the values file:

```
architecture: replicaset
replicaCount: 3
persistence:
  storageClass: "linode-block-storage"
auth:
  rootPassword: secret-root-pwd
```

```
- Install MongoDB using Helm:

```
helm install mongodb --values helm-mongodb.yaml bitnami/mongodb
```

<img src="https://i.imgur.com/Xuyky8y.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

<img src="https://i.imgur.com/tb0KFwQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

### 3. Deploy MongoExpress

- Create a deployment and service YAML file for MongoExpress:

```
touch helm-mongo-express.yaml
```

- Edit the deployment and service YAML file:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          value: root
        - name: ME_CONFIG_MONGODB_SERVER
          value: mongodb-0.mongodb-headless
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD 
          valueFrom:
            secretKeyRef:
              name: mongodb
              key: mongodb-root-password
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
```

- Apply the configuration:

```
kubectl apply -f helm-mongo-express.yaml
```

<img src="https://i.imgur.com/fmEB14X.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

### 4. Configure Nginx Ingress

- Install the Nginx Ingress Controller:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```

- Create an Ingress rule:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: mongo-express
spec:
  rules:
  - host: YOUR_HOST_DNS_NAME
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: mongo-express-service
              port:
                number: 8081
```

- Apply the configuration:

```
kubectl apply -f helm-ingress.yaml
kubectl get ingress

```

<img src="https://i.imgur.com/Eegywjp.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
