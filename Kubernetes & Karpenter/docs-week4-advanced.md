
# 🎓 TUẦN 4: NÂNG CAO - Helm, ArgoCD, Istio, CI/CD

> **Chào mừng bạn đến tuần 4!** Bạn đã biết cơ bản K8s rồi. Giờ mình sẽ dạy những công cụ mà **dân Production thật sự dùng hàng ngày**.

---

# 🔰 BÀI 8: HELM CHARTS - ĐÓNG GÓI ỨNG DỤNG

## Câu chuyện: Vấn đề khi deploy app thật

```
Bạn có 1 app gồm:
  - deployment.yaml      (chạy app)
  - service.yaml         (expose port)
  - configmap.yaml       (config)
  - secret.yaml          (mật khẩu)
  - ingress.yaml         (domain)
  - hpa.yaml             (auto scale)
  - pvc.yaml             (storage)

→ 7 files YAML! Mỗi lần deploy phải:
  kubectl apply -f deployment.yaml
  kubectl apply -f service.yaml
  kubectl apply -f configmap.yaml
  ... 7 lần ❌

Thêm nữa:
  - Dev environment: image tag = "dev-v1", replicas = 1
  - Staging: image tag = "staging-v2", replicas = 2
  - Production: image tag = "v3.1.0", replicas = 5

→ Phải SỬA TỪNG FILE cho mỗi môi trường ❌❌❌
```

## Helm = "App Store" của Kubernetes

> Giống như bạn cài app trên điện thoại:
> - Không cần biết app có bao nhiêu file
> - Chỉ cần nhấn "Install"
> - Muốn tuỳ chỉnh? Vào "Settings"

```
📱 Điện thoại:                   ☸️ Kubernetes:
App Store → Tìm "Zalo"          Helm → Tìm "nginx"
→ Nhấn Install                  → helm install nginx
→ Settings → Tuỳ chỉnh          → values.yaml → Tuỳ chỉnh
→ Update → Nhấn Update          → helm upgrade nginx
→ Xoá → Nhấn Delete             → helm uninstall nginx
```

## Cài Helm

```powershell
winget install Helm.Helm

# Kiểm tra
helm version
# → version.BuildInfo{Version:"v3.x.x"} ✅
```

## Dùng Helm cài app có sẵn (Người dùng)

```powershell
# 1. Thêm "cửa hàng" (repository)
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# 2. Tìm kiếm
helm search repo nginx
# → Thấy bitnami/nginx, bitnami/nginx-ingress-controller, ...

# 3. Cài đặt Nginx (1 lệnh = deploy hết tất cả!)
helm install my-website bitnami/nginx

# 4. Kiểm tra
helm list                    # Xem apps đã cài
kubectl get all              # Xem resources được tạo
# → Deployment, Service, Pod, ... TẤT CẢ tự tạo! 🪄

# 5. Tuỳ chỉnh khi cài
helm install my-website bitnami/nginx `
  --set replicaCount=3 `
  --set service.type=NodePort

# 6. Cập nhật
helm upgrade my-website bitnami/nginx --set replicaCount=5

# 7. Xoá sạch (1 lệnh = xoá tất cả resources)
helm uninstall my-website
```

## Tự tạo Helm Chart (Người phát triển)

> 🎯 Giờ bạn sẽ học **đóng gói app của riêng mình** thành Helm Chart

### Tạo chart mới

```powershell
# Helm tạo sẵn cấu trúc cho bạn
helm create my-app

# Cấu trúc được tạo:
# my-app/
# ├── Chart.yaml          ← Thông tin chart (tên, version)
# ├── values.yaml          ← Giá trị mặc định (CÁI QUAN TRỌNG NHẤT)
# ├── templates/           ← Các file YAML template
# │   ├── deployment.yaml
# │   ├── service.yaml
# │   ├── ingress.yaml
# │   ├── hpa.yaml
# │   └── _helpers.tpl     ← Hàm helper
# └── charts/              ← Dependency charts
```

### Hiểu `values.yaml` (File quan trọng nhất)

```yaml
# File: values.yaml
# Đây là "bảng cài đặt" - ai dùng chart cũng chỉ cần sửa file này

replicaCount: 2              # Số pods

image:
  repository: nginx          # Tên image
  tag: "1.25"                # Version
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilization: 70
```

