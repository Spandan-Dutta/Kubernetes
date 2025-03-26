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

## 📁 Project Structure

├── manifests/
│   ├── voting-app-deployment.yaml
│   ├── voting-app-service.yaml
│   ├── redis-deployment.yaml
│   ├── redis-service.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── secrets.yaml
├── README.md
├── .gitignore
└── LICENSE
