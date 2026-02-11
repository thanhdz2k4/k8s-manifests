# 🎓 Lớp Học Kubernetes - Từ Zero Đến Hero

> **Xin chào! Mình là thầy giáo của bạn.** Mình sẽ dạy bạn từng bước một, giải thích bằng ví dụ thực tế. Đừng lo nếu chưa biết gì - mình sẽ bắt đầu từ những thứ đơn giản nhất.

---

# 🔰 BÀI 1: HIỂU KUBERNETES LÀ GÌ (Không cần code)

## Câu chuyện: Anh Minh mở quán phở

Hãy tưởng tượng **anh Minh** mở quán phở online (website bán phở giao tận nhà).

### Giai đoạn 1: Khởi nghiệp - Chỉ 1 máy tính

```
Anh Minh thuê 1 server → chạy website trên đó
Khách ít (10 người/ngày) → OK, chạy tốt ✅
```

### Giai đoạn 2: Nổi tiếng - Vấn đề xuất hiện

```
Lên TikTok viral → 10,000 khách đổ vào cùng lúc
1 server không chịu nổi → Website sập ❌
Anh Minh phải:
  - Thuê thêm server thủ công
  - Cài đặt lại từ đầu trên từng server
  - Nếu 1 server chết → tự mình phát hiện và sửa
  - Cập nhật code → phải vào TỪNG server để update
```

**Quá mệt! Quá tốn thời gian!** 😩

### Giai đoạn 3: Dùng Kubernetes - Giải quyết tất cả

```
Anh Minh dùng Kubernetes:
  ✅ Website tự chạy trên nhiều server
  ✅ Server chết → K8s tự khởi động lại
  ✅ Đông khách → K8s tự thêm server
  ✅ Vắng khách → K8s tự giảm server (tiết kiệm tiền)
  ✅ Cập nhật code → 1 lệnh, K8s tự update tất cả
```

> 💡 **Vậy Kubernetes = "Người quản lý tự động" cho các ứng dụng của bạn.**

---

## Hiểu Kubernetes qua ví dụ "Khách sạn"

Hãy nghĩ Kubernetes như một **khách sạn lớn**:

```
🏨 KHÁCH SẠN (= Kubernetes Cluster)
│
├── 👔 BAN GIÁM ĐỐC (= Control Plane)
│   │
│   ├── 📞 Lễ tân (= API Server)
│   │     → Nhận yêu cầu từ khách (developer)
│   │     → "Tôi muốn đặt 3 phòng loại A"
│   │
│   ├── 📋 Quản lý phòng (= Scheduler)
│   │     → Tìm tầng nào còn phòng trống
│   │     → Phân bổ khách vào phòng phù hợp
│   │
│   ├── 🔍 Giám sát (= Controller)
│   │     → Kiểm tra phòng có sạch không
│   │     → Nếu phòng hỏng → chuyển khách sang phòng khác
│   │
│   └── 📚 Sổ sách (= etcd)
│         → Ghi nhớ tất cả thông tin
│         → "Phòng 101: khách A, phòng 102: khách B..."
│
├── 🏢 TẦNG 1 (= Node 1)
│   ├── 🚪 Phòng 101 (= Pod) → Khách A ở đây
│   └── 🚪 Phòng 102 (= Pod) → Khách B ở đây
│
├── 🏢 TẦNG 2 (= Node 2)
│   ├── 🚪 Phòng 201 (= Pod) → Khách C ở đây
│   └── 🚪 Phòng 202 (= Pod) → Khách D ở đây
│
└── 🏢 TẦNG 3 (= Node 3)
    └── 🚪 Phòng 301 (= Pod) → Khách E ở đây
```

### Bảng so sánh dễ hiểu

| Khách sạn | Kubernetes | Giải thích đơn giản |
|---|---|---|
| 🏨 Khách sạn | **Cluster** | Toàn bộ hệ thống |
| 🏢 Tầng lầu | **Node** | 1 máy tính (vật lý hoặc ảo) |
| 🚪 Phòng | **Pod** | Nơi app của bạn chạy |
| 👔 Ban giám đốc | **Control Plane** | Bộ não điều khiển mọi thứ |
| 📞 Lễ tân | **API Server** | Nơi bạn gửi yêu cầu |
| 📋 Quản lý phòng | **Scheduler** | Quyết định app chạy ở đâu |
| 🔍 Giám sát | **Controller** | Đảm bảo mọi thứ chạy đúng |
| 📢 Biển chỉ dẫn | **Service** | Giúp mọi người tìm đến đúng phòng |

