# 🚀 Argo CD Installation on EC2 (Amazon Linux – dnf)

This guide explains how to install **Docker**, **Minikube**, and **Argo CD**
on an **EC2 instance running Amazon Linux** using **dnf**.

---

## 📌 Prerequisites

- EC2 instance (Amazon Linux 2023 / Amazon Linux 2)
- SSH access to the instance
- User with sudo privileges
- Minimum: **2 vCPU / 4 GB RAM**
- Security Group open ports:
  - 22 (SSH)
  - 8080 (Argo CD UI)

---

## 🔹 Step 1: Update System

```bash
sudo dnf update -y
```

---

## 🔹 Step 2: Install Docker

### Install Docker
```bash
sudo dnf install -y docker
```

### Start & Enable Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### Add user to Docker group
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Verify Docker
```bash
docker --version
docker ps
```

---

## 🔹 Step 3: Install kubectl

```bash
curl -LO https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

### Verify
```bash
kubectl version --client
```

---

## 🔹 Step 4: Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

### Start Minikube (Docker driver)
```bash
minikube start --driver=docker
```

### Verify Cluster
```bash
kubectl get nodes
```

---

## 🔹 Step 5: Install Argo CD

### Create Namespace
```bash
kubectl create namespace argocd
```

### Install Argo CD
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Check Pods
```bash
kubectl get pods -n argocd
```

Wait until all pods are in **Running** state.

---

## 🔹 Step 6: Expose Argo CD UI

### Option 1: Port Forward (Recommended)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access UI:
```
https://<EC2_PUBLIC_IP>:8080
```

---

### Option 2: NodePort

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

```bash
kubectl get svc -n argocd
```

Access:
```
https://<EC2_PUBLIC_IP>:<NODE_PORT>
```

---

## 🔹 Step 7: Get Argo CD Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

Login:
- **Username:** admin
- **Password:** output of the command

---

## 🔹 Step 8: (Optional) Install Argo CD CLI

```bash
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

chmod +x argocd
sudo mv argocd /usr/local/bin/argocd
```

### Verify
```bash
argocd version
```

---

## 🧪 Final Verification

```bash
kubectl get all -n argocd
```

If all resources are running → 🎉 **Argo CD Installed Successfully**

---

## 🛑 Troubleshooting

- Docker permission denied → relogin or run `newgrp docker`
- Minikube not starting → check:
  ```bash
  minikube status
  ```
- UI not accessible → check EC2 Security Group ports

---

## 📚 References

- https://argo-cd.readthedocs.io
- https://kubernetes.io
- https://minikube.sigs.k8s.io
