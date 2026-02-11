# 🚀 GitHub Actions — Deploy Kubernetes YAML to EKS

This document explains how the GitHub Actions workflow (`deploy.yml`) automatically deploys Kubernetes manifests to an Amazon EKS cluster whenever YAML files inside the `k8s/` directory are changed.

This guide is beginner-friendly and walks through:

* Adding required secrets
* Understanding authentication flow
* Workflow file setup
* How deployment automation works

---

## 📌 Overview — How This Works

```
Developer pushes YAML → GitHub Actions runs
        ↓
AWS credentials configured
        ↓
Workflow connects to EKS
        ↓
kubectl applies changed YAML files
```

Result:

Your cluster updates automatically when manifests change.

---

# ✅ Step 1 — Add Required GitHub Secrets

GitHub Actions must authenticate with AWS and Kubernetes securely.
We store credentials as **Repository Secrets**.

Go to:

```
GitHub Repo
→ Settings
→ Secrets and variables
→ Actions
→ New repository secret
```

---

## 🔐 1️⃣ Add Kubeconfig Secret

On your machine:

```bash
cat ~/.kube/config | base64
```

Copy the output.

Create secret:

```
Name: KUBE_CONFIG
Value: <paste encoded output>
```

---

### 🧠 Why this is needed

Authentication flow:

```
aws configure
   ↓
AWS credentials created
   ↓
aws eks update-kubeconfig
   ↓
kubeconfig contains cluster access info
   ↓
kubectl communicates with EKS
```

---

## 🔐 2️⃣ Add AWS Secrets

Create these repository secrets:

| Secret Name           | Value             |
| --------------------- | ----------------- |
| AWS_ACCESS_KEY_ID     | IAM Access Key    |
| AWS_SECRET_ACCESS_KEY | IAM Secret Key    |
| AWS_REGION            | us-east-2         |
| CLUSTER_NAME          | your_cluster_name |

These allow GitHub runner to access AWS and locate your cluster.

---

# ✅ Step 2 — Create Workflow File

Create:

```
.github/workflows/deploy.yml
```

Paste:

```yaml
name: Deploy YAML to EKS

on:
  push:
    paths:
      - 'k8s/**.yaml'
      - 'k8s/**.yml'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:

    # Checkout repository code
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0

    # Configure AWS credentials
    - name: Configure AWS
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    # Connect kubectl to cluster
    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig \
          --region ${{ secrets.AWS_REGION }} \
          --name ${{ secrets.CLUSTER_NAME }}

    # Apply only changed YAML files
    - name: Deploy changed YAMLs
      run: |
        CHANGED=$(git diff --name-only ${{ github.event.before }} ${{ github.sha }} | grep '^k8s/.*\.ya\?ml$' || true)

        for file in $CHANGED; do
          echo "Applying $file"
          kubectl apply -f $file
        done
```

---

# 🧩 What This Workflow Does (Beginner Explanation)

### Trigger

Runs when:

```
Any .yaml/.yml file changes inside k8s/
```

---

### Step Breakdown

## 1️⃣ Checkout Code

Downloads repo into runner.

## 2️⃣ Configure AWS

Uses secrets to login to AWS.

## 3️⃣ Update kubeconfig

Connects kubectl to your EKS cluster.

## 4️⃣ Deploy Changes

Detects modified YAML files and runs:

```
kubectl apply -f <file>
```

Only changed files are applied — efficient deployment.

---

# ✅ Step 3 — Using It

Now simply:

```
Edit or add YAML inside k8s/
git commit
git push
```

GitHub Actions will:

✔ Detect change
✔ Connect to cluster
✔ Deploy automatically

---

# 🔎 Verification

Check cluster:

```bash
kubectl get pods
kubectl get svc
```

Or view workflow logs:

```
GitHub Repo
→ Actions tab
→ Deploy YAML to EKS
```

---

# ⚠️ Security Best Practices

* Never commit kubeconfig to repo
* Use minimal IAM permissions
* Rotate access keys regularly
* Consider switching to OIDC authentication later

---
## 🎉 Summary

You now have CI/CD deployment for Kubernetes manifests:

* Automated
* Secure
* Beginner-friendly
* Production-style workflow foundation