---

# 🔰 BÀI 2: CÀI ĐẶT VÀ CHẠY THỬ LẦN ĐẦU

## Bước 1: Cài đặt công cụ

> 🎯 Mình sẽ dùng **Minikube** - nó tạo 1 Kubernetes cluster giả lập NGAY trên máy tính của bạn. Miễn phí, không cần AWS.

### Yêu cầu: Đã cài Docker Desktop

Nếu chưa có Docker Desktop → tải tại: https://www.docker.com/products/docker-desktop/

### Cài Minikube và kubectl

Mở **PowerShell** (Run as Administrator) và chạy lần lượt:

```powershell
# Bước 1: Cài Minikube (công cụ tạo cluster trên máy bạn)
winget install Kubernetes.minikube

# Bước 2: Cài kubectl (công cụ ra lệnh cho Kubernetes)
winget install Kubernetes.kubectl

# Bước 3: Đóng PowerShell và mở lại để nhận PATH mới
```

### Khởi tạo cluster đầu tiên

```powershell
# Tạo cluster (giống như "xây khách sạn")
minikube start --driver=docker

# Kiểm tra xem cluster đã sẵn sàng chưa
kubectl cluster-info

# Bạn sẽ thấy dạng:
# Kubernetes control plane is running at https://127.0.0.1:xxxxx
# → Nghĩa là OK! ✅
```

### Xem Node (tầng lầu)

```powershell
kubectl get nodes

# Kết quả:
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   1m    v1.28.3
#
# → Bạn có 1 node tên "minikube", trạng thái "Ready" (sẵn sàng)
```

---

## Bước 2: Tạo Pod đầu tiên (Đặt phòng đầu tiên)

> 🎯 **Pod** = nơi app của bạn chạy. Giống như đặt 1 phòng khách sạn.

Mình sẽ chạy **Nginx** (một web server đơn giản) trong Pod.

```powershell
# Chạy file: practices/01-basic-pod/pod.yaml
kubectl apply -f practices/01-basic-pod/pod.yaml
```

File `pod.yaml` này nói với Kubernetes:

```yaml
# "Này Kubernetes, hãy tạo cho tôi 1 Pod"
apiVersion: v1          # Phiên bản API
kind: Pod               # Loại: Pod (= 1 phòng)
metadata:
  name: my-first-pod    # Tên phòng: "my-first-pod"
  labels:
    app: nginx          # Nhãn dán: "đây là phòng chạy nginx"
spec:
  containers:           # Bên trong phòng có gì?
    - name: nginx       # Có 1 container tên "nginx"
      image: nginx:latest  # Dùng phần mềm Nginx mới nhất
      ports:
        - containerPort: 80  # Mở cửa số 80 (cổng web)
```

### Kiểm tra Pod

```powershell
# Xem danh sách pods (xem phòng nào đang có khách)
kubectl get pods

# Kết quả:
# NAME           READY   STATUS    RESTARTS   AGE
# my-first-pod   1/1     Running   0          30s
#
# STATUS = Running → App đang chạy! ✅
# READY = 1/1 → 1 container, cả 1 đều sẵn sàng ✅
```

```powershell
# Xem chi tiết pod (xem thông tin phòng)
kubectl describe pod my-first-pod

# → Hiển thị: IP, Node nào đang chạy, events (nhật ký)
```

### Truy cập website trong Pod

```powershell
# Mở đường hầm từ máy bạn → Pod
kubectl port-forward my-first-pod 8080:80

# Giải thích:
# port 8080 (máy bạn) ──► port 80 (trong Pod)
#
# Giờ mở browser → http://localhost:8080
# Bạn sẽ thấy trang "Welcome to nginx!" 🎉
```

> 💡 **Nhấn Ctrl+C** để tắt port-forward

### Xem log (nhật ký hoạt động)

```powershell
# Xem pod đã làm gì
kubectl logs my-first-pod

# Xem log realtime (theo dõi trực tiếp)
kubectl logs -f my-first-pod
# Nhấn Ctrl+C để thoát
```

### Dọn dẹp

```powershell
kubectl delete pod my-first-pod
# → Xóa pod (trả phòng)
```