### Hiểu Template (File YAML có biến)

```yaml
# File: templates/deployment.yaml
# {{ .Values.xxx }} = lấy giá trị từ values.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app    # Tên release + "-app"
spec:
  replicas: {{ .Values.replicaCount }}   # Lấy từ values.yaml
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

```
Giải thích đơn giản:
  {{ .Values.replicaCount }}    → Thay bằng giá trị "replicaCount" trong values.yaml
  {{ .Release.Name }}           → Tên khi bạn helm install <TÊN>
  {{ .Chart.Name }}             → Tên chart từ Chart.yaml
  {{- toYaml ... | nindent 12 }} → Copy nguyên block YAML, thụt vào 12 spaces
```

### Deploy chart của mình

```powershell
# Kiểm tra trước (dry run)
helm template my-release ./my-app
# → Hiển thị YAML đã được điền giá trị, KHÔNG tạo thật

# Deploy môi trường Dev
helm install dev-app ./my-app `
  --set replicaCount=1 `
  --set image.tag="dev-latest"

# Deploy môi trường Production (dùng file values riêng)
helm install prod-app ./my-app -f values-production.yaml
```

```yaml
# File: values-production.yaml (values riêng cho Production)
replicaCount: 5
image:
  tag: "v3.1.0"
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "1000m"
    memory: "1Gi"
autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
```

```
Kết quả:
  Dev:  1 pod,  image dev-latest, CPU 100m
  Prod: 5 pods, image v3.1.0,    CPU 500m, HPA bật
  → CÙNG 1 CHART, KHÁC giá trị! ✅
```

---

# 🔰 BÀI 9: ARGOCD - GITOPS (Deploy tự động từ Git)

## Câu chuyện: Deploy thủ công quá mệt

```
Quy trình cũ:
  1. Dev push code lên Git
  2. Build Docker image
  3. SSH vào server
  4. kubectl apply -f ...
  5. Kiểm tra pods
  6. Có lỗi → rollback thủ công

Vấn đề:
  ❌ Ai đã deploy? Deploy lúc nào? Version nào?
  ❌ 1 người deploy sai → cả team chịu
  ❌ Rollback phức tạp
  ❌ Không ai review YAML trước khi deploy
```

## ArgoCD = "Robot deploy tự động từ Git"

> **GitOps** = Git là "nguồn sự thật duy nhất". Mọi thứ trên cluster PHẢI giống với code trên Git.

```
Quy trình mới với ArgoCD:

  1. Dev sửa YAML trên Git → Tạo Pull Request
  2. Team review → Approve → Merge
  3. ArgoCD TỰ ĐỘNG phát hiện Git thay đổi
  4. ArgoCD TỰ ĐỘNG apply lên Kubernetes
  5. ArgoCD TỰ ĐỘNG kiểm tra app healthy
  6. Có lỗi → Git revert → ArgoCD TỰ ĐỘNG rollback

  → Không ai cần SSH vào server
  → Mọi thay đổi đều có lịch sử trên Git
  → Review trước khi deploy
```

```
┌────────────┐     push      ┌────────────┐
│  Developer │ ────────────► │    Git      │
│  sửa YAML  │               │  (GitHub)   │
└────────────┘               └─────┬──────┘
                                   │ watch
                                   ▼
                             ┌────────────┐
                             │  ArgoCD    │
                             │  (tự phát  │
                             │  hiện thay │
                             │  đổi)      │
                             └─────┬──────┘
                                   │ sync
                                   ▼
                             ┌────────────┐
                             │ Kubernetes │
                             │  Cluster   │
                             └────────────┘
```

## Cài ArgoCD

```powershell
# 1. Tạo namespace
kubectl create namespace argocd

# 2. Cài ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. Chờ pods sẵn sàng (~2 phút)
kubectl get pods -n argocd -w
# → Tất cả Running ✅

# 4. Lấy mật khẩu admin
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }

# 5. Mở ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# 6. Mở browser: https://localhost:8080
#    Username: admin
#    Password: (mật khẩu từ bước 4)
```

## Cài ArgoCD CLI (tuỳ chọn)

```powershell
# Cài CLI
winget install Argoproj.ArgoCD

