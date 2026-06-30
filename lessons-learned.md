# Lessons Learned

# Kubernetes on AWS using kOps
## Lessons Learned

---

# Project Overview

This project involved deploying the **vProfile Java Application** on a Kubernetes cluster provisioned using **kOps** on **AWS**. The deployment included configuring infrastructure, networking, persistent storage, application services, ingress routing, and Git version control.

Beyond achieving a successful deployment, the project provided practical experience in Kubernetes administration, AWS infrastructure management, troubleshooting, and DevOps best practices.

---

# 1. Build Infrastructure Before Deploying Applications

One of the earliest lessons was the importance of following the correct deployment sequence.

Initially, the NGINX Ingress Controller was installed before creating the Kubernetes cluster, resulting in API connection errors because no Kubernetes API server existed.

The correct sequence is:

1. Create AWS infrastructure
2. Create Kubernetes cluster using kOps
3. Export kubeconfig
4. Validate cluster health
5. Install Ingress Controller
6. Deploy Kubernetes resources

Infrastructure should always be fully operational before deploying workloads.

---

# 2. Always Validate the Cluster

Before deploying any application, verify that the Kubernetes cluster is healthy.

Useful commands:

```bash
kops validate cluster

kubectl get nodes

kubectl get ns
```

A healthy cluster saves significant troubleshooting time later.

---

# 3. Export kubeconfig After Cluster Creation

Creating the cluster alone is not sufficient.

`kubectl` cannot communicate with the cluster until kubeconfig has been exported.

Required commands:

```bash
export KOPS_STATE_STORE=s3://kopsstate0584

kops export kubecfg \
--name kubepro.hkhsinfo.shop \
--admin
```

This establishes communication between `kubectl` and the Kubernetes API server.

---

# 4. Validate YAML Before Deployment

Small YAML formatting mistakes can prevent Kubernetes from creating resources.

Examples encountered:

- Incorrect indentation
- Missing spaces
- Incorrect list formatting
- Invalid field placement

Using client-side validation before applying manifests prevented unnecessary deployment failures.

```bash
kubectl apply --dry-run=client -f filename.yaml
```

This became a standard validation step before every deployment.

---

# 5. Understand Kubernetes Resource Relationships

The project reinforced how Kubernetes resources depend on each other.

Deployment

↓

Pod

↓

Service

↓

Ingress

↓

DNS

↓

Application

A failure in any component affects the complete application path.

Understanding these relationships simplifies troubleshooting.

---

# 6. Service Names Matter

Kubernetes service discovery relies entirely on DNS.

A single character mismatch in a service name prevented the application from starting.

Example:

Incorrect

```
vpromq01
```

Correct

```
vprormq01
```

Always verify:

- Service names
- Selectors
- Labels
- DNS references

---

# 7. Init Containers Improve Application Reliability

The application should not start before dependent services become available.

Using Init Containers ensured that:

- MySQL became reachable
- RabbitMQ became reachable

before the application container started.

This eliminated startup race conditions.

---

# 8. Resource Planning is Critical

The application repeatedly failed with:

```
OOMKilled
```

because memory limits were too small.

Increasing CPU and memory requests solved the problem.

Lessons learned:

- Java applications typically require more memory than lightweight services.
- Resource requests and limits should be planned based on application requirements rather than minimal defaults.

---

# 9. Persistent Storage Requires Proper Configuration

The database deployment depended on:

- PersistentVolumeClaim
- StorageClass
- EBS volumes

The PVC successfully provisioned an AWS EBS volume automatically.

This demonstrated Kubernetes dynamic volume provisioning.

Verification commands:

```bash
kubectl get pvc

kubectl get sc
```

---

# 10. Kubernetes Troubleshooting is Systematic

Instead of guessing, troubleshoot layer by layer.

Typical workflow:

1. Check cluster
2. Check nodes
3. Check deployments
4. Check pods
5. Check services
6. Check ingress
7. Check logs
8. Describe resources
9. Verify events