---

## Bước 3: Tạo Deployment (Đặt NHIỀU phòng cùng lúc)

> 🎯 **Vấn đề:** Nếu Pod chết → app chết luôn, không ai khởi động lại.
>
> **Giải pháp: Deployment** = Bạn nói "tôi muốn LUÔN CÓ 3 phòng chạy nginx". Nếu 1 phòng chết → Kubernetes tự tạo phòng mới thay thế.

```powershell
# Chạy file deployment
kubectl apply -f practices/01-basic-pod/deployment.yaml
```

File `deployment.yaml` giải thích:

```yaml
apiVersion: apps/v1
kind: Deployment         # Loại: Deployment (quản lý nhiều Pod)
metadata:
  name: nginx-deployment
spec:
  replicas: 3            # ⭐ "Tôi muốn LUÔN CÓ 3 Pod chạy"
  selector:
    matchLabels:
      app: nginx         # Quản lý các Pod có nhãn "app: nginx"
  template:              # Mẫu để tạo Pod
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          resources:
            requests:     # "Mỗi Pod CẦN ÍT NHẤT chừng này"
              cpu: "100m"     # 0.1 CPU (m = millicpu)
              memory: "128Mi" # 128 MB RAM
            limits:       # "Mỗi Pod KHÔNG ĐƯỢC VƯỢT QUÁ"
              cpu: "250m"     # 0.25 CPU
              memory: "256Mi" # 256 MB RAM
```

### Kiểm tra

```powershell
kubectl get deployments
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           1m
#                    ↑ 3/3 = 3 Pod đang chạy / 3 Pod mong muốn ✅

kubectl get pods
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-xxxxx-aaaaa        1/1     Running   0          1m
# nginx-deployment-xxxxx-bbbbb        1/1     Running   0          1m
# nginx-deployment-xxxxx-ccccc        1/1     Running   0          1m
# → 3 pods đang chạy! ✅
```

### Thử nghiệm: Xóa 1 Pod, xem K8s tự khởi động lại

```powershell
# Xóa 1 pod (giả lập Pod chết)
kubectl delete pod nginx-deployment-xxxxx-aaaaa
# (thay xxxxx-aaaaa bằng tên pod thật của bạn)

# Xem ngay lập tức
kubectl get pods
# → Kubernetes TỰ ĐỘNG tạo 1 Pod mới thay thế! 🪄
# → Luôn giữ đúng 3 Pod như bạn yêu cầu
```

### Thử nghiệm: Scale (tăng/giảm số Pod)

```powershell
# Tăng lên 5 Pod
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods
# → 5 pods! ✅

# Giảm xuống 2 Pod
kubectl scale deployment nginx-deployment --replicas=2
kubectl get pods
# → 2 pods! ✅ (3 pods bị terminate)
```

---

## Bước 4: Tạo Service (Biển chỉ dẫn)

> 🎯 **Vấn đề:** Mỗi Pod có IP riêng, và IP thay đổi khi Pod restart. Vậy làm sao client tìm đến đúng Pod?
>
> **Giải pháp: Service** = Một địa chỉ CỐ ĐỊNH, tự động chuyển request đến Pod đang chạy.

```
Không có Service:              Có Service:
                               
Client → Pod IP:10.0.0.5       Client → Service (IP cố định)
Pod chết, IP mới: 10.0.0.9           ↓ (tự động)
Client → ??? Không biết        ┌─────┼─────┐
         IP mới ❌             Pod1  Pod2  Pod3
                               (Load Balance tự động) ✅
```

```powershell
kubectl apply -f practices/01-basic-pod/service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort        # Kiểu: mở port trên Node
  selector:
    app: nginx          # "Tìm tất cả Pod có nhãn app=nginx"
  ports:
    - port: 80          # Port của Service
      targetPort: 80    # Port trong container
      nodePort: 30080   # Port mở trên máy bạn (30000-32767)
```

```powershell
# Với Minikube, truy cập service
minikube service nginx-service

# Hoặc dùng port-forward
kubectl port-forward service/nginx-service 8080:80
# → Mở http://localhost:8080
```

### Dọn dẹp bài 2

```powershell
kubectl delete -f practices/01-basic-pod/
```

---

# 🔰 BÀI 3: CONFIGMAP & SECRET (Cấu hình ứng dụng)

