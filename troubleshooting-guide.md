# Troubleshooting Guide

# Kubernetes on AWS using kOps
## Troubleshooting Guide

---

# Project Overview

This document captures the real issues encountered while deploying the **vProfile Application on Kubernetes using kOps on AWS**, along with their root causes and resolutions.

Recording these issues provides a quick reference for future deployments and helps reduce troubleshooting time.

---

# Issue 1
## kubectl unable to connect to Kubernetes API

### Error

```
error validating data:

failed to download openapi

Get http://localhost:8080/openapi/v2

connect: connection refused
```

---

### Cause

Ingress Controller was installed before the Kubernetes cluster had been created.

Since no cluster existed, `kubectl` attempted to connect to the default localhost API server.

---

### Resolution

Create the cluster first.

```bash
kops create cluster ...
```

Update the cluster.

```bash
kops update cluster --yes
```

Export kubeconfig.

```bash
export KOPS_STATE_STORE=s3://kopsstate0584

kops export kubecfg \
--name kubepro.hkhsinfo.shop \
--admin
```

Validate.

```bash
kops validate cluster
```

After the cluster becomes Ready, install the Ingress Controller.

---

# Issue 2
## Cluster not configured for kubectl

### Symptoms

kubectl commands fail.

```
connection refused

no configuration found
```

---

### Cause

kubeconfig was never exported.

---

### Resolution

```bash
export KOPS_STATE_STORE=s3://kopsstate0584

kops export kubecfg \
--name kubepro.hkhsinfo.shop \
--admin
```

Verify.

```bash
kubectl get nodes
```

---

# Issue 3
## Service YAML Validation Error

### Error

```
strict decoding error

unknown field

spec.ports[0].type
```

---

### Cause

Service type was placed under the port definition.

Incorrect

```yaml
ports:
  - port: 8080
    targetPort: vproapp-port
    type: ClusterIP
```

---

### Correct

```yaml
spec:
  type: ClusterIP

  ports:
    - port: 8080
      targetPort: vproapp-port
```

---

# Issue 4
## YAML Parsing Error

### Error

```
mapping values are not allowed in this context
```

---

### Cause

Incorrect indentation.

Example

```
-mountPath:
```

instead of

```
- mountPath:
```

Another example

```
-path:
```

instead of

```
- path:
```

---

### Resolution

Validate spacing carefully.

Useful commands

```bash
cat -n filename.yaml

sed -n '1,80p' filename.yaml
```

---

# Issue 5
## PVC Already Exists

### Error

```
persistentvolumeclaim already exists
```

---

### Cause

PVC was created individually before running

```bash
kubectl create -f .
```

---

### Resolution

Ignore the error.

PVC had already been created successfully.

Confirm.

```bash
kubectl get pvc
```

---

# Issue 6
## Application Pod Stuck in Init

### Status

```
Init:0/2

Init:1/2
```

---

### Cause

The application waits for dependent services.

The Init Containers perform DNS lookups before starting the application.

---

### Investigation

```bash
kubectl describe pod <pod-name>
```

Found

```
waiting for RabbitMQ
```

---

### Root Cause

Service name mismatch.

Application expected

```
vprormq01
```

while configuration contained

```
vpromq01
```

---

### Resolution

Correct the service name inside

```
appdeploy.yaml
```

Restart deployment.

```bash
kubectl rollout restart deployment vproapp
```

---

# Issue 7
## Wrong Init Container Logs

### Error

```
container init-mydb is not valid
```

---

### Cause

Incorrect container name.

---

### Resolution

Find available Init Containers.

```bash
kubectl get pod <pod> \
-o jsonpath='{.spec.initContainers[*].name}'
```

Then use the correct container.

```bash
kubectl logs <pod> \
-c init-rabbitmq
```

---

# Issue 8
## OOMKilled

### Symptoms

```
OOMKilled

CrashLoopBackOff
```

---

### Cause

Application memory limits were too low.

Original values

```yaml
requests:
  memory: 64Mi

limits:
  memory: 128Mi
```

The Java application required significantly more memory.

---

### Resolution

Increase resources.

