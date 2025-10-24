# Helm Demo – Managed Kubernetes Cluster on Linode ☸️

## 🧩 Overview
This project demonstrates how to **deploy a replicated MongoDB database** on a **Managed Kubernetes Cluster (LKE)** using **Helm**.  
It also shows how to configure **persistent storage**, expose the database through a **Mongo Express UI**, and manage everything efficiently using Helm commands.

---

## 🚀 Objectives
- Create a **Managed Kubernetes Cluster** on **Linode** (LKE)
- Deploy **MongoDB** using Helm charts
- Configure **Persistence Volumes** for data durability
- Deploy **Mongo Express** for UI access
- Use **Ingress Controller** (NGINX) to expose services
- Manage app lifecycle with **Helm** (`install`, `upgrade`, `rollback`, `uninstall`)

---

## 🧱 Prerequisites
Make sure you have the following installed:

| Tool | Description |
|------|--------------|
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | Command-line tool for Kubernetes |
| [helm](https://helm.sh/docs/intro/install/) | Package manager for Kubernetes |
| [linode-cli](https://www.linode.com/docs/guides/linode-cli/) | Linode command-line interface |
| A Linode account with LKE enabled | Managed Kubernetes Environment |

---

## ⚙️ Cluster Setup (Linode LKE)

Create a managed Kubernetes cluster on Linode using the CLI:

```bash
linode-cli lke cluster-create   --label my-k8s   --region us-east   --node_pools 'type=g6-standard-2,count=3'
```

Once created, download the `kubeconfig` file from Linode Dashboard or CLI and connect `kubectl`:

```bash
export KUBECONFIG=/path/to/linode-kubeconfig.yaml
kubectl get nodes
```

You should see all nodes in `Ready` state.

---

## 🛠️ Deploy MongoDB using Helm

Add the Bitnami Helm repo and update it:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Inspect the chart and customize your values:

```bash
helm show values bitnami/mongodb > my-values.yaml
```

Example snippet of `my-values.yaml`:
```yaml
replicaCount: 3
auth:
  rootPassword: mySecret
persistence:
  enabled: true
  size: 10Gi
```

Install MongoDB with your custom values:
```bash
helm install mongo bitnami/mongodb -f my-values.yaml -n data --create-namespace
```

Check deployment status:
```bash
kubectl get pods -n data
kubectl get svc -n data
```

---

## 💻 Deploy Mongo Express UI

Install Mongo Express and connect it to your MongoDB service:

```bash
helm install mongo-express bitnami/mongo-express   --set mongodbServer=mongo.data.svc.cluster.local   --namespace data
```

---

## 🌐 Expose the UI via Ingress

Install an NGINX Ingress controller:

```bash
helm install nginx-ingress ingress-nginx/ingress-nginx   --namespace ingress --create-namespace
```

Create an `Ingress` resource (mongo-express-ingress.yaml):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mongo-express-ingress
  namespace: data
spec:
  rules:
  - host: mongo.mycluster.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mongo-express
            port:
              number: 8081
```

Apply it:
```bash
kubectl apply -f mongo-express-ingress.yaml
kubectl get ingress -n data
```

Open your browser at the assigned **External IP** to access Mongo Express.

---

## 🔁 Helm Lifecycle Management

Upgrade configuration:
```bash
helm upgrade mongo bitnami/mongodb -f my-values.yaml -n data
```

Rollback to a previous version:
```bash
helm rollback mongo 1 -n data
```

Uninstall release:
```bash
helm uninstall mongo -n data
```

List all releases:
```bash
helm ls -A
```

---

## 📂 Project Structure
```
.
├── my-values.yaml                  # Custom Helm values for MongoDB
├── mongo-express-ingress.yaml      # Ingress resource definition
├── Helm_Demo_Managed_K8s_Cluster_Detailed_Summary.pdf
├── Helm_Demo_Managed_K8s_Cluster_Detailed_Summary.docx
└── README.md                       # This file
```

---

## 📘 References
- [Helm Official Docs](https://helm.sh/docs/)
- [Bitnami MongoDB Chart](https://artifacthub.io/packages/helm/bitnami/mongodb)
- [Linode Kubernetes Engine (LKE)](https://www.linode.com/products/kubernetes/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)

---

## 🧠 Author
**Mohamed Eldemerdash**  
Cloud & DevOps Enthusiast | Kubernetes | Helm | CI/CD  
🔗 [LinkedIn](https://www.linkedin.com)

---
