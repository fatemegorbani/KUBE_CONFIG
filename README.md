# پروژه میکروسرویس Todo با Kubernetes و Minikube


## 📝 فهرست مطالب
- [معرفی پروژه](#-معرفی-پروژه)
- [پیش‌نیازها](#-پیشنیازها)
- [راه‌اندازی اولیه](#-راهاندازی-اولیه)
- [استقرار روی Kubernetes](#-استقرار-روی-kubernetes)
- [مدیریت و مقیاس‌پذیری](#-مدیریت-و-مقیاسپذیری)


## 🌟 معرفی پروژه
این پروژه یک سرویس مدیریت کارهای روزانه (Todo) است که به صورت میکروسرویس توسعه داده شده و با استفاده از Kubernetes مستقر می‌شود.

## 🛠️ پیش‌نیازها

### نرم‌افزارهای مورد نیاز
- [Docker](https://docs.docker.com/get-docker/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)



## 🚀 راه‌اندازی اولیه

### 1. کلون کردن مخزن
```bash
git clone https://github.com/fatemegorbani/KUBE_CONFIG.git
cd KUBE_CONFIG
```

### 2. ساخت image داکر
```bash
docker build -t todo-app .
```

### 3. راه‌اندازی Minikube
```bash
minikube start
eval $(minikube docker-env)  # برای استفاده از imageهای محلی
```

## ☸️ استقرار روی Kubernetes

### فایل‌های پیکربندی

#### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo
  template:
    metadata:
      labels:
        app: todo
    spec:
      containers:
      - name: todo
        image: todo-app
        ports:
        - containerPort: 5000
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 10
```

#### service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: todo-service
spec:
  selector:
    app: todo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```

### دستورات استقرار
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### دسترسی به سرویس
```bash
minikube service todo-service
```

## 📈 مدیریت و مقیاس‌پذیری

### مقیاس‌دهی دستی
```bash
kubectl scale deployment todo-app --replicas=3
```

### مقیاس‌دهی خودکار (HPA)
```bash
minikube addons enable metrics-server
kubectl autoscale deployment todo-app --cpu-percent=50 --min=2 --max=5
```

### مشاهده وضعیت
```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get hpa
```

## 🚨 عیب‌یابی

### مشکلات رایج و راه‌حل‌ها

| مشکل | راه‌حل |
|------|--------|
| `ImagePullBackOff` | `eval $(minikube docker-env)` و rebuild image |
| `CrashLoopBackOff` | `kubectl logs <pod-name>` |
| سرویس در دسترس نیست | `minikube service list` |
| خطا در `git push` | `git pull origin main --allow-unrelated-histories` |

### بررسی لاگ‌ها
```bash
kubectl logs -f <pod-name>
kubectl describe pod <pod-name>
```

## 📚 مستندات فنی

### ساختار پروژه
```
myproject/
├── .github/
│   └── workflows/
│       └── main.yml
├── app.py
├── Dockerfile
├── deployment.yaml
├── service.yaml
└── README.md
```

### API Endpoints
- `GET /tasks` - لیست همه کارها
- `POST /tasks` - اضافه کردن کار جدید
- `GET /tasks/{id}` - دریافت کار بر اساس ID
- `DELETE /tasks/{id}` - حذف کار

### CI/CD Pipeline
پیکربندی GitHub Actions در `.github/workflows/main.yml`:
```yaml
name: Deploy to Kubernetes
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - run: kubectl apply -f deployment.yaml
```
