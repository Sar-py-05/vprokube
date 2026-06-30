# vProfile Application Deployment on Kubernetes using kOps on AWS

## Project Overview

This project demonstrates the deployment of the **vProfile Java Enterprise Application** on a production-grade **Kubernetes cluster** provisioned on **Amazon Web Services (AWS)** using **kOps**.

The objective of this project was to gain hands-on experience in provisioning a Kubernetes cluster on AWS, deploying a multi-tier application using Kubernetes manifests, configuring persistent storage, exposing the application through an NGINX Ingress Controller, integrating DNS using Route53, and troubleshooting real-world deployment issues encountered during the implementation.

Unlike local Kubernetes environments such as Minikube or Kind, this project provisions actual AWS infrastructure consisting of EC2 instances, Elastic Block Storage (EBS), Route53, Elastic Load Balancer (ELB), and Amazon S3 for the kOps state store. This closely resembles the architecture used in enterprise production environments.

The deployed application consists of multiple interconnected services:

- Java Web Application (vProfile)
- MySQL Database
- RabbitMQ Message Broker
- Memcached Caching Layer

Each component is deployed as an independent Kubernetes Deployment and exposed internally through Kubernetes Services. The application is made accessible externally through an NGINX Ingress Controller backed by an AWS Classic Load Balancer.

By completing this project, I gained practical experience with Kubernetes administration, cluster provisioning, application deployment, service networking, persistent storage, ingress configuration, DNS integration, and production troubleshooting.

---

# Project Objectives

The primary objectives of this project were to:

- Provision a Kubernetes cluster on AWS using kOps.
- Configure kubectl to communicate with the newly created cluster.
- Deploy the NGINX Ingress Controller.
- Deploy MySQL using Persistent Volume Claims.
- Deploy RabbitMQ and Memcached.
- Deploy the vProfile Java web application.
- Configure Kubernetes Services for internal communication.
- Configure Kubernetes Ingress for external access.
- Integrate Route53 DNS with the Ingress Load Balancer.
- Validate the complete deployment.
- Troubleshoot and resolve deployment issues.
- Synchronize the final project with GitHub using SSH authentication.

---

# Project Architecture

The application follows a standard three-tier architecture deployed on Kubernetes.

```

```
                        Internet
                            │
                            │
                   Route53 DNS Record
                            │
                            │
                 AWS Elastic Load Balancer
                            │
                            │
               NGINX Ingress Controller
                            │
                            │
                  Kubernetes Ingress
                            │
                            │
                vproapp-service (ClusterIP)
                            │
                            │
                  vProfile Application Pod
                    │         │         │
                    │         │         │
                    ▼         ▼         ▼
               MySQL      RabbitMQ   Memcached
                 │
                 ▼
        Persistent Volume Claim
                 │
                 ▼
             Amazon EBS Volume

```

The Kubernetes cluster itself is provisioned using **kOps**, which automates the creation and management of AWS infrastructure required for Kubernetes.

---

# Technology Stack

This project uses the following technologies.

### Cloud Platform

- Amazon Web Services (AWS)

### Kubernetes

- Kubernetes
- kOps
- kubectl

### Networking

- Route53
- AWS Classic Load Balancer
- NGINX Ingress Controller

### Storage

- Amazon EBS
- Persistent Volume Claim (PVC)

### Application Components

- Java (vProfile)
- MySQL
- RabbitMQ
- Memcached

### Source Control

- Git
- GitHub
- SSH Authentication

### Operating System

- Ubuntu Linux

---

# AWS Services Used

The following AWS services were provisioned during this project.

| Service | Purpose |
|----------|---------|
| EC2 | Kubernetes Control Plane and Worker Nodes |
| VPC | Network Isolation |
| Internet Gateway | Internet Connectivity |
| Route Tables | Routing Configuration |
| Security Groups | Firewall Rules |
| Route53 | DNS Management |
| Elastic Load Balancer | External Access |
| Amazon EBS | Persistent Storage |
| Amazon S3 | kOps State Store |
| IAM | Authentication and Authorization |

---

# Kubernetes Components

The following Kubernetes resources were created during this deployment.

## Deployments

- vproapp
- vprodb
- vpromc
- vprormq

## Services

- vproapp-service
- vprodb
- vprocache01
- vprormq01

## Storage

- Persistent Volume Claim
- Dynamic EBS Volume

## Networking

- Ingress
- NGINX Ingress Controller

## Security

- Kubernetes Secret

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

The `kubedefs` directory contains all Kubernetes resource definitions required for deploying the complete application stack.

The repository also contains detailed project documentation covering the architecture, deployment procedure, troubleshooting steps, operational runbook, and lessons learned throughout the implementation.

---

# Deployment Workflow

The deployment followed the sequence shown below.

1. Provision AWS infrastructure using kOps.
2. Export the Kubernetes configuration.
3. Validate the cluster.
4. Install the NGINX Ingress Controller.
5. Create the Persistent Volume Claim.
6. Create Kubernetes Secrets.
7. Deploy MySQL.
8. Deploy RabbitMQ.
9. Deploy Memcached.
10. Deploy the vProfile application.
11. Create Kubernetes Services.
12. Create the Kubernetes Ingress.
13. Configure Route53 DNS.
14. Validate the application deployment.
15. Troubleshoot deployment issues.
16. Synchronize the project repository with GitHub.

Each deployment stage was validated before proceeding to the next, ensuring that dependencies such as the database, message broker, and cache were available before the application container started.
