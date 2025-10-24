# Managed-Kubernetes-Cluster-on-Linode
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

Create a managed Kubernetes cluster on Linode using the UI:

<img width="1349" height="671" alt="image" src="https://github.com/user-attachments/assets/433b9439-9204-4118-acfc-64e130517fff" />

---

Once created, download the kubeconfig file from Linode Dashboard:
<img width="1306" height="78" alt="image" src="https://github.com/user-attachments/assets/5917cdfe-b16a-4a12-8a17-97bbdab9befd" />

Then Set read only permission for my user account (so no unauthorized user can make changes to K8S Cluster)‎
<img width="1287" height="51" alt="image" src="https://github.com/user-attachments/assets/276708b9-1463-4c7c-8ef9-6a88a5c6094e" />

And set this kubeconfig as an enviromental variable

```bash
export KUBECONFIG=/path/to/linode-kubeconfig.yaml
```
<img width="1292" height="55" alt="image" src="https://github.com/user-attachments/assets/9517e182-9c05-4049-892e-8208ade3181a" />

```bash
kubectl get nodes
```
<img width="1295" height="122" alt="image" src="https://github.com/user-attachments/assets/0f61e694-5755-4f6c-832e-f5510b29c79b" />
All nodes in Ready state.

---

## 🛠️ Deploy MongoDB statefulSet using Helm

Add the Bitnami Helm repo and update it:

<img width="1243" height="325" alt="image" src="https://github.com/user-attachments/assets/a404acf2-eb6d-4c74-a431-5fdef4bbf914" />

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Inspect the chart and customize your values:

And then search for mongoDB ‎
```bash
helm search repo bitnami/mongodb 
```
and see the parameters that chart provide‎ to add in > my-values.yaml
<img width="1218" height="635" alt="image" src="https://github.com/user-attachments/assets/ea335c75-1fa7-4d9e-856f-3054f7bc8d99" />

and we create value file => helm-mongodb-values.yaml
```

Install MongoDB using those override values:
```bash
helm install mongodb --values helm-mongodb-values.yaml bitnami/mongodb
```
<img width="1296" height="560" alt="image" src="https://github.com/user-attachments/assets/546f5c4b-5504-4f80-be1a-82831825e484" />

Check deployment and service status:
```bash
kubectl get all
```
<img width="1303" height="344" alt="image" src="https://github.com/user-attachments/assets/8b698cab-7fd2-4d64-b401-e0ed60560318" />
<img width="1289" height="227" alt="image" src="https://github.com/user-attachments/assets/7a7bf17a-6049-49a9-9017-be4ee0ed76ec" />

<img width="1357" height="469" alt="image" src="https://github.com/user-attachments/assets/72630906-a7fd-49c6-86bd-ef19ce58d101" />

now I have pv that dynamically created

---

## 💻 Deploy Mongo Express UI

Install Mongo Express and connect it to your MongoDB service:

Now I will create mongo express Deployment and service ‎=> mongo-express.yaml

Then apply mongo-express file

```bash
kubectl apply -f mongo-express.yaml
```
<img width="1287" height="97" alt="image" src="https://github.com/user-attachments/assets/9123f207-252c-40e3-86e7-ac38993132b5" />

Now I have Mongo express pod 
<img width="1294" height="167" alt="image" src="https://github.com/user-attachments/assets/9592380a-4644-4c0f-bd25-6aac2c7539e2" />

---

## 🌐 Expose the UI via Ingress

Install an NGINX Ingress controller as Loadbalancer (using Helm Chart)‎:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true
```
<img width="1299" height="29" alt="image" src="https://github.com/user-attachments/assets/b684aad6-4a23-43f8-a374-ff2e8b913ea1" />
<img width="1291" height="558" alt="image" src="https://github.com/user-attachments/assets/e64a24b7-eb41-4efa-a387-a18879306eaf" />

Now ingress controller was deployed
<img width="1277" height="186" alt="image" src="https://github.com/user-attachments/assets/09b4a28d-578d-496d-aaf0-78a176d89a6d" />
And also nginx ingress service was created ‎
<img width="1298" height="358" alt="image" src="https://github.com/user-attachments/assets/fd540bdc-7451-409d-8e0e-6ebeb809d275" />
<img width="1357" height="331" alt="image" src="https://github.com/user-attachments/assets/142e0967-d1f6-4550-9b83-8e1fb9a60475" />

---

Create an Ingress rule for mongo express (ingress.yaml):

Apply it:
```bash
kubectl apply -f mongo-express-ingress.yaml
```
<img width="1299" height="99" alt="image" src="https://github.com/user-attachments/assets/9a0503e4-77ae-4c9b-a0e5-442799659e52" />
(add host to ingress rules)
<img width="1359" height="337" alt="image" src="https://github.com/user-attachments/assets/ebae795b-764a-4593-afe9-f2045eb7b0a3" />


```bash
kubectl get ingress 
```
<img width="1307" height="96" alt="image" src="https://github.com/user-attachments/assets/dd378708-f4d3-4e3e-a5bf-798a96d56d4d" />


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
DevOps & Mechanical Engineer | Kubernetes | Helm | CI/CD  
---