# Login
argocd login localhost:8080 --username admin --password <password> --insecure
```

## Tạo Application đầu tiên

### Cách 1: Qua UI (dễ hiểu hơn)

```
1. Mở ArgoCD UI → New App
2. Điền:
   - Application Name: my-app
   - Project: default
   - Sync Policy: Automatic (tự deploy khi Git thay đổi)
   
   SOURCE:
   - Repository URL: https://github.com/YOUR_USERNAME/k8s-manifests.git
   - Path: apps/my-app     (thư mục chứa YAML)
   
   DESTINATION:
   - Cluster URL: https://kubernetes.default.svc
   - Namespace: default
   
3. Click CREATE
```

### Cách 2: Qua YAML

```yaml
# File: practices/09-argocd/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-web-app
  namespace: argocd
spec:
  project: default

  # SOURCE: Lấy YAML từ đâu?
  source:
    repoURL: https://github.com/YOUR_USERNAME/k8s-manifests.git
    targetRevision: main        # Branch
    path: apps/my-app           # Thư mục chứa YAML

  # DESTINATION: Deploy vào đâu?
  destination:
    server: https://kubernetes.default.svc
    namespace: default

  # SYNC POLICY: Deploy tự động hay thủ công?
  syncPolicy:
    automated:
      prune: true       # Xoá resource nếu bị xoá khỏi Git
      selfHeal: true    # Tự sửa nếu ai đó sửa tay trên cluster
    syncOptions:
      - CreateNamespace=true
```

```powershell
kubectl apply -f practices/09-argocd/application.yaml
```

### Cấu trúc Git repo cho ArgoCD

```
k8s-manifests/                    (Git repository)
├── apps/
│   ├── my-app/                   (ArgoCD app 1)
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   │
│   ├── api-server/               (ArgoCD app 2)
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── ingress.yaml
│   │
│   └── monitoring/               (ArgoCD app 3)
│       └── values.yaml           (Helm values)
│
└── README.md
```

## Quy trình deploy với ArgoCD

```
Ví dụ: Cập nhật image từ v1.0 → v2.0

Bước 1: Sửa deployment.yaml trên Git
         image: my-app:v1.0 → image: my-app:v2.0

Bước 2: Push lên Git

Bước 3: ArgoCD phát hiện (trong vòng 3 phút)
         Trạng thái: "OutOfSync" (Git ≠ Cluster)

Bước 4: ArgoCD tự động sync
         → kubectl apply deployment mới
         → Rolling update (không downtime)

Bước 5: Trạng thái: "Synced" + "Healthy" ✅

--- Nếu có lỗi ---
Bước 6: Git revert commit → ArgoCD tự rollback ↩️
```

### Xem trạng thái trên ArgoCD UI

```
🟢 Synced + Healthy     → Mọi thứ OK
🟡 OutOfSync            → Git khác cluster, đang chờ sync
🔴 Degraded             → App có vấn đề (pod crash, etc)
⚪ Unknown              → Không check được trạng thái
```

---

# 🔰 BÀI 10: ISTIO - SERVICE MESH

## Câu chuyện: Khi app có nhiều microservices

```
App của bạn gồm 10 services:
  User Service → Order Service → Payment Service
                               → Inventory Service
                → Notification Service
                → ...

Vấn đề:
  ❓ Service A gọi Service B bị chậm → lỗi do đâu?
  ❓ Làm sao chuyển 10% traffic sang version mới để test?
  ❓ Làm sao mã hoá traffic giữa các service?
  ❓ Service nào gọi service nào? Bao nhiêu request/giây?
  ❓ Làm sao retry tự động khi request thất bại?
```

## Istio = "Quản lý giao thông" cho Microservices

> Hãy tưởng tượng thành phố có nhiều con đường (services). Istio là **hệ thống đèn giao thông + camera + cảnh sát giao thông** tự động.

```
Không có Istio:                    Có Istio:

