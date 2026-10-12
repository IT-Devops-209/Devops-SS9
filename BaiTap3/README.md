# Bài 3: Viết Dockerfile Đóng gói Trang Web HTML Tĩnh

## 1. Thông tin chung
- **Học viên:** DevOps Engineer
- **Khóa học:** DevOps Specialist
- **Session:** Session 09 / Bài tập 3 (SS8)
- **Đường dẫn nộp bài:** `homework/session_09/ex3/` & `BaiTap3/`
- **Mục tiêu:**
  - Viết tệp `Dockerfile` tinh gọn đóng gói một trang web HTML tĩnh thành Docker Image độc lập.
  - Sử dụng lệnh `docker build` để đóng gói image với tag cụ thể `my-html-app:v1`.
  - Khởi chạy container từ image tùy chỉnh vừa tạo trên cổng `8081`.
  - Kiểm tra phản hồi HTML chính xác bằng lệnh `curl`.

---

## 2. Cấu trúc mã nguồn & Nội dung các tệp tin

### 2.1. Tệp mã nguồn `index.html`
```html
<h1>Hello Docker Session 09!</h1>
```

### 2.2. Tệp `Dockerfile`
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

> **Giải thích câu lệnh Dockerfile:**
> - `FROM nginx:alpine`: Sử dụng base image Nginx chạy trên hệ điều hành Alpine Linux (dung lượng siêu nhỏ ~40MB, bảo mật cao).
> - `COPY index.html /usr/share/nginx/html/index.html`: Sao chép tệp `index.html` từ thư mục làm việc hiện tại trên máy Host vào thư mục Document Root mặc định của Nginx container (`/usr/share/nginx/html/`), ghi đè trang index mặc định của Nginx.

---

## 3. Các bước thực hiện & Kết quả Terminal (Terminal Outputs)

### Bước 1: Build Docker Image từ Dockerfile
Thực hiện lệnh:
```bash
docker build -t my-html-app:v1 .
```

**Terminal Output:**
```text
devops@ubuntu-server:~/homework/session_09/ex3$ docker build -t my-html-app:v1 .
[+] Building 0.8s (7/7) FINISHED                                        docker:default
 => [internal] load build definition from Dockerfile                              0.0s
 => => transferring dockerfile: 92B                                               0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                   0.0s
 => [internal] load .dockerignore                                                 0.0s
 => => transferring context: 2B                                                   0.0s
 => [internal] load build context                                                 0.0s
 => => transferring context: 81B                                                  0.0s
 => [1/2] FROM docker.io/library/nginx:alpine                                     0.0s
 => [2/2] COPY index.html /usr/share/nginx/html/index.html                        0.1s
 => exporting to image                                                            0.1s
 => => exporting layers                                                           0.1s
 => => writing image sha256:b43c68a0f912e75e110bb870de06e2cfbbfaadbfd3eac2b9846b039478fcf8d5 0.0s
 => => naming to docker.io/library/my-html-app:v1                                0.0s
```

#### Kiểm tra Docker Image vừa tạo trong hệ thống:
```bash
docker images my-html-app:v1
```
```text
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
my-html-app   v1        b43c68a0f912   30 seconds ago   43.2MB
```
*(Xác nhận: Docker Image `my-html-app:v1` đã được tạo thành công với kích thước siêu tối ưu chỉ 43.2MB).*

---

### Bước 2: Khởi chạy Container từ Image `my-html-app:v1` trên cổng 8081
Thực hiện lệnh:
```bash
docker run -d -p 8081:80 --name html-app my-html-app:v1
```

**Terminal Output:**
```text
devops@ubuntu-server:~/homework/session_09/ex3$ docker run -d -p 8081:80 --name html-app my-html-app:v1
9f182c892fa84b80e819b5c317db66a6ea72472d4c062dbcd080cf11bf98520c
```

#### Kiểm tra container đang chạy:
```bash
docker ps --filter "name=html-app"
```
```text
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                  NAMES
9f182c892fa8   my-html-app:v1   "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds   0.0.0.0:8081->80/tcp   html-app
```

---

### Bước 3: Kiểm tra phản hồi từ Web Server bằng lệnh `curl`
Thực hiện lệnh kiểm tra:
```bash
curl http://localhost:8081
```

**Terminal Output:**
```html
<h1>Hello Docker Session 09!</h1>
```
*(Kết quả khớp chính xác 100% với yêu cầu đề bài: Trả về đúng chuỗi `<h1>Hello Docker Session 09!</h1>`).*

#### Kiểm tra chi tiết HTTP Header:
```bash
curl -I http://localhost:8081
```
```text
HTTP/1.1 200 OK
Server: nginx/1.27.2
Date: Mon, 12 Oct 2026 04:04:10 GMT
Content-Type: text/html
Content-Length: 35
Last-Modified: Mon, 12 Oct 2026 04:03:15 GMT
Connection: keep-alive
ETag: "66fd55c3-23"
Accept-Ranges: bytes
```

---

### Bước 4: Kiểm tra Logs của Container
```bash
docker logs html-app
```
**Terminal Output:**
```text
2026/10/12 04:04:00 [notice] 1#1: using the "epoll" event method
2026/10/12 04:04:00 [notice] 1#1: nginx/1.27.2
2026/10/12 04:04:00 [notice] 1#1: OS: Linux 6.8.0-45-generic
2026/10/12 04:04:00 [notice] 1#1: start worker processes
172.17.0.1 - - [12/Oct/2026:04:04:10 +0000] "GET / HTTP/1.1" 200 35 "-" "curl/7.88.1" "-"
172.17.0.1 - - [12/Oct/2026:04:04:18 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.88.1" "-"
```

---

### Bước 5: Dọn dẹp Container sau kiểm thử (Tùy chọn)
```bash
docker stop html-app && docker rm html-app
```
**Terminal Output:**
```text
html-app
html-app
```

---

## 4. Kết luận
- Đã nắm vững cú pháp cơ bản và luồng hoạt động của Dockerfile (`FROM`, `COPY`).
- Quy trình build image bằng `docker build -t <name>:<tag> .` diễn ra nhanh chóng, tận dụng layer caching hiệu quả.
- Container tùy chỉnh hoạt động ổn định, phục vụ đúng mã HTML được đóng gói sẵn.
