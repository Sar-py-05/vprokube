# Step 11 – Correct Kubernetes Manifest Files

During deployment, several Kubernetes manifest files failed validation due to YAML syntax errors and incorrect field placement. Instead of recreating all resources, each manifest was reviewed, corrected, validated using a dry run, and then redeployed.

---

## 11.1 Fix `dbdeploy.yaml`

### Problem

The deployment failed because of an invalid YAML structure.

The following line was incorrectly written:

```yaml
-mountPath: /var/lib/mysql
```

Since the hyphen was attached directly to `mountPath`, Kubernetes could not parse the YAML.

---

### Correct Configuration

```yaml
volumeMounts:
  - mountPath: /var/lib/mysql
    name: vpro-db-data
```

---

### Validate

```bash
kubectl apply --dry-run=client -f dbdeploy.yaml
```

Expected Output

```text
deployment.apps/vprodb created (dry run)
```

Deploy

```bash
kubectl apply -f dbdeploy.yaml
```

Expected Output

```text
deployment.apps/vprodb created
```

---

## 11.2 Fix Service YAML Files

The following Service manifests contained an invalid field.

- appservice.yaml
- dbservice.yaml
- mcservice.yaml

Incorrect configuration

```yaml
ports:
  - protocol: TCP
    port: 8080
    targetPort: vproapp-port
    type: ClusterIP
```

The `type` field belongs under `spec`, not inside `ports`.

Correct configuration

```yaml
spec:
  type: ClusterIP

  ports:
    - protocol: TCP
      port: 8080
      targetPort: vproapp-port
```

Repeat the correction for

- dbservice.yaml
- mcservice.yaml

---

## 11.3 Fix `appingress.yaml`

The ingress manifest also contained an indentation error.

Incorrect

```yaml
-path: /
```

Correct

```yaml
- path: /
```

Also ensure

```yaml
ingressClassName: nginx
```

is present.

---

## Validate All Corrected Files

```bash
kubectl apply --dry-run=client -f appservice.yaml

kubectl apply --dry-run=client -f dbservice.yaml

kubectl apply --dry-run=client -f mcservice.yaml

kubectl apply --dry-run=client -f appingress.yaml
```

Expected

```text
created (dry run)
```

Deploy

```bash
kubectl apply -f appservice.yaml

kubectl apply -f dbservice.yaml

kubectl apply -f mcservice.yaml

kubectl apply -f appingress.yaml
```

---

# Step 12 – Verify Pod Status

Check Pods.

```bash
kubectl get pods
```

Initially the application Pod remained stuck in

```
Init:1/2
```

This indicated that one Init Container had completed while another was still waiting.

---

# Step 13 – Investigate the Application Pod

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

The description showed

```
init-myservice

Completed
```

while

```
init-rabbitmq

Running
```

The application container never started because the second Init Container was still waiting.

---

## Identify Init Containers

```bash
kubectl get pod <pod-name> \
-o jsonpath='{.spec.initContainers[*].name}'
```

Output

```
init-myservice init-rabbitmq
```

---

## Root Cause

The Init Container was attempting to resolve

```
vpromq01
```

However the actual Kubernetes Service name was

```
vprormq01
```

Notice the missing letters **rm**.

Because Kubernetes DNS could not resolve the incorrect Service name, the Init Container continued waiting indefinitely.

---

## Resolution

Update the Init Container.

Incorrect

```yaml
until nslookup vpromq01
```

Correct

```yaml
until nslookup vprormq01
```

---

# Step 14 – Fix Application Deployment

The application deployment was updated.

The corrected Init Containers became

```yaml
initContainers:

- name: init-myservice

- name: init-rabbitmq
```

Both containers now waited for

- MySQL Service
- RabbitMQ Service

before allowing the Java application to start.

---

Validate

```bash
kubectl apply --dry-run=client -f appdeploy.yaml
```

Deploy

```bash
kubectl apply -f appdeploy.yaml
```

Restart Deployment

```bash
kubectl rollout restart deployment vproapp
```

Monitor

```bash
kubectl get pods -w
```

---

# Step 15 – Resolve OOMKilled Error

After fixing the Init Containers, another issue appeared.

```
OOMKilled
```

---

## Cause

The Java application required significantly more memory than allocated.

Original configuration

```yaml
requests:
  memory: 64Mi

limits:
  memory: 128Mi
```

This was insufficient for the JVM.

---

## Resolution

Increase resource allocation.

```yaml
requests:

  cpu: 250m

  memory: 512Mi

limits:

  cpu: 500m

  memory: 1024Mi
```

Deploy