Service A ──► Service B            Service A ──► [Sidecar Proxy]
(gọi trực tiếp,                          │
 không kiểm soát)                         ▼
                                   [Sidecar Proxy] ──► Service B
                                   
                                   Istio Control Plane theo dõi:
                                   ✅ Bao nhiêu requests?
                                   ✅ Latency bao lâu?
                                   ✅ Error rate bao nhiêu %?
                                   ✅ Mã hoá TLS tự động
                                   ✅ Retry khi lỗi
                                   ✅ Canary deployment
```

### Sidecar Proxy là gì?

```
Mỗi Pod được Istio gắn thêm 1 container phụ (sidecar):

┌─────────────────────────────┐
│           POD               │
│  ┌───────────┐ ┌─────────┐ │
│  │  Your App │ │ Envoy   │ │ ← Sidecar proxy (Istio tự thêm)
│  │  (nginx)  │ │ Proxy   │ │
│  └─────┬─────┘ └────┬────┘ │
│        └──────┬──────┘      │
│               │             │
└───────────────┼─────────────┘
                │
        Tất cả traffic đi qua Envoy Proxy
        → Istio theo dõi & kiểm soát
```

## Cài Istio

```powershell
# 1. Tải Istio CLI
# Windows: download từ https://github.com/istio/istio/releases
# Hoặc:
winget install Istio.Istioctl

# 2. Cài Istio lên cluster (profile demo = đủ tính năng để học)
istioctl install --set profile=demo -y

# 3. Bật auto-inject sidecar cho namespace default
kubectl label namespace default istio-injection=enabled

# → Từ giờ, TẤT CẢ pod trong namespace "default"
#   sẽ tự động được thêm Envoy sidecar proxy

# 4. Kiểm tra
kubectl get pods -n istio-system
# → istiod, istio-ingressgateway, ... đều Running ✅
```

## Cài Kiali (Dashboard cho Istio)

```powershell
# Kiali = Dashboard hiển thị traffic giữa các services
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/kiali.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.21/samples/addons/prometheus.yaml

# Mở Kiali
istioctl dashboard kiali

# → Thấy bản đồ traffic giữa các services! 🗺️
```

## Tính năng 1: Traffic Splitting (Canary Deployment)

```
Bạn có app v1 đang chạy. Muốn thử v2 nhưng sợ lỗi.
→ Chuyển 10% traffic sang v2, 90% vẫn ở v1
→ Nếu v2 OK → tăng dần lên 50%, rồi 100%
→ Nếu v2 lỗi → quay lại 100% v1
```

```yaml
# File: practices/10-istio/virtual-service.yaml

# Bước 1: Định nghĩa destination (các version)
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app-destination
spec:
  host: my-app-service
  subsets:
    - name: v1
      labels:
        version: v1      # Pods có label version=v1
    - name: v2
      labels:
        version: v2      # Pods có label version=v2

---
# Bước 2: Chia traffic
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app-routing
spec:
  hosts:
    - my-app-service
  http:
    - route:
        - destination:
            host: my-app-service
            subset: v1
          weight: 90          # 90% traffic → v1
        - destination:
            host: my-app-service
            subset: v2
          weight: 10          # 10% traffic → v2 (thử nghiệm)
```

```powershell
# Dần dần tăng traffic cho v2
# Sửa weight: v1=50, v2=50 → rồi v1=0, v2=100
kubectl apply -f practices/10-istio/virtual-service.yaml
```

## Tính năng 2: Retry & Timeout tự động

```yaml
# File: practices/10-istio/retry-policy.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-app-resilience
spec:
  hosts:
    - my-app-service
  http:
    - route:
        - destination:
            host: my-app-service
      timeout: 5s              # Timeout sau 5 giây
      retries:
        attempts: 3            # Retry 3 lần nếu lỗi
        perTryTimeout: 2s      # Mỗi lần retry timeout 2s
        retryOn: 5xx           # Chỉ retry khi server error (500, 502, ...)
```

## Tính năng 3: Circuit Breaker (Ngắt mạch)

```
Giống cầu chì điện:
  - Service B đang chết (trả lỗi 500 liên tục)
  - Nếu Service A cứ gọi B → A cũng chậm theo
  - Circuit Breaker: sau 5 lỗi liên tiếp → NGẮT
    → Trả lỗi ngay, không gọi B nữa
    → Sau 30 giây → thử lại 1 request
    → Nếu OK → mở lại kết nối
