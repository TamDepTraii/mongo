# 🚀 Báo cáo Thực hành: MongoDB Replication & Sharding

Dự án này là kết quả của quá trình tìm hiểu và triển khai mô phỏng hai cơ chế cốt lõi của MongoDB: **Replication** (để đảm bảo tính sẵn sàng cao) và **Sharding** (để đảm bảo khả năng mở rộng).

Hệ thống được xây dựng hoàn toàn trên môi trường Docker & Docker Compose, giúp dễ dàng tái lập và kiểm thử.

---

## 📚 Mục lục
1. [Tổng quan dự án](#-tổng-quan-dự-án)
2. [Phần 1: Triển khai Replication](#-phần-1-triển-khai-replication-replica-set)
3. [Phần 2: Triển khai Sharding](#-phần-2-triển-khai-sharding-sharded-cluster)
4. [Kết quả & Minh họa](#-kết-quả--minh-họa)

---

## 📝 Tổng quan dự án

| Thành phần | Công nghệ / Công cụ | Mô tả |
| :--- | :--- | :--- |
| **Database** | MongoDB v6.0+ | Cơ sở dữ liệu NoSQL |
| **Containerization** | Docker | Đóng gói môi trường |
| **Orchestration** | Docker Compose | Quản lý đa container |
| **Management** | MongoDB Compass | Giao diện quản lý trực quan |

---

## 🛡 Phần 1: Triển khai Replication (Replica Set)

Mục tiêu: Xây dựng một cụm Replica Set gồm 3 node để đảm bảo dữ liệu luôn an toàn ngay cả khi một node bị sập.

### 1. Kiến trúc hệ thống
*   **Replica Set Name**: `rs0`
*   **Node 1 (Primary)**: `mongo1` (Port Host: 27017)
*   **Node 2 (Secondary)**: `mongo2` (Port Host: 27018)
*   **Node 3 (Secondary)**: `mongo3` (Port Host: 27019)

### 2. Các bước thực hiện

**Bước 1: Khởi động cụm container**
```bash
cd replication
docker-compose up -d
```

**Bước 2: Khởi tạo Replica Set**
Truy cập vào node `mongo1` và thực thi lệnh khởi tạo:
```bash
docker exec -it mongo1 mongosh
```
```javascript
// Trong mongosh
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
})
```

**Bước 3: Kiểm tra trạng thái**
```javascript
rs.status()
```
*Kết quả mong đợi: Một node là PRIMARY, hai node còn lại là SECONDARY.*

---

## 🧩 Phần 2: Triển khai Sharding (Sharded Cluster)

Mục tiêu: Xây dựng hệ thống phân tán dữ liệu (Sharding) để xử lý lượng dữ liệu lớn, bao gồm Router, Config Server và các Shard.

### 1. Kiến trúc hệ thống
*   **Router**: `mongos-router` (Port Host: 27016) - Cổng giao tiếp với ứng dụng.
*   **Config Server**: `config-server` (Port Host: 27019) - Lưu metadata của cluster.
*   **Shard 1**: `shard1-server` (Port Host: 27020) - Lưu trữ một phần dữ liệu.
*   **Shard 2**: `shard2-server` (Port Host: 27021) - Lưu trữ phần dữ liệu còn lại.

### 2. Các bước thực hiện

**Bước 1: Khởi động hạ tầng**
```bash
cd sharding
docker-compose up -d
```

**Bước 2: Khởi tạo Replica Set cho từng thành phần**

*   **Config Server:**
    ```bash
    docker exec -it config-server mongosh --eval "rs.initiate({_id: 'csReplSet', configsvr: true, members: [{_id: 0, host: 'config-server:27017'}]})"
    ```

*   **Shard 1:**
    ```bash
    docker exec -it shard1-server mongosh --eval "rs.initiate({_id: 'shard1ReplSet', members: [{_id: 0, host: 'shard1-server:27017'}]})"
    ```

*   **Shard 2:**
    ```bash
    docker exec -it shard2-server mongosh --eval "rs.initiate({_id: 'shard2ReplSet', members: [{_id: 0, host: 'shard2-server:27017'}]})"
    ```

**Bước 3: Kết nối Shards vào Router**
Truy cập vào Router và thêm các Shard vào Cluster:
```bash
docker exec -it mongos-router mongosh
```
```javascript
// Trong mongosh của Router
sh.addShard("shard1ReplSet/shard1-server:27017")
sh.addShard("shard2ReplSet/shard2-server:27017")
```

**Bước 4: Bật Sharding cho Database (Ví dụ)**
```javascript
sh.enableSharding("myDatabase")
sh.shardCollection("myDatabase.myCollection", { "userId": "hashed" })
```

---



---
*Thực hiện bởi: [Tâm le]*