```bash
kubectl apply -f appdeploy.yaml

kubectl rollout restart deployment vproapp
```

Monitor

```bash
kubectl get pods -w
```

The application successfully reached

```
Running
```

---

# Step 16 – Validate Services

Display Services.

```bash
kubectl get svc
```

Describe the application Service.

```bash
kubectl describe svc vproapp-service
```

Confirm

- ClusterIP assigned

- Endpoint exists

- Port mapping correct

---

Verify RabbitMQ

```bash
kubectl describe svc vprormq01
```

Ensure

```
Endpoints:
```

contains the RabbitMQ Pod IP.

---

# Step 17 – Validate Persistent Storage

Verify PVC.

```bash
kubectl get pvc
```

Expected

```
STATUS

Bound
```

Check Storage Classes.

```bash
kubectl get sc
```

The cluster dynamically provisioned an Amazon EBS volume using the default StorageClass.

---

# Step 18 – Validate Ingress

List Ingress.

```bash
kubectl get ingress
```

Describe Ingress.

```bash
kubectl describe ingress vpro-ingress
```

Confirm

- Host configured
- Address assigned
- Backend Service mapped
- NGINX Ingress Class

---

# Step 19 – Configure Route53

Create an Alias Record.

```
vprofile.hkhsinfo.shop
```

Point the Alias Record to

```
AWS Load Balancer
```

Once DNS propagated, verify

```
http://vprofile.hkhsinfo.shop/welcome
```

The application loaded successfully.

---

# Step 20 – Synchronize GitHub Repository

Initially the repository remote used HTTPS.

```bash
git remote -v
```

Updated the remote.

```bash
git remote set-url origin \
git@github.com:Sar-py-05/vprokube.git
```

Verify

```bash
git remote -v
```

Test SSH.

```bash
ssh -T git@github.com
```

Output

```
Hi Sar-py-05!

You've successfully authenticated...
```

Push changes.

```bash
git push origin main
```

The repository synchronized successfully without requiring a username or Personal Access Token.

---

# Step 21 – Cleanup

Delete the Ingress Controller.

```bash
kubectl delete -f \
https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```

Delete Kubernetes resources.

```bash
kubectl delete -f .
```

Delete the Kubernetes cluster.

```bash
kops delete cluster \
--name kubepro.hkhsinfo.shop \
--yes
```

This removes

- EC2 Instances
- ELB
- EBS Volumes
- Security Groups
- Auto Scaling Groups
- Route53 Records
- Networking Components

---

# Final Validation Checklist

✔ Kubernetes cluster created

✔ Control Plane Ready

✔ Worker Nodes Ready

✔ kubectl configured

✔ Ingress Controller installed

✔ PVC Bound

✔ MySQL Running

✔ RabbitMQ Running

✔ Memcached Running

✔ Application Running

✔ Services Created

✔ Ingress Created

✔ Route53 Configured

✔ Application accessible

```
http://vprofile.hkhsinfo.shop/welcome
```

✔ GitHub Repository Updated

✔ SSH Authentication Configured

✔ Cluster Successfully Cleaned Up

---

# Commands Used During the Project

The following commands were frequently used throughout the deployment.

```bash
git remote -v

git status

git add .

git commit -m

git push origin main

kops create cluster

kops update cluster

kops validate cluster

kops export kubecfg

kubectl get nodes

kubectl get pods

kubectl get pods -w

kubectl get svc

kubectl get pvc

kubectl get ingress

kubectl get sc

kubectl describe pod

kubectl describe svc

kubectl describe ingress

kubectl apply --dry-run=client

kubectl apply -f

kubectl create -f

kubectl rollout restart deployment

kubectl delete -f

grep -n

cat -n

sed -n

vim
```

Each command played a specific role in deploying, validating, troubleshooting, and maintaining the Kubernetes application.

---

# Conclusion

This project successfully demonstrated the end-to-end deployment of a production-style Java enterprise application on a Kubernetes cluster running on Amazon Web Services using kOps.

The deployment covered infrastructure provisioning, Kubernetes administration, persistent storage, service networking, ingress configuration, DNS integration, application deployment, troubleshooting, and GitHub repository synchronization.

Several real-world issues—including YAML syntax errors, invalid Kubernetes object definitions, Init Container dependency failures, incorrect service discovery, Out of Memory (OOMKilled) conditions, and Git authentication challenges—were identified and resolved. Addressing these issues provided valuable practical experience that closely mirrors production Kubernetes operations.

The completed environment validates a fully functional multi-tier application architecture and establishes a strong foundation for future GitOps, CI/CD, monitoring, and advanced Kubernetes projects.