```

```yaml
# File: practices/10-istio/circuit-breaker.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-app-circuit-breaker
spec:
  host: my-app-service
  trafficPolicy:
    connectionPool:
      http:
        h2UpgradePolicy: DEFAULT
        http1MaxPendingRequests: 100   # Tối đa 100 request chờ
        http2MaxRequests: 1000         # Tối đa 1000 request đồng thời
    outlierDetection:
      consecutive5xxErrors: 5          # Sau 5 lỗi 500 liên tiếp
      interval: 30s                    # Kiểm tra mỗi 30 giây
      baseEjectionTime: 30s           # Ngắt trong 30 giây
      maxEjectionPercent: 50           # Tối đa ngắt 50% pods
```

---

# 🔰 BÀI 11: CI/CD PIPELINE - TỰ ĐỘNG HOÁ HOÀN TOÀN

## Bức tranh toàn cảnh

```
Developer push code
       │
       ▼
  ┌─────────────────────────────────────────────────┐
  │              CI/CD PIPELINE                     │
  │                                                 │
  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
  │  │    CI     │  │  Build   │  │     CD       │  │
  │  │  (Test)   │→│ (Docker) │→│  (Deploy)    │  │
  │  │          │  │          │  │              │  │
  │  │ - lint   │  │ - build  │  │ - update Git │  │
  │  │ - test   │  │   image  │  │   manifest   │  │
  │  │ - scan   │  │ - push   │  │ - ArgoCD     │  │
  │  │          │  │   to ECR │  │   auto sync  │  │
  │  └──────────┘  └──────────┘  └──────────────┘  │
  └─────────────────────────────────────────────────┘
       │
       ▼
  Kubernetes Cluster (app chạy version mới)
```

## CI/CD là gì?

```
CI = Continuous Integration (Tích hợp liên tục)
  → Mỗi khi push code → TỰ ĐỘNG test, build

CD = Continuous Delivery/Deployment (Triển khai liên tục)
  → Sau khi CI pass → TỰ ĐỘNG deploy lên server

Ví dụ:
  09:00 - Dev push code lên Git
  09:01 - CI: Chạy unit tests → PASS ✅
  09:02 - CI: Build Docker image → OK ✅
  09:03 - CI: Push image lên registry → OK ✅
  09:04 - CD: Update YAML (image tag mới) → OK ✅
  09:05 - ArgoCD: Sync lên cluster → App chạy version mới ✅

  → 5 PHÚT từ push code → app chạy trên production!
  → KHÔNG AI cần làm gì thủ công!
```

## GitHub Actions + Kubernetes

### Cấu trúc project

```
my-project/
├── .github/
│   └── workflows/
│       └── ci-cd.yaml           ← Pipeline definition
├── src/                         ← Source code
│   ├── index.js
│   ├── package.json
│   └── ...
├── Dockerfile                   ← Build instructions
├── k8s-manifests/               ← Kubernetes YAML (hoặc Git repo riêng)
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ...
└── README.md
```

### Dockerfile

```dockerfile
# File: Dockerfile
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY src/ ./src/

EXPOSE 3000
CMD ["node", "src/index.js"]
```

### GitHub Actions Pipeline

```yaml
# File: .github/workflows/ci-cd.yaml
name: CI/CD Pipeline

# Khi nào chạy?
on:
  push:
    branches: [main]        # Push vào main → chạy
  pull_request:
    branches: [main]        # Tạo PR vào main → chạy (chỉ CI)

# Biến môi trường
env:
  AWS_REGION: ap-southeast-1
  ECR_REPOSITORY: my-app
  IMAGE_TAG: ${{ github.sha }}    # Dùng commit hash làm tag

