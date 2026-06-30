# Deployment Guide

## Project Overview

This document provides a complete step-by-step guide for deploying the **vProfile Java Enterprise Application** on a Kubernetes cluster provisioned on **Amazon Web Services (AWS)** using **kOps**.

The deployment covers the complete lifecycle of the project, beginning with Kubernetes cluster provisioning and ending with successful application deployment using Kubernetes manifests. Throughout the implementation, several deployment issues were encountered and resolved, making this guide not only a deployment manual but also a practical troubleshooting reference.

The application consists of multiple interconnected services:

- vProfile Java Application
- MySQL Database
- RabbitMQ Message Broker
- Memcached Cache

Each component is deployed independently using Kubernetes Deployments and exposed internally through ClusterIP Services. External client traffic is routed through an AWS Elastic Load Balancer and the NGINX Ingress Controller.

---

# Deployment Workflow

The overall deployment followed the sequence below.

```
AWS Infrastructure
        │
        ▼
Create Kubernetes Cluster using kOps
        │
        ▼
Configure kubectl
        │
        ▼
Validate Cluster
        │
        ▼
Install NGINX Ingress Controller
        │
        ▼
Deploy Persistent Storage
        │
        ▼
Deploy Kubernetes Secret
        │
        ▼
Deploy MySQL
        │
        ▼
Deploy RabbitMQ
        │
        ▼
Deploy Memcached
        │
        ▼
Deploy vProfile Application
        │
        ▼
Create Kubernetes Services
        │
        ▼
Create Kubernetes Ingress
        │
        ▼
Configure Route53 DNS
        │
        ▼
Validate Application
        │
        ▼
Synchronize GitHub Repository
```

---

# Prerequisites

Before beginning the deployment, ensure the following prerequisites are available.

## AWS Resources

- AWS Account
- IAM User with Administrator Access
- Route53 Hosted Zone
- EC2 Key Pair
- Amazon S3 Bucket for kOps State Store

---

## Local / EC2 Software

- Ubuntu Linux
- Git
- kubectl
- kOps
- AWS CLI
- SSH Keys

---

## Project Repository

Clone the project repository.

```bash
git clone https://github.com/Sar-py-05/vprokube.git
cd vprokube
```

Verify the configured Git remote.

```bash
git remote -v
```

Expected Output

```text
origin  https://github.com/Sar-py-05/vprokube.git (fetch)
origin  https://github.com/Sar-py-05/vprokube.git (push)
```

Purpose

- Confirms the repository has been cloned correctly.
- Verifies the configured remote repository before making changes.

---

# Step 1 – Create the Kubernetes Cluster

The Kubernetes cluster was provisioned using **kOps**.

Command

```bash
kops create cluster \
--name=kubepro.hkhsinfo.shop \
--state=s3://kopsstate0584 \
--zones=us-east-1a \
--node-count=2 \
--node-size=t3.small \
--control-plane-size=t3.small \
--dns-zone=kubepro.hkhsinfo.shop
```

Explanation

| Parameter | Description |
|-----------|-------------|
| --name | Kubernetes Cluster Name |
| --state | Amazon S3 bucket storing cluster state |
| --zones | AWS Availability Zone |
| --node-count | Number of Worker Nodes |
| --node-size | Worker Node EC2 Instance Type |
| --control-plane-size | Control Plane Instance Type |
| --dns-zone | Route53 Hosted Zone |

This command generates the cluster configuration but does not provision AWS resources.

---

# Step 2 – Provision AWS Infrastructure

Apply the cluster configuration.

```bash
kops update cluster \
--name kubepro.hkhsinfo.shop \
--yes
```

Purpose

This command creates the required AWS resources, including:

- EC2 Instances
- Security Groups
- Auto Scaling Groups
- Elastic Load Balancer
- Route53 Records
- IAM Roles
- Networking Components

Expected Output

```text
Cluster is starting.
It should be ready in a few minutes.
```

---

# Step 3 – Configure kOps State Store

Export the S3 bucket used by kOps.

```bash
export KOPS_STATE_STORE=s3://kopsstate0584
```

Purpose

Allows future kOps commands to locate the cluster configuration stored in Amazon S3.

---

# Step 4 – Configure kubectl

Export the Kubernetes configuration.

```bash
kops export kubecfg \
--admin \
--name kubepro.hkhsinfo.shop
```

Alternative command

```bash
kops export kubecfg \
--name kubepro.hkhsinfo.shop \
--admin
```

