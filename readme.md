# vProfile Application Deployment on Kubernetes using kOps on AWS

---

# Project Overview

This project demonstrates the complete deployment of the **vProfile Java Enterprise Application** on a **production-style Kubernetes cluster** provisioned using **kOps** on **Amazon Web Services (AWS)**.

Unlike local Kubernetes environments such as Minikube or Kind, this project provisions a highly available Kubernetes cluster on AWS infrastructure and deploys a multi-tier Java application using native Kubernetes resources.

The deployment includes:

- Kubernetes Cluster Provisioned using kOps
- NGINX Ingress Controller
- Multi-tier Java Web Application
- MySQL Database with Persistent Storage
- RabbitMQ Message Broker
- Memcached Caching Layer
- Kubernetes Services
- Persistent Volume Claims
- Kubernetes Secrets
- DNS Integration
- AWS Elastic Load Balancer

This project closely resembles how enterprise applications are deployed in production Kubernetes environments.

---

# Project Objectives

The primary objectives of this project were:

- Provision Kubernetes Cluster on AWS using kOps
- Configure kubectl access
- Deploy NGINX Ingress Controller
- Deploy MySQL Database
- Configure Persistent Storage
- Deploy RabbitMQ
- Deploy Memcached
- Deploy Java Web Application
- Configure Kubernetes Services
- Configure Kubernetes Ingress
- Configure DNS Mapping
- Validate Complete Application Deployment
- Practice Kubernetes Troubleshooting
- Learn Production Deployment Workflow

---

# Architecture Overview

The architecture consists of four major layers.

## Infrastructure Layer

- AWS EC2 Instances
- Amazon Route53
- AWS Elastic Load Balancer
- AWS S3 (kOps State Store)
- Amazon EBS Volumes

---

## Kubernetes Layer

- Kubernetes Control Plane
- Worker Nodes
- Pods
- Deployments
- Services
- Persistent Volume Claims
- Secrets
- Ingress Controller

---

## Application Layer

- vProfile Application
- MySQL Database
- RabbitMQ
- Memcached

---

## Client Layer

User

↓

DNS

↓

AWS Load Balancer

↓

NGINX Ingress

↓

vProfile Service

↓

vProfile Pod

↓

MySQL

RabbitMQ

Memcached

---

# Technology Stack

| Category | Technology |
|----------|------------|
| Cloud Provider | AWS |
| Kubernetes Provisioning | kOps |
| Container Runtime | containerd |
| Orchestration | Kubernetes |
| Ingress | NGINX Ingress Controller |
| Application | Java (vProfile) |
| Database | MySQL |
| Cache | Memcached |
| Messaging | RabbitMQ |
| Storage | Amazon EBS |
| DNS | Route53 |
| Version Control | Git |
| Repository | GitHub |
| OS | Ubuntu Linux |
| CLI Tools | kubectl, kops |

---

# Kubernetes Resources Created

The following Kubernetes resources were created during this project.

### Deployments

- vproapp
- vprodb
- vpromc
- vprormq

---

### Services

- vproapp-service
- vprodb
- vprocache01
- vprormq01

---

### Ingress

- vpro-ingress

---

### Persistent Volume Claim

- db-pv-claim

---

### Secret

- app-secret

---

### Namespace

- default
- ingress-nginx
- kube-system
- kube-public
- kube-node-lease

---

# AWS Resources Used

The deployment utilized the following AWS services.

- EC2
- VPC
- Internet Gateway
- Route Tables
- Security Groups
- Route53
- Elastic Load Balancer
- Amazon EBS
- Amazon S3
- IAM

---

# Deployment Workflow

The deployment followed the sequence below.

```
Provision AWS Infrastructure
            │
            ▼
Create Kubernetes Cluster using kOps
            │
            ▼
Export kubectl Configuration
            │
            ▼
Validate Cluster
            │
            ▼
Install NGINX Ingress Controller
            │
            ▼
Create Persistent Volume Claim
            │
            ▼
Create Kubernetes Secret
            │
            ▼
Deploy Database
            │
            ▼
Deploy RabbitMQ
            │
            ▼
Deploy Memcached
            │
            ▼
Deploy Application
            │
            ▼
Create Kubernetes Services
            │
            ▼
Create Kubernetes Ingress
            │
            ▼
Configure DNS
            │
            ▼
Application Available
```