jobs:
  # ==================== JOB 1: CI (Test) ====================
  test:
    name: "🧪 Run Tests"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

  # ==================== JOB 2: Build & Push Docker ====================
  build:
    name: "🐳 Build & Push Image"
    needs: test               # Chỉ chạy SAU KHI test PASS
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'   # Chỉ chạy trên branch main

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

  # ==================== JOB 3: CD (Update manifest) ====================
  deploy:
    name: "🚀 Deploy to Kubernetes"
    needs: build              # Chỉ chạy SAU KHI build PASS
    runs-on: ubuntu-latest

    steps:
      - name: Checkout manifest repo
        uses: actions/checkout@v4
        with:
          repository: YOUR_USERNAME/k8s-manifests
          token: ${{ secrets.GIT_TOKEN }}

      - name: Update image tag in deployment
        run: |
          # Sửa image tag trong deployment.yaml
          sed -i "s|image: .*my-app:.*|image: 123456789.dkr.ecr.ap-southeast-1.amazonaws.com/my-app:${{ github.sha }}|" apps/my-app/deployment.yaml

      - name: Commit and push
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .
          git commit -m "🚀 Update my-app to ${{ github.sha }}"
          git push

      # ArgoCD sẽ TỰ ĐỘNG phát hiện thay đổi và deploy! 🪄
```

### Giải thích luồng chạy

```
Developer push code lên branch "main"
         │
         ▼
📋 Job 1: TEST
   ├── Checkout code
   ├── npm ci (cài dependencies)
   ├── npm run lint (kiểm tra code style)
   └── npm test (chạy unit tests)
         │ PASS ✅
         ▼
🐳 Job 2: BUILD
   ├── Login vào AWS ECR (Docker registry)
   ├── docker build (tạo Docker image)
   └── docker push (đẩy image lên ECR)
         │ PASS ✅
         ▼
🚀 Job 3: DEPLOY
   ├── Checkout k8s-manifests repo
   ├── Sửa image tag mới trong deployment.yaml
   └── Git push → ArgoCD tự sync → App updated!
```

## Thiết lập GitHub Secrets

```
GitHub repo → Settings → Secrets and variables → Actions → New secret

Thêm:
  AWS_ACCESS_KEY_ID     → Access key từ AWS IAM
  AWS_SECRET_ACCESS_KEY → Secret key từ AWS IAM
  GIT_TOKEN             → Personal Access Token (để push manifest repo)
```

## So sánh tổng thể

| Công cụ | Vai trò | Ví dụ dễ hiểu |
|---|---|---|
| **Helm** | Đóng gói app K8s | Như file .apk cho Android |
| **ArgoCD** | Deploy tự động từ Git | Như Shopee tự giao hàng khi bạn đặt |
| **Istio** | Quản lý traffic giữa services | Như đèn giao thông + camera |
| **GitHub Actions** | CI/CD pipeline | Như dây chuyền sản xuất tự động |

## Kết hợp tất cả

```
┌──────────────────────────────────────────────────────┐
│                PRODUCTION SETUP                      │
│                                                      │
│  Developer ──push──► GitHub ──CI──► Docker Image     │
│                         │                   │        │
│                         │ CD                │ push   │
│                         ▼                   ▼        │
│                    Git Manifests ◄──── ECR Registry   │
│                         │                            │
│                         │ watch                      │
│                         ▼                            │
│                      ArgoCD                          │
│                         │ sync                       │
│                         ▼                            │
│  ┌──────────── Kubernetes (EKS) ──────────────┐     │
│  │                                             │     │
│  │   Istio (traffic management)                │     │
│  │     │                                       │     │
│  │     ├── App v1 (90% traffic)                │     │
│  │     └── App v2 (10% traffic - canary)       │     │
│  │                                             │     │
│  │   Karpenter (auto-scale nodes)              │     │
│  │   HPA (auto-scale pods)                     │     │
│  │                                             │     │
│  │   Prometheus + Grafana (monitoring)         │     │
│  │                                             │     │
│  └─────────────────────────────────────────────┘     │
│                                                      │
│  Terraform (quản lý tất cả hạ tầng bằng code)       │
└──────────────────────────────────────────────────────┘
```

---

## 📎 Link tham khảo Tuần 4

| Tài liệu | Link |
|---|---|
| Helm Docs | https://helm.sh/docs/ |
| ArgoCD Docs | https://argo-cd.readthedocs.io/ |
| Istio Docs | https://istio.io/latest/docs/ |
| GitHub Actions | https://docs.github.com/en/actions |
| Kiali (Istio Dashboard) | https://kiali.io/docs/ |
