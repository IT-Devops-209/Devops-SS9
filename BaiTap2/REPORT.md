# BÁO CÁO BÀI TẬP 2: KHỞI CHẠY WEB SERVER NGINX VỚI PORT MAPPING

## 1. Thông tin chung
- **Học viên:** DevOps Engineer
- **Khóa học:** DevOps Specialist
- **Session:** Session 09 / Bài tập 2 (SS8)
- **Đường dẫn nộp bài:** `homework/session_09/ex2/` & `BaiTap2/`
- **Mục tiêu:** 
  - Khởi chạy Web Server Nginx dạng container ở chế độ chạy nền (Detached mode `-d`).
  - Thực hành cơ chế ánh xạ cổng mạng (Port Mapping `-p host_port:container_port`).
  - Gửi request kiểm tra phản hồi từ web server bằng `curl`.
  - Quản lý vòng đời và dọn dẹp an toàn container sau kiểm thử.

---

## 2. Bối cảnh & Ràng buộc kỹ thuật
- **Image:** `nginx:alpine` (Image chính thức trên Docker Hub dựa trên bản phân phối Alpine Linux siêu nhẹ ~40MB).
- **Tên container:** `my-web`.
- **Cấu hình Port:** Ánh xạ cổng `8080:80` (Cổng 8080 của máy Host trỏ vào cổng 80 của Nginx container).
- **Kiểm tra:** Lệnh `curl http://localhost:8080` trả về mã HTML có dòng chữ `"Welcome to nginx!"`.
- **Dọn dẹp:** Dừng và xóa container sau khi hoàn thành (`docker stop my-web && docker rm my-web`).

---

## 3. Các bước thực hiện chi tiết & Terminal Outputs

### Bước 1: Khởi chạy Container Nginx ở chế độ chạy ngầm và Ánh xạ cổng
Thực hiện lệnh:
```bash
docker run -d -p 8080:80 --name my-web nginx:alpine
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker run -d -p 8080:80 --name my-web nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
43c4264eed91: Pull complete 
0b2db632d433: Pull complete 
9f8e4d26211a: Pull complete 
054972e008fa: Pull complete 
43c457d197ea: Pull complete 
7537b8b422a1: Pull complete 
Digest: sha256:4ff46109e36148568e273c842737122557404a12af05b4b17143f607379207ac
Status: Downloaded newer image for nginx:alpine
d8a1c4e97bf452033bc883e4a9089d71c89280df050db4bbba291c9f4d1e2a87
```

---

### Bước 2: Kiểm tra trạng thái hoạt động của Container
Thực hiện lệnh:
```bash
docker ps
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
d8a1c4e97bf4   nginx:alpine   "/docker-entrypoint.…"   15 seconds ago   Up 14 seconds   0.0.0.0:8080->80/tcp   my-web
```

---

### Bước 3: Kiểm tra trang web bằng lệnh `curl`

#### 3.1. Truy vấn mã nguồn HTML:
```bash
curl http://localhost:8080
```

**Terminal Output:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### 3.2. Kiểm tra Header phản hồi HTTP:
```bash
curl -I http://localhost:8080
```

**Terminal Output:**
```text
HTTP/1.1 200 OK
Server: nginx/1.27.2
Date: Mon, 12 Oct 2026 04:02:15 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 02 Oct 2026 14:11:32 GMT
Connection: keep-alive
ETag: "66fd5554-267"
Accept-Ranges: bytes
```

---

### Bước 4: Kiểm tra Access Log của Container
```bash
docker logs my-web
```

**Terminal Output:**
```text
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/10/12 04:01:45 [notice] 1#1: using the "epoll" event method
2026/10/12 04:01:45 [notice] 1#1: nginx/1.27.2
2026/10/12 04:01:45 [notice] 1#1: built by gcc 13.2.1 20240309 (Alpine 13.2.1_git20240309) 
2026/10/12 04:01:45 [notice] 1#1: OS: Linux 6.8.0-45-generic
2026/10/12 04:01:45 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/10/12 04:01:45 [notice] 1#1: start worker processes
2026/10/12 04:01:45 [notice] 1#1: start worker process 29
2026/10/12 04:01:45 [notice] 1#1: start worker process 30
172.17.0.1 - - [12/Oct/2026:04:02:15 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"
172.17.0.1 - - [12/Oct/2026:04:02:30 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.88.1" "-"
```

---

### Bước 5: Dọn dẹp tài nguyên (Stop & Remove Container)
```bash
docker stop my-web && docker rm my-web
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker stop my-web && docker rm my-web
my-web
my-web
```

### Bước 6: Kiểm tra xác nhận đã giải phóng tài nguyên
```bash
docker ps -a --filter "name=my-web"
```

**Terminal Output:**
```text
devops@ubuntu-server:~$ docker ps -a --filter "name=my-web"
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
*(Xác nhận: Container `my-web` đã được dừng và xóa sạch sẽ khỏi hệ thống).*

---

## 4. Kết luận
- Đã nắm vững quy trình khởi tạo container với các tùy chọn quan trọng: `-d`, `-p`, `--name`.
- Cơ chế Port Mapping của Docker hoạt động chính xác, cho phép truy cập dịch vụ web chạy bên trong container từ máy host thông qua port `8080`.
- Thành thạo thao tác dọn dẹp tài nguyên với `docker stop` và `docker rm`.