---

# Major Learning Outcomes

This project provided practical experience in:

- Kubernetes Architecture
- Cluster Provisioning using kOps
- kubectl Administration
- Deployments
- Services
- Ingress
- Persistent Volumes
- Persistent Volume Claims
- Kubernetes Secrets
- Init Containers
- Service Discovery
- DNS Configuration
- Storage Management
- AWS Load Balancer Integration
- YAML Debugging
- Kubernetes Troubleshooting
- Git Repository Synchronization
- Git Rebase Conflict Resolution
- SSH Authentication with GitHub

---

# Challenges Faced

Several real-world issues were encountered and resolved during this project.

- Forgot to create Kubernetes cluster before deploying Ingress
- kubectl connection refused
- YAML indentation errors
- Incorrect Service definitions
- Invalid Service field placement
- Ingress YAML parsing errors
- Deployment YAML syntax issues
- Init Container waiting indefinitely
- RabbitMQ service naming mismatch
- OOMKilled due to insufficient memory allocation
- Git merge conflicts during rebase
- GitHub HTTPS authentication issue
- Migrated repository authentication to SSH

These issues have been documented in detail within the project documentation.

---

# Project Validation

The deployment was successfully validated.

Validation included:

- Kubernetes Cluster Ready
- Worker Nodes Healthy
- Control Plane Healthy
- Ingress Controller Running
- PVC Bound Successfully
- Database Running
- RabbitMQ Running
- Memcached Running
- Application Running
- Services Accessible
- Ingress Created
- DNS Working
- Application Accessible through Browser

Application URL:

```
http://vprofile.hkhsinfo.shop/welcome
```

---

# Repository Structure

```
vprokube/

├── Docker-files/
├── docker-compose/
├── kubedefs/
│   ├── appdeploy.yaml
│   ├── appingress.yaml
│   ├── appservice.yaml
│   ├── dbdeploy.yaml
│   ├── dbpvc.yaml
│   ├── dbservice.yaml
│   ├── mcdep.yaml
│   ├── mcservice.yaml
│   ├── rmqdeploy.yaml
│   ├── rmqservice.yaml
│   └── secret.yaml
│
├── README.md
├── architecture.md
├── deployment-guide.md
├── runbook.md
├── troubleshooting-guide.md
└── lessons-learned.md
```

---

# Documentation

This repository contains detailed documentation for every stage of the project.

| Document | Description |
|----------|-------------|
| README.md | Project Overview |
| architecture.md | Complete Architecture Explanation |
| deployment-guide.md | Step-by-Step Deployment Guide |
| runbook.md | Operational Procedures |
| troubleshooting-guide.md | Errors Encountered and Fixes |
| lessons-learned.md | Knowledge Gained Throughout the Project |

---

# Skills Demonstrated

- AWS
- Kubernetes
- kOps
- Linux Administration
- Networking
- DNS
- Persistent Storage
- Container Orchestration
- Kubernetes Services
- Ingress Controller
- YAML Authoring
- Git
- GitHub
- SSH Authentication
- Production Troubleshooting
- Infrastructure Provisioning

---

# Future Improvements

Possible enhancements include:

- Helm Charts
- Horizontal Pod Autoscaler
- Metrics Server
- Prometheus Monitoring
- Grafana Dashboards
- ArgoCD
- GitOps Workflow
- TLS Certificates using cert-manager
- ExternalDNS
- AWS ALB Ingress Controller
- CI/CD Pipeline Integration
- Automated Backup Strategy
- Multi-AZ Cluster Deployment
- High Availability Database

---

# Author

**Abhishek Roy**

DevOps | Cloud | Kubernetes | AWS | CI/CD | Linux

This repository documents a complete hands-on deployment of a production-style Kubernetes application on AWS using kOps while following industry-standard deployment, troubleshooting, validation, and documentation practices.
