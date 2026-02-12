# AWS EKS Kubernetes Setup & Sample Deployments

This repository documents the step-by-step process used to provision an **Amazon EKS cluster**, configure access tools, and deploy sample workloads using Kubernetes manifests.

It includes deployment YAML files inside the `k8s/` directory for:

* Nginx
* Prometheus
* Grafana
* Jenkins
* WildFly

---

## 📌 Overview

This guide covers:

1. Creating an Ubuntu VM
2. Installing required CLI tools
3. Configuring AWS credentials
4. Creating an EKS cluster using `eksctl`
5. Connecting using `kubectl`
6. Deploying applications using Kubernetes YAML files

---

## 🧰 Prerequisites

* AWS Account
* Ubuntu VM (EC2 or local)
* Internet access
* Sudo privileges

---

## ⚙️ Step 1 — Install `eksctl`

```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# Optional checksum verification
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" \
  | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp
rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin
rm /tmp/eksctl
```

Verify:

```bash
eksctl version
```

---

## ☁️ Step 2 — Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Verify:

```bash
aws --version
```

---

## 🔐 Step 3 — Configure IAM User

1. Go to **AWS Console → IAM**
2. Create User
3. Attach Policy:

   ```
   AdministratorAccess
   ```
4. Create Access Key

   * Use case: CLI
5. Save Access Key ID & Secret

Configure on VM:

```bash
aws configure
```

Provide:

```
Access Key ID
Secret Access Key
Region (optional)
Output format (optional)
```

---

## 🔑 Step 4 — Generate SSH Key

```bash
ssh-keygen -t rsa
```

---

## 🚀 Step 5 — Create EKS Cluster

Create file:

### `cluster.yaml`

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-first-eks
  region: us-east-2

managedNodeGroups:
  - name: ng-feb-1
    instanceType: t2.large
    desiredCapacity: 3
    minSize: 1
    maxSize: 5
    ssh:
      allow: true
```

Create cluster:

```bash
eksctl create cluster -f cluster.yaml
```

Check status:

```bash
eksctl get cluster
eksctl get nodegroup --cluster=my-first-eks
Note: Add inboud rule for the cluster's security group so that you can access the services.
```

---

## ⎈ Step 6 — Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

curl -LO "https://dl.k8s.io/release/$(curl -L -s \
https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

echo "$(cat kubectl.sha256) kubectl" | sha256sum --check

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify:

```bash
kubectl version --client
```

---

## 🔗 Step 7 — Connect to Cluster

```bash
kubectl config current-context
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods --all-namespaces -o wide
kubectl get namespace
kubectl get svc --all-namespaces
```

---

## 📦 Step 8 — Deploy Applications

All Kubernetes manifests are inside:

```
k8s/
```

Example deployments include:

* `nginx-np.yaml`
* `prometheus-np.yaml`
* `grafana-np.yaml`
* `jenkins-np.yaml`
* `wildfly-np.yaml`

---

### Deploy Example

```bash
kubectl apply -f k8s/nginx-np.yaml
```

Check resources:

```bash
kubectl get pods
kubectl get deploy
kubectl get svc
```

---

## 📝 Important Label Matching Note

For Kubernetes services to route traffic correctly:

* Deployment labels
* Pod template labels
* Service selectors

**Must match**

Example:

```yaml
selector:
  matchLabels:
    app: my-webapp
```

and

```yaml
labels:
  app: my-webapp
```

If they differ — Service will not connect to Pods.

---

## 🧪 Troubleshooting

### ImagePullBackOff

Check:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

Common causes:

* Wrong image name
* Network issues
* DockerHub rate limits
* Private registry authentication

---

## 📁 Repository Structure

```
.
├── README.md
├── cluster.yaml
└── k8s/
    ├── nginx-np.yaml
    ├── prometheus-np.yaml
    ├── grafana-np.yaml
    ├── jenkins-np.yaml
    └── wildfly-np.yaml
```

---

## 🧹 Cleanup (Avoid AWS Charges)

Delete cluster when done:

```bash
eksctl delete cluster --name my-first-eks --region us-east-2
OR
eksctl delete nodegroup --cluster <cluster-name> --all
# This helps deleting the managed nodegroups along with cluster cause if there is already managed nodegroup inside cluster then we can't delete it.
```

---

## ✅ Conclusion

This project demonstrates:

* End-to-end EKS provisioning
* CLI-driven Kubernetes management
* Multi-application deployments
* Exposure via NodePort services

Useful for:

* Learning Kubernetes on AWS
* DevOps practice
* Monitoring stack experimentation

---