## Vấn đề thực tế

```
App của bạn cần kết nối database.
Thông tin kết nối: host, port, username, password

❌ SAI: Ghi cứng trong code → Đổi config phải build lại app
❌ SAI: Commit password lên Git → Lộ mật khẩu

✅ ĐÚNG: Dùng ConfigMap (config thường) + Secret (mật khẩu)
```

## ConfigMap = Bảng thông tin công khai

> Giống bảng thông báo ở sảnh khách sạn: "WiFi: lobby_wifi, Giờ ăn sáng: 6-9h"

```powershell
kubectl apply -f practices/02-config/configmap.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config          # Tên bảng thông báo
data:                       # Nội dung:
  DATABASE_HOST: "postgres-service"
  DATABASE_PORT: "5432"
  DATABASE_NAME: "mydb"
  APP_ENV: "production"
```

## Secret = Két sắt an toàn

> Giống két sắt trong phòng: chỉ khách (Pod) mới mở được.

```powershell
kubectl apply -f practices/02-config/secret.yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  # Giá trị phải mã hóa base64
  username: YWRtaW4=                 # "admin" → base64
  password: c3VwZXJzZWNyZXQxMjM=    # "supersecret123" → base64
```

```powershell
# Cách tạo base64 trên PowerShell:
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("admin"))
# → YWRtaW4=
```

## Sử dụng trong Pod

```powershell
kubectl apply -f practices/02-config/app-deployment.yaml
```

```yaml
spec:
  containers:
    - name: app
      image: nginx:latest
      envFrom:
        - configMapRef:
            name: app-config       # Load TẤT CẢ key từ ConfigMap
      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-credentials  # Load từ Secret
              key: username
```

### Kiểm tra Pod đã nhận config chưa

```powershell
# Vào bên trong Pod
kubectl exec -it <pod-name> -- bash

# In ra biến môi trường
echo $DATABASE_HOST    # → postgres-service
echo $DB_USERNAME      # → admin
echo $APP_ENV          # → production

# Thoát
exit
```

### Dọn dẹp

```powershell
kubectl delete -f practices/02-config/
```

---

# 🔰 BÀI 4: TERRAFORM - TẠO HẠ TẦNG BẰNG CODE

## Câu chuyện tiếp

```
Anh Minh muốn chạy K8s trên AWS (Amazon Web Services).
Anh cần tạo:
  - VPC (mạng riêng)
  - Subnet (phân vùng mạng)
  - EKS (Kubernetes của AWS)
  - IAM Roles (quyền truy cập)
  - Security Groups (tường lửa)
  - ... rất nhiều thứ

Cách 1: Click từng cái trên AWS Console → MẤT 2-3 TIẾNG, DỄ SAI ❌
Cách 2: Viết Terraform code → Chạy 1 lệnh → TỰ TẠO HẾT ✅
```

## Terraform = "Bản vẽ kỹ thuật" cho hạ tầng

```
Xây nhà thật:                    Xây hạ tầng cloud:
1. Kiến trúc sư vẽ bản vẽ       1. Viết file .tf (Terraform)
2. Đưa thợ xây                  2. Chạy "terraform apply"
3. Thợ xây theo bản vẽ          3. Terraform tự tạo trên AWS
4. Sửa bản vẽ → sửa nhà         4. Sửa code → terraform apply
5. Bản vẽ lưu trữ               5. Code lưu trên Git
```

## Cài đặt Terraform

```powershell
winget install HashiCorp.Terraform
terraform version
# → Terraform v1.x.x ✅
```

## 4 Lệnh DUY NHẤT cần nhớ

```powershell
terraform init      # 📦 Tải plugins cần thiết (chạy 1 lần đầu)
terraform plan      # 👀 Xem trước: "sẽ tạo/sửa/xóa những gì?"
terraform apply     # 🚀 Thực thi: tạo hạ tầng thật
terraform destroy   # 💣 Xóa tất cả (cẩn thận!)
```

## Ví dụ đơn giản nhất

```hcl
# File: main.tf
# "Tôi muốn tạo 1 server EC2 trên AWS"

provider "aws" {
  region = "ap-southeast-1"    # Chọn vùng Singapore
}

resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"   # Hệ điều hành Amazon Linux
  instance_type = "t3.micro"                 # Loại máy nhỏ (miễn phí)

  tags = {
    Name = "My-First-Server"
  }
}
```

