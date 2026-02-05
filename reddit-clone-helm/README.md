# دليل التشغيل على EC2 + Minikube + Argo CD

هذا الدليل يشرح خطوة بخطوة تشغيل مشروع Helm على **EC2 Amazon Linux (RHEL-based)** باستخدام **Minikube**، ثم تفعيل **Argo CD** ونشر كل المكونات (gateway, backend, frontend) من ملفات Argo CD الموجودة في هذا الريبو.

## المتطلبات

- EC2 Amazon Linux (مبني على Red Hat / dnf)
- 2 vCPU و 4 GB RAM على الأقل
- منافذ الـ Security Group:
  - 22 للـ SSH
  - 8080 لواجهة Argo CD (لو Port Forward)
  - 80 و 443 للتجارب الخارجية لو احتجتها

---

## 1) تحديث النظام

```bash
sudo dnf update -y
sudo dnf upgrade -y
```

---

## 2) تثبيت Docker

```bash
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

تأكد:
```bash
docker --version
```

---

## 3) تثبيت kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

تأكد:
```bash
kubectl version --client
```

---

## 4) تثبيت Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube-linux-amd64
sudo mv minikube-linux-amd64 /usr/local/bin/minikube
```

تشغيل الكلاستر:
```bash
minikube start --driver=docker
```

تأكد:
```bash
kubectl get nodes
```

---

## 5) تثبيت Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

تأكد:
```bash
helm version
```

---

## 6) تثبيت Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

انتظر حتى كل الـ Pods تشتغل:
```bash
kubectl get pods -n argocd
```

---

## 7) فتح واجهة Argo CD

**الطريقة المفضلة: Port Forward**
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

افتح:
```
https://<EC2_PUBLIC_IP>:8080
```

**الباسورد الافتراضي:**
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

اليوزر:
```
admin
```

---

## 8) تجهيز الـ Namespace للتطبيقات

```bash
kubectl create namespace reddit
kubectl label namespace reddit expose-via-gateway=true
```

هذا اللابل ضروري لأن الـ Gateway في [gateway/values.yaml](gateway/values.yaml) يسمح بالـ Routes فقط للـ namespaces اللي عليها هذا اللابل.

---

## 9) تعديل repoURL لو أنت مستخدم Fork

ملفات Argo CD موجودة في:
- [Argo CD/backend-app.yaml](Argo%20CD/backend-app.yaml)
- [Argo CD/frontend-app.yaml](Argo%20CD/frontend-app.yaml)
- [Argo CD/gateway-app.yaml](Argo%20CD/gateway-app.yaml)

لو الريبو مختلف، عدل `repoURL` بالقيمة الصحيحة:
```
https://github.com/USERNAME/reddit-clone-helm.git
```

---

## 10) نشر الـ Applications من Argo CD

```bash
kubectl apply -f "Argo CD/gateway-app.yaml"
kubectl apply -f "Argo CD/backend-app.yaml"
kubectl apply -f "Argo CD/frontend-app.yaml"
```

تابع الحالة:
```bash
kubectl get applications -n argocd
```

أو من واجهة Argo CD UI.

---

## 11) التحقق من التشغيل

```bash
kubectl get pods -n reddit
kubectl get svc -n reddit
kubectl get gateway -n reddit
kubectl get httproute -n reddit
```

---

## ملاحظات مهمة

- `gateway` هو نقطة الدخول الرئيسية.
- `backend` و `frontend` يتنشروا في نفس الـ namespace.
- كل القيم قابلة للتعديل من ملفات `values.yaml` داخل كل Chart.
- لو عندك TLS، عدل الـ Secret في [gateway/values.yaml](gateway/values.yaml).

---

## تشغيل سريع بدون Argo CD (اختياري)

```bash
helm install gateway ./gateway -n reddit
helm install backend ./backend -n reddit
helm install frontend ./frontend -n reddit
```

---

## استكشاف الأخطاء

- لو Minikube لم يبدأ:
  ```bash
  minikube status
  minikube logs
  ```
- لو Argo CD UI غير متاحة:
  - تأكد الـ Port Forward شغال
  - تأكد الـ Security Group يسمح بالـ 8080
- لو الـ Apps لا تتزامن:
  - راجع `repoURL` و `path` في ملفات Argo CD

---

## مراجع

- https://argo-cd.readthedocs.io
- https://kubernetes.io
- https://minikube.sigs.k8s.io
- https://helm.sh