Expected Output

```text
kOps has set your kubectl context to kubepro.hkhsinfo.shop
```

Purpose

This command configures kubectl to communicate with the newly created Kubernetes cluster.

---

# Step 5 – Validate the Cluster

Verify that the cluster has been created successfully.

```bash
kops validate cluster
```

Expected Output

```text
Your cluster kubepro.hkhsinfo.shop is ready
```

Validation

Check Kubernetes Nodes.

```bash
kubectl get nodes
```

Expected Output

```text
NAME                      STATUS   ROLES
control-plane             Ready    control-plane
worker-node-1             Ready    node
worker-node-2             Ready    node
```

Purpose

Ensures all Kubernetes nodes are healthy and ready before deploying workloads.

---

# Step 6 – Install the NGINX Ingress Controller

Initially, the Ingress Controller installation was attempted before the Kubernetes cluster had been created.

Command Executed

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```

Error Received

```text
failed to download openapi

dial tcp 127.0.0.1:8080

connection refused
```

Reason

kubectl was unable to connect to a Kubernetes API Server because the cluster did not yet exist.

Resolution

The Kubernetes cluster was created using kOps, the kubeconfig was exported, and the Ingress Controller installation command was executed again.

Successful Installation

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```

Resources Created

- Namespace
- Service Accounts
- RBAC Roles
- Cluster Roles
- Role Bindings
- Cluster Role Bindings
- ConfigMap
- Service
- Deployment
- Admission Jobs
- IngressClass
- Validating Webhook

---

# Step 7 – Verify Ingress Controller

Verify the namespace.

```bash
kubectl get ns
```

Verify the controller Pods.

```bash
kubectl get pods --namespace ingress-nginx
```

Expected Output

```text
ingress-nginx-controller   Running
```

Purpose

Confirms that the Ingress Controller has been deployed successfully.

---

# Step 8 – Deploy Persistent Storage

Navigate to the Kubernetes manifest directory.

```bash
cd ~/vprokube/kubedefs
```

Review the Persistent Volume Claim.

```bash
cat dbpvc.yaml
```

Create the PVC.

```bash
kubectl create -f dbpvc.yaml
```

Expected Output

```text
persistentvolumeclaim/db-pv-claim created
```

Purpose

The Persistent Volume Claim requests a 3 GiB persistent volume that will later be dynamically provisioned as an Amazon EBS volume for the MySQL database.

Verify the PVC.

```bash
kubectl get pvc
```

Expected Output

```text
db-pv-claim   Bound
```

---

# Step 9 – Deploy All Kubernetes Resources

Attempt to deploy all manifests.

```bash
kubectl create -f .
```

Several resources were created successfully.

- vProfile Deployment
- RabbitMQ Deployment
- Memcached Deployment
- RabbitMQ Service
- Kubernetes Secret

However, the deployment also produced multiple validation and YAML parsing errors that prevented some resources from being created.

Examples included:

- Invalid Service definitions
- Incorrect YAML indentation
- Incorrect placement of the `type` field
- Invalid Ingress YAML syntax
- Deployment manifest formatting issues

Rather than continuing with partially created resources, each manifest was validated and corrected individually before being redeployed.

This incremental approach significantly simplified troubleshooting and ensured each component was functioning correctly before moving to the next deployment stage.

---

# Step 10 – Begin Manifest Validation

The following commands were used extensively during troubleshooting.

List manifest files.

```bash
ls
```

Display Service definitions.

```bash
cat appservice.yaml
cat dbservice.yaml
cat mcservice.yaml
```

Search for invalid fields.

```bash
grep -n "type:" *.yaml
```

Display line numbers for easier debugging.

```bash
cat -n dbdeploy.yaml
```

Print a specific section of a file.

```bash
sed -n '1,80p' dbdeploy.yaml
```

Edit manifests.

```bash
vim dbdeploy.yaml
```

Before applying any corrected manifest, validate it using the client-side dry run.

```bash
kubectl apply --dry-run=client -f <manifest-file>
```

This command became one of the most valuable troubleshooting techniques during the project because it validates YAML syntax and Kubernetes object structure without creating resources in the cluster.

---

**End of Part 1**

The next section begins with correcting all manifest files (`dbdeploy.yaml`, `appservice.yaml`, `dbservice.yaml`, `mcservice.yaml`, `appingress.yaml`, and `appdeploy.yaml`) and continues through application deployment, debugging, GitHub synchronization, cleanup, and final validation.