```powershell
terraform init       # Tải AWS plugin
terraform plan       # Xem: "Sẽ tạo 1 EC2 instance"
terraform apply      # Nhấn "yes" → Server được tạo trên AWS!
terraform destroy    # Xóa server (để không bị tính phí)
```

## Tạo EKS Cluster bằng Terraform

> 📁 Xem đầy đủ tại: `practices/06-terraform-eks/`

Terraform sẽ tự động tạo:

```
terraform apply
    │
    ├── VPC (mạng riêng cho cluster)
    │   ├── 3 Public Subnets
    │   └── 3 Private Subnets
    │
    ├── NAT Gateway (cho private subnet ra internet)
    │
    ├── EKS Cluster (Kubernetes trên AWS)
    │   └── 2 Worker Nodes (t3.medium)
    │
    └── IAM Roles & Security Groups

⏱️ Mất khoảng 15-20 phút
💰 CHÚ Ý: EKS CÓ PHÍ! ~$0.10/giờ cho cluster + EC2 instances
```

```powershell
# Sau khi terraform apply xong:
aws eks update-kubeconfig --region ap-southeast-1 --name my-eks-cluster
kubectl get nodes
# → Bạn sẽ thấy 2 nodes trên AWS! 🎉
```

---

# 🔰 BÀI 5: KARPENTER - TỰ ĐỘNG THÊM/BỚT MÁY

## Vấn đề

```
Bạn có EKS cluster với 2 nodes.
Mỗi node có 2 CPU, 4GB RAM.

Đột nhiên cần deploy 20 pods, mỗi pod cần 1 CPU.
→ 2 nodes chỉ đủ cho 4 pods (2 CPU × 2 nodes)
→ 16 pods còn lại: PENDING (chờ, không chạy được) ❌

VẬY AI SẼ TẠO THÊM NODES?
```

## 2 giải pháp

```
Giải pháp cũ: Cluster Autoscaler (CAS)
  - Dùng "Node Group" cố định (vd: chỉ dùng t3.medium)
  - Chậm: mất 3-5 phút để thêm node
  - Không thông minh trong việc chọn instance

Giải pháp mới: Karpenter ⭐
  - TỰ CHỌN loại instance tối ưu nhất
  - Nhanh: 30-60 giây
  - Tự dùng Spot Instance (rẻ hơn 70%)
  - Tự "gom" pods khi vắng → giảm node → tiết kiệm tiền
```

## Karpenter hoạt động thế nào?

```
Bước 1: Bạn deploy app, cần 10 CPU
        ↓
Bước 2: Kubernetes: "Không đủ chỗ! 6 pods đang Pending"
        ↓
Bước 3: Karpenter phát hiện pods Pending
        ↓
Bước 4: Karpenter tính toán: "Cần 6 CPU nữa"
        → Chọn 2 x c5.xlarge (4 CPU mỗi cái)
        → Hoặc 1 x c5.2xlarge (8 CPU) ← tuỳ cái nào rẻ hơn
        ↓
Bước 5: Tạo EC2 instance mới (~30 giây)
        ↓
Bước 6: Pods được schedule vào nodes mới ✅
        ↓
--- 2 tiếng sau, traffic giảm ---
        ↓
Bước 7: Karpenter: "Node này chỉ dùng 10% CPU, lãng phí!"
        → Di chuyển pods sang node khác
        → Terminate node trống
        → Tiết kiệm tiền! 💰
```

## Cấu hình Karpenter

> 📁 Xem file: `practices/07-karpenter/`

```yaml
# nodepool.yaml - "Hãy cho Karpenter biết nó được tạo loại máy nào"
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        # Cho phép dùng các loại instance:
        # c = Compute (nhiều CPU)
        # m = Memory (cân bằng)
        # r = RAM (nhiều RAM)
        - key: karpenter.k8s.aws/instance-category
          operator: In
          values: ["c", "m", "r"]

        # Kích cỡ cho phép
        - key: karpenter.k8s.aws/instance-size
          operator: In
          values: ["medium", "large", "xlarge"]

        # Spot = giá rẻ (nhưng có thể bị thu hồi)
        # On-Demand = giá đầy đủ (luôn có)
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]

  # Giới hạn: tối đa 100 CPU và 400GB RAM
  limits:
    cpu: "100"
    memory: "400Gi"

  # Khi node ít việc → tự gom pods và xoá node
  disruption:
    consolidationPolicy: WhenUnderutilized
```

