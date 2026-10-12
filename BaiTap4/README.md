# Bài 4: Lưu trữ Dữ liệu Đơn giản với Docker Volume

## 1. Thông tin chung
- **Học viên:** DevOps Engineer
- **Khóa học:** DevOps Specialist
- **Session:** Session 09 / Bài tập 4 (SS8)
- **Thư mục bài tập:** `BaiTap4/`
- **Mục tiêu:**
  - Hiểu rõ cơ chế lưu trữ dữ liệu bền vững (Data Persistence) trong Docker thông qua Named Volume.
  - Phân biệt giữa dữ liệu tạm thời trong Container Layer (sẽ mất khi container bị xóa) và dữ liệu được quản lý bởi Volume (tồn tại độc lập với vòng đời container).
  - Thực hành tạo Named Volume, mount volume vào container và kiểm chứng việc chia sẻ/lưu trữ dữ liệu qua nhiều container khác nhau.

---

## 2. Bối cảnh & Ràng buộc kỹ thuật
- **Tên Volume:** `easy-volume`.
- **Ghi dữ liệu:** Sử dụng một container Alpine tạm thời ghi chuỗi `"Luu tru du lieu Docker"` vào tệp `/data/test.txt`:
  ```bash
  docker run --rm -v easy-volume:/data alpine sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"
  ```
- **Đọc dữ liệu & Kiểm tra:** Sử dụng một container Alpine thứ hai mount cùng volume và đọc tệp `/data/test.txt`:
  ```bash
  docker run --rm -v easy-volume:/data alpine cat /data/test.txt
  ```
- **Kết quả mong đợi:** Terminal in ra đúng dòng chữ: `Luu tru du lieu Docker`.

---

## 3. Các bước thực hiện chi tiết & Terminal Outputs

### Bước 1: Khởi tạo Named Volume `easy-volume`
Thực hiện lệnh:
```bash
docker volume create easy-volume
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker volume create easy-volume
easy-volume
```

---

### Bước 2: Kiểm tra danh sách và thông tin chi tiết của Volume

#### 2.1. Liệt kê danh sách volume:
```bash
docker volume ls
```
**Terminal Output:**
```text
devops@ubuntu-server:~$ docker volume ls
DRIVER    VOLUME NAME
local     easy-volume
```

#### 2.2. Kiểm tra thông tin cấu hình chi tiết của volume:
```bash
docker volume inspect easy-volume
```
**Terminal Output:**
```json
[
    {
        "CreatedAt": "2026-10-12T04:08:10Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/easy-volume/_data",
        "Name": "easy-volume",
        "Options": null,
        "Scope": "local"
    }
]
```
> **Giải thích:**
> - `Driver: local`: Volume được lưu trữ cục bộ trên máy host.
> - `Mountpoint: /var/lib/docker/volumes/easy-volume/_data`: Đường dẫn vật lý trên hệ điều hành host nơi Docker lưu trữ toàn bộ dữ liệu của volume này.

---

### Bước 3: Dùng Container Alpine thứ nhất ghi nội dung vào Volume
Thực hiện lệnh:
```bash
docker run --rm -v easy-volume:/data alpine sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker run --rm -v easy-volume:/data alpine sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
43c4264eed91: Pull complete 
Digest: sha256:beefdbd8a1da20b63d16d604f059abb258463371f9fc707580d4614478834476
Status: Downloaded newer image for alpine:latest
```
> **Phân tích kỹ thuật:**
> - `-v easy-volume:/data`: Gắn Named Volume `easy-volume` vào thư mục `/data` bên trong container.
> - `sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"`: Ghi dữ liệu vào file `test.txt` nằm trong thư mục `/data` (tương ứng với volume).
> - `--rm`: Ngay sau khi lệnh ghi hoàn tất, container tự động dừng và bị hủy hoàn toàn khỏi hệ điều hành host (`docker ps -a` sẽ không còn container này).

---

### Bước 4: Dùng Container Alpine thứ hai đọc lại tệp tin từ Volume (Kiểm tra)
Thực hiện lệnh kiểm tra:
```bash
docker run --rm -v easy-volume:/data alpine cat /data/test.txt
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker run --rm -v easy-volume:/data alpine cat /data/test.txt
Luu tru du lieu Docker
```
*(Kết quả khớp chính xác 100% yêu cầu đề bài: Dù container thứ nhất đã bị hủy bỏ hoàn toàn, container thứ hai khi khởi chạy và mount vào `easy-volume` vẫn đọc được nguyên vẹn nội dung: `Luu tru du lieu Docker`)*.

---

### Bước 5: Kiểm tra trực tiếp trên đường dẫn Mountpoint của Host (Mở rộng)
Xác thực sự tồn tại của file trực tiếp trên filesystem của máy chủ:
```bash
sudo ls -l /var/lib/docker/volumes/easy-volume/_data/
sudo cat /var/lib/docker/volumes/easy-volume/_data/test.txt
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ sudo ls -l /var/lib/docker/volumes/easy-volume/_data/
total 4
-rw-r--r-- 1 root root 23 Oct 12 04:08 test.txt

devops@ubuntu-server:~$ sudo cat /var/lib/docker/volumes/easy-volume/_data/test.txt
Luu tru du lieu Docker
```

---

### Bước 6: Dọn dẹp tài nguyên (Xóa Volume sau khi hoàn thành)
```bash
docker volume rm easy-volume
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker volume rm easy-volume
easy-volume
```

Kiểm tra lại danh sách volume:
```bash
docker volume ls
```
```text
devops@ubuntu-server:~$ docker volume ls
DRIVER    VOLUME NAME
```
*(Xác nhận: Volume `easy-volume` đã được giải phóng an toàn).*

---

## 4. Kết luận
- **Data Persistence:** Named Volume giải quyết triệt để vấn đề mất dữ liệu khi container bị dừng hoặc xóa (ephemeral state).
- **Decoupled Architecture:** Tách biệt hoàn toàn giữa tầng xử lý (compute / container) và tầng lưu trữ (storage / volume), là nền tảng quan trọng khi triển khai Database (MySQL, PostgreSQL, MongoDB, Redis) trong môi trường Docker / Kubernetes.