```yaml
requests:
  cpu: 250m
  memory: 512Mi

limits:
  cpu: 500m
  memory: 1024Mi
```

Apply changes.

```bash
kubectl apply -f appdeploy.yaml

kubectl rollout restart deployment vproapp
```

Application started successfully.

---

# Issue 9
## Dry Run Validation

### Best Practice

Always validate before applying.

```bash
kubectl apply \
--dry-run=client \
-f filename.yaml
```

This prevents malformed YAML from reaching the cluster.

---

# Issue 10
## Git Push Requested Username and Password

### Symptoms

```
Username for https://github.com

Password
```

---

### Cause

Repository remote used HTTPS.

```
https://github.com/...
```

GitHub no longer accepts password authentication.

---

### Resolution

Switch remote to SSH.

```bash
git remote set-url origin \
git@github.com:Sar-py-05/vprokube.git
```

Verify.

```bash
git remote -v
```

Expected

```
git@github.com:...
```

---

# Issue 11
## Git Rebase Merge Conflicts

### Symptoms

```
CONFLICT

both modified
```

---

### Cause

Remote repository had newer commits while local repository also contained changes.

---

### Resolution

Accept local version (or remote version as appropriate).

Example

```bash
git checkout --theirs filename
```

or

```bash
git checkout --ours filename
```

Then

```bash
git add .

git rebase --continue
```

---

# Issue 12
## Git Authentication Verification

Verify SSH authentication.

```bash
ssh -T git@github.com
```

Expected

```
Hi Sar-py-05!

You've successfully authenticated.
```

---

# Issue 13
## Verify Ingress

List Ingress.

```bash
kubectl get ingress
```

Describe.

```bash
kubectl describe ingress vpro-ingress
```

Verify

- Host
- Backend Service
- Load Balancer Address

---

# Issue 14
## Verify Services

```bash
kubectl get svc
```

Describe Service.

```bash
kubectl describe svc vproapp-service
```

Verify

- ClusterIP
- TargetPort
- Endpoints

---

# Issue 15
## Verify Persistent Volume

```bash
kubectl get pvc
```

Expected

```
STATUS

Bound
```

Storage Classes

```bash
kubectl get sc
```

---

# Issue 16
## Verify Cluster Health

Nodes

```bash
kubectl get nodes
```

Pods

```bash
kubectl get pods
```

Namespaces

```bash
kubectl get ns
```

Validate Cluster

```bash
kops validate cluster
```

Expected

```
Your cluster is ready
```

---

# Issue 17
## Application Verification

Verify application.

```
http://vprofile.hkhsinfo.shop/welcome
```

Successful deployment confirms:

- DNS resolution
- Ingress configuration
- Load Balancer
- Service routing
- Kubernetes networking
- Application deployment
- Database connectivity
- RabbitMQ connectivity
- Memcached connectivity

---

# General Troubleshooting Checklist

Before debugging, always verify the following:

✓ Cluster created

✓ kubeconfig exported

✓ Cluster validated

✓ Nodes Ready

✓ PVC Bound

✓ Deployments Running

✓ Pods Running

✓ Services Created

✓ Endpoints Available

✓ Ingress Created

✓ DNS Configured

✓ Load Balancer Provisioned

✓ Application Reachable

---

# Useful Commands

```bash
kubectl get all

kubectl get pods

kubectl get svc

kubectl get ingress

kubectl get pvc

kubectl get sc

kubectl describe pod <pod>

kubectl logs <pod>

kubectl rollout restart deployment vproapp

kubectl get events --sort-by=.metadata.creationTimestamp

kops validate cluster

kubectl get nodes
```

---

# Summary

During this deployment, several real-world issues were encountered, including:

- Cluster creation sequence
- Missing kubeconfig
- YAML syntax errors
- Incorrect Service definitions
- Service naming mismatches
- Init Container failures
- Java application memory limits
- Git authentication using HTTPS
- Git rebase conflicts
- Kubernetes deployment validation

Resolving these issues significantly improved understanding of Kubernetes, kOps, AWS infrastructure, YAML validation, Kubernetes networking, and Git workflows. This guide serves as a practical reference for future deployments and production troubleshooting.

---