## Test thử Karpenter

```powershell
# 1. Apply stress test deployment (bắt đầu với 0 pods)
kubectl apply -f practices/07-karpenter/stress-test.yaml

# 2. Tăng lên 10 pods (mỗi pod cần 1 CPU)
kubectl scale deployment inflate --replicas=10

# 3. Xem Karpenter tạo nodes mới (xem realtime)
kubectl get nodes -w
# → Sẽ thấy nodes mới xuất hiện trong ~30-60s! 🪄

# 4. Xem log của Karpenter
kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter -f

# 5. Giảm xuống 0 → Karpenter xóa nodes
kubectl scale deployment inflate --replicas=0
kubectl get nodes -w
# → Nodes bị terminate sau vài phút 💰
```

---

# 🔰 BÀI 6: MONITORING - XEM LOG BẰNG GRAFANA & PROMETHEUS

## Tại sao cần monitoring?

```
Không có monitoring:
  → App chết lúc 3 giờ sáng
  → Sáng dậy khách hàng phàn nàn
  → Debug từ đầu, không biết lỗi ở đâu 😱

Có monitoring:
  → Prometheus: theo dõi CPU, Memory, Request 24/7
  → Grafana: hiển thị biểu đồ đẹp, dễ đọc
  → AlertManager: CPU > 80% → tự gửi thông báo Slack/Email
  → Bạn phát hiện và sửa lỗi TRƯỚC KHI khách hàng bị ảnh hưởng 😎
```

## Prometheus = "Camera giám sát" 📹

```
Prometheus cứ mỗi 15 giây sẽ:
  1. Hỏi mỗi Pod: "CPU bao nhiêu? RAM bao nhiêu? Bao nhiêu request?"
  2. Ghi lại số liệu (metrics) vào database
  3. Lưu trữ 15-30 ngày

→ Bạn có thể xem lại: "3 ngày trước lúc 14:00, CPU là bao nhiêu?"
```

## Grafana = "Màn hình TV hiển thị camera" 📺

```
Grafana lấy dữ liệu từ Prometheus → vẽ thành biểu đồ đẹp:
  - Đường cong CPU theo thời gian
  - Bảng danh sách pods
  - Gauge hiển thị % memory
  - Alert khi vượt ngưỡng
```

## Cài đặt (trên Minikube)

```powershell
# 1. Cài Helm (package manager cho K8s)
winget install Helm.Helm

# 2. Thêm repository chứa Prometheus + Grafana
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# 3. Cài đặt bộ monitoring (Prometheus + Grafana + AlertManager)
helm install monitoring prometheus-community/kube-prometheus-stack `
  --namespace monitoring `
  --create-namespace `
  --set grafana.adminPassword="admin123"

# ⏱️ Chờ khoảng 2-3 phút
# Kiểm tra pods đã chạy chưa:
kubectl get pods -n monitoring
# → Tất cả pods STATUS = Running ✅
```

## Mở Grafana

```powershell
# Mở đường hầm đến Grafana
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

# Mở browser: http://localhost:3000
# Đăng nhập:
#   Username: admin
#   Password: admin123
```

### Xem Dashboard có sẵn

```
Trong Grafana:
1. Click ☰ (menu bên trái) → Dashboards
2. Bạn sẽ thấy nhiều dashboard có sẵn:

📊 "Kubernetes / Compute Resources / Cluster"
   → Tổng quan CPU, Memory của TOÀN BỘ cluster

📊 "Kubernetes / Compute Resources / Namespace (Pods)"
   → Xem chi tiết từng Pod trong namespace

📊 "Kubernetes / Networking / Cluster"
   → Xem traffic mạng

📊 "Node Exporter / Nodes"
   → Xem chi tiết phần cứng từng node
```

### Import thêm Dashboard

```
Grafana → Menu → Dashboards → New → Import
→ Nhập ID: 315 → Load → Import
   (Đây là dashboard "Kubernetes cluster monitoring" rất đẹp)

Các ID dashboard hay:
  315   → Kubernetes Cluster Monitoring
  6417  → Kubernetes Cluster (chi tiết)
  1860  → Node Exporter Full (chi tiết hardware)
  13770 → Karpenter Dashboard
