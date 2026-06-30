# Runbook

# Kubernetes on AWS using kOps
## Operational Runbook

---

## Project

Deploy the vProfile Java application on a Kubernetes cluster provisioned with kOps on AWS using:

- EC2
- Route53
- S3 (kOps State Store)
- EBS Persistent Volumes
- NGINX Ingress Controller
- Kubernetes Deployments
- Services
- PVC
- Ingress

---

# Environment Information

AWS Region

```
us-east-1
```

Availability Zone

```
us-east-1a
```

Cluster Name

```
kubepro.hkhsinfo.shop
```

State Store

```
s3://kopsstate0584
```

DNS

```
Route53 Hosted Zone
```

Repository

```
https://github.com/Sar-py-05/vprokube
```

---

# Verify Cluster

Check Nodes

```bash
kubectl get nodes
```

Expected

```
Ready
Ready
Ready
```

---

Validate Cluster

```bash
kops validate cluster
```

Expected

```
Your cluster kubepro.hkhsinfo.shop is ready
```

---

Check Namespace

```bash
kubectl get ns
```

---

Check Pods

```bash
kubectl get pods
```

---

Watch Pods

```bash
kubectl get pods -w
```

---

Describe Pod

```bash
kubectl describe pod <pod-name>
```

Example

```bash
kubectl describe pod vproapp-84ffc5c4ff-gr52r
```

---

View Logs

```bash
kubectl logs <pod-name>
```

Example

```bash
kubectl logs vproapp-84ffc5c4ff-gr52r
```

---

Logs from Init Container

List Init Containers

```bash
kubectl get pod <pod> -o jsonpath='{.spec.initContainers[*].name}'
```

View Logs

```bash
kubectl logs <pod> -c <init-container-name>
```

---

# Deploy Resources

Database PVC

```bash
kubectl apply -f dbpvc.yaml
```

Deploy Everything

```bash
kubectl apply -f .
```

Deploy Individual Files

```bash
kubectl apply -f dbdeploy.yaml
kubectl apply -f dbservice.yaml
kubectl apply -f rmqdeploy.yaml
kubectl apply -f rmqservice.yaml
kubectl apply -f mcdep.yaml
kubectl apply -f mcservice.yaml
kubectl apply -f appservice.yaml
kubectl apply -f appdeploy.yaml
kubectl apply -f appingress.yaml
```

---

# Validate YAML

Dry Run

```bash
kubectl apply --dry-run=client -f filename.yaml
```

Example

```bash
kubectl apply --dry-run=client -f appdeploy.yaml
```

---

# Restart Application

```bash
kubectl rollout restart deployment vproapp
```

---

Check Rollout

```bash
kubectl rollout status deployment vproapp
```

---

Deployment Status

```bash
kubectl get deployments
```

---

ReplicaSets

```bash
kubectl get rs
```

---

Describe Deployment

```bash
kubectl describe deployment vproapp
```

---

# Service Operations

List Services

```bash
kubectl get svc
```

Describe Service

```bash
kubectl describe svc vproapp-service
```

Describe RabbitMQ

```bash
kubectl describe svc vprormq01
```

---

# Storage

List PVC

```bash
kubectl get pvc
```

Storage Classes

```bash
kubectl get sc
```

---

# Ingress

List Ingress

```bash
kubectl get ingress
```

Describe Ingress

```bash
kubectl describe ingress vpro-ingress
```

Expected Host

```
vprofile.hkhsinfo.shop
```

Application URL

```
http://vprofile.hkhsinfo.shop/welcome
```

---

# NGINX Ingress Controller

Install

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```

Verify

```bash
kubectl get pods -n ingress-nginx
```

Delete

```bash
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```

---

# Git Operations

Repository Status

```bash
git status
```

View Remote

```bash
git remote -v
```

Stage Changes

```bash
git add .
```

Commit

```bash
git commit -m "message"
```

Push

```bash
git push origin main
```

Pull Latest

```bash
git pull origin main
```

---

# kOps Operations

Create Cluster

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

Update Cluster

```bash
kops update cluster --name kubepro.hkhsinfo.shop --yes
```

Export kubeconfig

```bash
export KOPS_STATE_STORE=s3://kopsstate0584

kops export kubecfg \
--name kubepro.hkhsinfo.shop \
--admin
```

Validate

```bash
kops validate cluster
```

Delete Cluster

```bash
kops delete cluster --name kubepro.hkhsinfo.shop --yes
```

---

# Health Checks

Cluster

```bash
kubectl get nodes
```

Pods

```bash
kubectl get pods
```

Services

```bash
kubectl get svc
```

Ingress

```bash
kubectl get ingress
```

PVC

```bash
kubectl get pvc
```

Storage Classes

```bash
kubectl get sc
```

---

# Expected Final State

✔ Control Plane Ready

✔ Worker Nodes Ready

✔ PVC Bound

✔ Database Running

✔ RabbitMQ Running

✔ Memcached Running

✔ Application Running

✔ Ingress Created

✔ DNS Working

✔ Application Accessible

```
http://vprofile.hkhsinfo.shop/welcome
```

---

# Project Cleanup

Delete Kubernetes Resources

```bash
kubectl delete -f .
```

Delete Ingress Controller

```bash
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```

Delete Cluster

```bash
kops delete cluster --name kubepro.hkhsinfo.shop --yes
```

---

# End of Runbook
