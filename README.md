# Kubernetes Deployment for Voting App 🚀

![Kubernetes](https://img.shields.io/badge/Kubernetes-318CE7?logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![CI/CD](https://img.shields.io/badge/CI/CD-GitHub_Actions-blue?logo=githubactions)


This repository contains the Kubernetes configurations for deploying a **Voting Application** on a Minikube or cloud-based cluster. The application consists of:
- ✅ **Frontend:** Voting UI (Python), Result UI (NodeJS)
- ✅ **Backend:** Vote Processor (.NET Application)
- ✅ **Database:** Redis & PostgreSQL
- ✅ **Orchestration:** Kubernetes

The setup includes **Pods, Deployments, Services 🛠️

## 🔧 Prerequisites

Ensure you have the following installed:
- [Minikube](https://minikube.sigs.k8s.io/docs/start/) 🚀
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 🛠️
- [Docker](https://docs.docker.com/get-docker/) 🐳

## 🚀 Installation & Deployment

### 1️⃣ Start Minikube
```sh
minikube start
kubectl apply -f manifests/
kubectl get pods
kubectl get svc
minikube ip
```


---

### ✅ **6. Architecture Diagram**  
Use **diagrams.net** or **Mermaid.js** to create a **visual representation** of your Kubernetes architecture.  

**Example using Mermaid.js**  
```md
## 🏗️ Architecture Diagram
```mermaid
graph TD;
    User -->|Access| Ingress;
    Ingress -->|Routes Traffic| Service;
    Service -->|Forwards to| Pod;
    Pod -->|Reads/Writes| Database;


