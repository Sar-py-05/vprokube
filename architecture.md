# Project Architecture

## Architecture Overview

This project demonstrates the deployment of the **vProfile Java Enterprise Application** on a production-style Kubernetes cluster provisioned on **Amazon Web Services (AWS)** using **kOps**.

The infrastructure consists of a Kubernetes control plane, multiple worker nodes, networking components, persistent storage, ingress routing, and a multi-tier Java application. The cluster is created using kOps, which provisions all required AWS resources automatically.

The application follows a standard three-tier architecture consisting of a presentation layer, middleware services, and a database layer. External traffic enters through an AWS Elastic Load Balancer, passes through the NGINX Ingress Controller, and is routed to the application service. The application communicates internally with RabbitMQ, Memcached, and MySQL using Kubernetes ClusterIP Services.

---

# High-Level Architecture

```

```
                               Internet
                                   │
                                   │
                     Route53 DNS (vprofile.hkhsinfo.shop)
                                   │
                                   │
                    AWS Elastic Load Balancer (ELB)
                                   │
                                   │
                    NGINX Ingress Controller Service
                                   │
                                   │
                         Kubernetes Ingress Resource
                                   │
                                   │
                        vproapp-service (ClusterIP)
                                   │
                                   │
                          vProfile Application Pod
                   ┌───────────────┼────────────────┐
                   │               │                │
                   │               │                │
                   ▼               ▼                ▼
             MySQL Service   RabbitMQ Service   Memcached Service
                   │               │                │
                   ▼
              MySQL Pod
                   │
                   ▼
          Persistent Volume Claim
                   │
                   ▼
              Amazon EBS Volume

```

---

# Infrastructure Architecture

The Kubernetes cluster is provisioned using **kOps**, which automates the creation and configuration of AWS infrastructure.

The infrastructure consists of:

- One Kubernetes Control Plane node
- Two Kubernetes Worker nodes
- Amazon VPC
- Public Subnet
- Internet Gateway
- Route Tables
- Security Groups
- Route53 Hosted Zone
- Elastic Load Balancer
- Amazon S3 State Store
- Amazon EBS Persistent Storage

During deployment, kOps manages the lifecycle of these resources automatically.

---

# Kubernetes Cluster Architecture

The Kubernetes cluster contains one Control Plane node and two Worker nodes.

```

```
                   Kubernetes Cluster

               +-------------------------+
               |     Control Plane       |
               |-------------------------|
               | API Server              |
               | Scheduler               |
               | Controller Manager      |
               | etcd                    |
               +-------------------------+

                    /              \

                   /                \

          +---------------+   +---------------+
          | Worker Node 1 |   | Worker Node 2 |
          +---------------+   +---------------+
          | vProfile Pod  |   | RabbitMQ Pod  |
          | MySQL Pod     |   | Memcached Pod |
          +---------------+   +---------------+

```

The Control Plane manages scheduling, cluster state, networking, and API communication, while the Worker nodes run the application workloads.

---

# Application Architecture

The deployed application consists of four primary components.

## vProfile Application

The Java web application serves user requests and communicates with backend services.

Responsibilities:

- Web interface
- Business logic
- Database access
- Message queue communication
- Cache access

---

## MySQL Database

The MySQL database stores all persistent application data.

Responsibilities:

- User data
- Application metadata
- Persistent storage

The database stores its data on an Amazon EBS-backed Persistent Volume.

---

## RabbitMQ

RabbitMQ provides asynchronous messaging between application components.

Responsibilities:

- Message queuing
- Event delivery
- Reliable communication

---

## Memcached

Memcached improves application performance by caching frequently accessed data.

Responsibilities:

- Reduce database load
- Faster response time
- Temporary object storage

---

# Kubernetes Resources

The deployment consists of the following Kubernetes resources.

| Resource | Purpose |
|-----------|---------|
| Deployment | Creates application Pods |
| Service | Internal networking |
| Ingress | External HTTP routing |
| Secret | Stores sensitive data |
| PersistentVolumeClaim | Persistent storage |
| Namespace | Resource isolation |

---

# Networking Flow

Client requests follow the path shown below.

```

```
Browser

↓

Route53

↓

AWS Load Balancer

↓

NGINX Ingress Controller

↓

Ingress Resource

↓

vproapp-service

↓

vProfile Pod

↓

Database / RabbitMQ / Memcached

```

Each component communicates internally using Kubernetes ClusterIP Services.

---

# Storage Architecture

Persistent storage is provided using Kubernetes Persistent Volume Claims backed by Amazon EBS.

```

```
Application

↓

MySQL Pod

↓

Persistent Volume Claim

↓

Persistent Volume

↓

Amazon EBS

```

This ensures database data survives Pod recreation and node failures.

---

# DNS Architecture

The application is exposed through Route53.

```

```
vprofile.hkhsinfo.shop

↓

AWS Load Balancer

↓

NGINX Ingress

↓

Application Service

↓

Application Pod

```

Users access the application using the DNS name instead of the Load Balancer hostname.

---

# Security Architecture

The deployment includes several security components.

## Kubernetes Secrets

Sensitive information such as the MySQL root password is stored using Kubernetes Secrets rather than hardcoding credentials into manifests.

---

## Cluster Networking

Internal application communication uses ClusterIP Services, preventing direct external access to backend services.

Only the Ingress Controller is exposed externally.

---

## AWS Security Groups

Security Groups created by kOps control inbound and outbound traffic to Kubernetes nodes.

---

# Pod Startup Dependency Flow

The application Pod uses Init Containers to ensure backend services are available before the application starts.

```

```
Create Application Pod

↓

Init Container

↓

Wait for MySQL Service

↓

Success

↓

Wait for RabbitMQ Service

↓

Success

↓

Start Application Container

```

This prevents application startup failures caused by unavailable dependencies.

---

# AWS Resources Provisioned by kOps

During cluster creation, kOps automatically provisions the following AWS resources.

- EC2 Instances
- Auto Scaling Groups
- IAM Roles
- IAM Instance Profiles
- Elastic Load Balancers
- Route53 DNS Records
- Security Groups
- Launch Templates
- Amazon EBS Volumes
- Amazon S3 State Store
- Kubernetes Control Plane
- Worker Nodes

---

# Project Architecture Summary

This project demonstrates a production-style Kubernetes deployment using AWS infrastructure and kOps.

The completed architecture includes:

- Production Kubernetes Cluster
- Multi-node Deployment
- Java Enterprise Application
- MySQL Database
- RabbitMQ Messaging
- Memcached Cache
- Persistent Storage
- Kubernetes Secrets
- Kubernetes Services
- NGINX Ingress Controller
- Route53 DNS Integration
- AWS Elastic Load Balancer
- Amazon EBS Persistent Storage

The architecture closely resembles a real-world enterprise Kubernetes deployment and provides hands-on experience with cluster provisioning, workload deployment, networking, storage, ingress management, and production troubleshooting.
