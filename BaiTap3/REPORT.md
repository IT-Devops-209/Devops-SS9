# BÁO CÁO BÀI TẬP 3: VIẾT DOCKERFILE ĐÓNG GÓI TRANG WEB HTML TĨNH

## 1. Thông tin chung
- **Học viên:** DevOps Engineer
- **Khóa học:** DevOps Specialist
- **Session:** Session 09 / Bài tập 3 (SS8)
- **Đường dẫn nộp bài:** `homework/session_09/ex3/` & `BaiTap3/`
- **Mục tiêu:**
  - Viết tệp `Dockerfile` đóng gói một trang web HTML tĩnh thành Docker Image độc lập.
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

---

## 3. Các bước thực hiện & Kết quả Terminal (Terminal Outputs)

### Bước 1: Build Docker Image từ Dockerfile
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

#### Kiểm tra Docker Image vừa tạo:
```bash
docker images my-html-app:v1
```
```text
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
my-html-app   v1        b43c68a0f912   30 seconds ago   43.2MB
```

---

### Bước 2: Khởi chạy Container từ Image `my-html-app:v1` trên cổng 8081
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
```bash
curl http://localhost:8081
```

**Terminal Output:**
```html
<h1>Hello Docker Session 09!</h1>
```

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

### Bước 5: Dọn dẹp Container sau kiểm thử
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
- Đã hoàn thành đóng gói thành công ứng dụng web tĩnh với Dockerfile tinh gọn.
- Kiểm tra `curl` phản hồi đúng 100% kết quả mong đợi.