Useful commands:

```bash
kubectl describe

kubectl logs

kubectl get events
```

Following a structured approach reduces troubleshooting time.

---

# 11. Ingress Simplifies External Access

Instead of exposing the application directly, NGINX Ingress provided:

- HTTP routing
- Host-based routing
- Centralized access
- Cleaner architecture

The application became accessible through:

```
http://vprofile.hkhsinfo.shop/welcome
```

This mirrors production-grade Kubernetes deployments.

---

# 12. Git Best Practices

The project reinforced several Git best practices:

- Commit frequently
- Use meaningful commit messages
- Keep the repository synchronized
- Resolve merge conflicts carefully
- Validate changes before pushing

Switching from HTTPS authentication to SSH simplified repository access and eliminated repeated credential prompts.

Useful commands:

```bash
git status

git add .

git commit

git push origin main
```

---

# 13. Infrastructure as Code

Everything required to recreate the application exists as code.

This includes:

- Deployment manifests
- Services
- PersistentVolumeClaims
- Ingress
- Secrets

Benefits:

- Repeatable deployments
- Version control
- Easier collaboration
- Disaster recovery
- Consistent environments

---

# 14. Importance of Documentation

Maintaining documentation throughout the project proved invaluable.

The following documents were created:

- README.md
- architecture.md
- deployment-guide.md
- runbook.md
- troubleshooting-guide.md
- lessons-learned.md

These documents reduce future setup time and provide a clear reference for troubleshooting and operational tasks.

---

# 15. AWS Services Used

This project provided hands-on experience with several AWS services:

- EC2
- VPC
- Route53
- S3
- Elastic Load Balancer
- EBS
- IAM
- Security Groups

Understanding how Kubernetes integrates with AWS infrastructure is essential for production deployments.

---

# 16. Kubernetes Concepts Reinforced

This project strengthened understanding of:

- Pods
- Deployments
- ReplicaSets
- Services
- ClusterIP
- Ingress
- Init Containers
- PersistentVolumeClaims
- StorageClasses
- Namespaces
- Secrets
- Resource Requests
- Resource Limits
- DNS-based Service Discovery

---

# 17. Operational Best Practices

Key practices adopted during the project include:

- Validate YAML before deployment.
- Verify cluster health before deploying applications.
- Monitor pod status during rollouts.
- Review logs before modifying configurations.
- Use rollout restarts after updating deployments.
- Test application accessibility after every major change.
- Commit working configurations to Git regularly.
- Document issues and resolutions immediately after troubleshooting.

---

# Key Takeaways

This project demonstrated that successful Kubernetes deployments depend on much more than writing YAML files. They require a solid understanding of infrastructure provisioning, networking, storage, resource management, service discovery, and deployment workflows.

By resolving real-world issues such as cluster configuration, YAML syntax errors, service naming mismatches, resource constraints, Git authentication, and deployment sequencing, this project significantly improved practical DevOps skills.

The experience also highlighted the importance of automation, Infrastructure as Code, version control, validation, monitoring, and thorough documentation. These practices form the foundation of reliable and repeatable production deployments.

---

# Final Outcome

Successfully deployed the **vProfile Application** on a **kOps-managed Kubernetes cluster** running on **AWS**, with:

- Kubernetes cluster provisioned using kOps
- Dynamic EBS persistent storage
- NGINX Ingress Controller
- Route53 DNS integration
- Java application running successfully
- MySQL database with persistent storage
- RabbitMQ messaging service
- Memcached caching service
- GitHub repository updated with the latest manifests
- Comprehensive project documentation created

The application was successfully accessed at:

```
http://vprofile.hkhsinfo.shop/welcome
```

This project represents a complete end-to-end Kubernetes deployment lifecycle on AWS and serves as a strong foundation for more advanced topics such as GitOps, Helm, Argo CD, and production-grade Kubernetes operations.

---