```

## Mở Prometheus

```powershell
# Mở đường hầm đến Prometheus
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090

# Mở browser: http://localhost:9090
```

### Thử truy vấn PromQL (ngôn ngữ query của Prometheus)

Trong Prometheus UI → nhập vào ô "Expression":

```promql
# 1. Xem CPU usage của tất cả pods (đơn giản nhất)
rate(container_cpu_usage_seconds_total[5m])
# → Nhấn "Execute" → thấy bảng số liệu
# → Nhấn tab "Graph" → thấy biểu đồ

# 2. Xem Memory đang dùng (MB)
container_memory_working_set_bytes / 1024 / 1024
# → Chia cho 1024 hai lần để đổi bytes → MB

# 3. Đếm số pods đang chạy
count(kube_pod_status_phase{phase="Running"})

# 4. Xem pods bị restart (có vấn đề)
kube_pod_container_status_restarts_total > 0
```

## Tạo Alert (Cảnh báo tự động)

```powershell
kubectl apply -f practices/08-monitoring/alert-rules.yaml
```

Alert rules này sẽ:

```
📢 HighCPUUsage: CPU > 80% trong 5 phút → Cảnh báo WARNING
📢 HighMemoryUsage: Memory > 85% trong 5 phút → Cảnh báo CRITICAL
📢 PodCrashLooping: Pod restart > 5 lần/giờ → Cảnh báo CRITICAL
```

### Xem alerts trong Grafana

```
Grafana → Menu → Alerting → Alert rules
→ Thấy danh sách rules đã tạo
→ Khi có alert fire → hiển thị màu đỏ
```

---

# 🔰 BÀI 7: TỔNG HỢP & TROUBLESHOOTING

## Cheat Sheet - Lệnh hay dùng nhất

```powershell
# ========== XEM ============
kubectl get pods               # Danh sách pods
kubectl get pods -o wide       # Thêm cột IP, Node
kubectl get all                # Xem hết tất cả
kubectl get events             # Xem sự kiện gần đây

# ========== CHI TIẾT ============
kubectl describe pod <tên>     # Chi tiết pod
kubectl logs <tên>             # Xem log
kubectl logs -f <tên>          # Log realtime

# ========== SỬA LỖI ============
kubectl exec -it <tên> -- bash   # Vào bên trong pod
kubectl top pods                  # CPU/Memory mỗi pod
kubectl top nodes                 # CPU/Memory mỗi node

# ========== TRIỂN KHAI ============
kubectl apply -f file.yaml       # Tạo/Cập nhật
kubectl delete -f file.yaml      # Xóa
kubectl rollout restart deploy <tên>  # Restart deployment
```

## Lỗi thường gặp & cách sửa

| Lỗi | Nguyên nhân | Cách sửa |
|---|---|---|
| **ImagePullBackOff** | Sai tên image hoặc tag | Kiểm tra lại `image:` trong YAML |
| **CrashLoopBackOff** | App bên trong crash | `kubectl logs <pod>` để xem lỗi |
| **Pending** | Không đủ CPU/Memory | Giảm `requests` hoặc thêm node |
| **OOMKilled** | Hết RAM | Tăng `limits.memory` |
| **CreateContainerConfigError** | ConfigMap/Secret sai | Kiểm tra tên ConfigMap/Secret |

## Lộ trình học tiếp

```
✅ Tuần 1: Bài 1-3 (K8s cơ bản, thực hành trên Minikube)
✅ Tuần 2: Bài 4 (Terraform)
✅ Tuần 3: Bài 5-6 (Karpenter + Monitoring)
📚 Tuần 4+: Nâng cao:
   → Helm Charts (đóng gói app)
   → ArgoCD (GitOps - deploy tự động từ Git)
   → Istio (Service Mesh - quản lý traffic nâng cao)
   → CI/CD Pipeline (Jenkins/GitHub Actions + K8s)
```

---

## 📎 Link tham khảo

| Tài liệu | Link |
|---|---|
| Kubernetes chính thức | https://kubernetes.io/docs/home/ |
| Terraform | https://developer.hashicorp.com/terraform/docs |
| Karpenter | https://karpenter.sh/docs/ |
| Prometheus | https://prometheus.io/docs/ |
| Grafana | https://grafana.com/docs/ |
| Minikube | https://minikube.sigs.k8s.io/docs/ |
