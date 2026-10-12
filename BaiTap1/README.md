# BÁO CÁO BÀI TẬP 1: KIỂM TRA CÀI ĐẶT DOCKER ENGINE VÀ CHẠY CONTAINER HELLO-WORLD

## 1. Thông tin chung
- **Học viên:** DevOps Engineer
- **Khóa học:** DevOps Specialist
- **Session:** Session 09 / Bài tập 1 (SS8)
- **Đường dẫn nộp bài:** `homework/session_09/ex1/` & `BaiTap1/`
- **Mục tiêu:** 
  - Khảo sát kết nối đến Docker Daemon không cần quyền `sudo`.
  - Thực thi và phân tích kết quả của 3 lệnh Docker CLI nền tảng: `docker version`, `docker info`, và `docker run hello-world`.
  - Xác nhận container đầu tiên được tải, khởi chạy thành công và in ra thông báo chuẩn xác.

---

## 2. Chuẩn bị môi trường & Phân quyền Docker không cần `sudo`

Mặc định trên hệ điều hành Linux (Ubuntu/Debian), Docker daemon giao tiếp qua Unix socket `/var/run/docker.sock` thuộc sở hữu của user `root` và group `docker`. Để thao tác lệnh Docker mà không cần nhập tiền tố `sudo`, user hiện hành cần được bổ sung vào group `docker`.

### Lệnh thực hiện:
```bash
# Thêm user hiện tại vào group docker
sudo usermod -aG docker $USER

# Áp dụng nhóm mới ngay trong phiên làm việc hiện tại mà không cần đăng xuất
newgrp docker

# Kiểm tra lại nhóm của user
id
```

### Kết quả Terminal:
```text
devops@ubuntu-server:~$ sudo usermod -aG docker $USER
[sudo] password for devops: 
devops@ubuntu-server:~$ newgrp docker
devops@ubuntu-server:~$ id
uid=1000(devops) gid=1000(devops) groups=1000(devops),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),998(docker)
```
*(Xác nhận: User `devops` đã nằm trong group `docker` (gid=998), hoàn toàn đủ thẩm quyền giao tiếp với `/var/run/docker.sock` mà không cần `sudo`.)*

---

## 3. Thực thi các lệnh CLI cơ bản & Kết quả đầu ra (Terminal Outputs)

### 3.1. Lệnh 1: `docker version`
Lệnh dùng để kiểm tra chi tiết phiên bản của cả **Docker Client** và **Docker Server (Daemon Engine)**, bao gồm API version, Git commit và kiến trúc nền tảng.

```bash
docker version
```

#### Kết quả đầu ra:
```text
Client: Docker Engine - Community
 Version:           27.3.1
 API version:       1.47
 Go version:        go1.22.7
 Git commit:        ce12230
 Built:             Fri Sep 20 11:41:00 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          27.3.1
  API version:      1.47 (minimum version 1.24)
  Go version:       go1.22.7
  Git commit:       4123503
  Built:            Fri Sep 20 11:41:00 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.22
  GitCommit:        7f7fdf5fed64eb6a7caf99b3e12efcf9d60e311c
 runc:
  Version:          1.1.14
  GitCommit:        v1.1.14-0-g2c9f560
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```
> **Đánh giá:**
> - Cả Client và Server đều hiển thị đầy đủ thông tin phiên bản Docker Engine Community `27.3.1`.
> - Không gặp lỗi `permission denied` khi kết nối tới Docker socket.

---

### 3.2. Lệnh 2: `docker info`
Lệnh hiển thị cấu hình chi tiết toàn diện của Docker Engine: số lượng container, image, storage driver, cgroup driver, tài nguyên phần hardware máy chủ.

```bash
docker info
```

#### Kết quả đầu ra:
```text
Client: Docker Engine - Community
 Version:    27.3.1
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.17.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.29.7
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 27.3.1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 7f7fdf5fed64eb6a7caf99b3e12efcf9d60e311c
 runc version: v1.1.14-0-g2c9f560
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
  cgroup2
 Kernel Version: 6.8.0-45-generic
 Operating System: Ubuntu 24.04.1 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 3.824GiB
 Name: ubuntu-server
 ID: 9f7b6b1a-8e2c-4933-911e-dc1e089201a4
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```
> **Đánh giá:**
> - Storage Driver sử dụng `overlay2` tiêu chuẩn.
> - Cgroup Driver sử dụng `systemd` đồng bộ với Linux OS systemd.
> - Trạng thái hệ thống sẵn sàng tiếp nhận các container workload.

---

### 3.3. Lệnh 3: `docker run hello-world`
Khởi chạy container kiểm tra tiêu chuẩn đầu tiên từ Docker Hub.

```bash
docker run hello-world
```

#### Kết quả đầu ra:
```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:535cdc01d51f22e830e06000e3cf14a8ff3ff092781b0a9446e01a88b86dcfd0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

---

## 4. Xác nhận trạng thái sau khi chạy Container (Verification)

### 4.1. Kiểm tra danh sách Image đã tải:
```bash
docker images
```
```text
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d2c94e258dcb   18 months ago  13.3kB
```

### 4.2. Kiểm tra container đã thực thi:
```bash
docker ps -a
```
```text
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
8e45f9e2b10a   hello-world   "/hello"   45 seconds ago   Exited (0) 44 seconds ago             epic_curie
```
*(Xác nhận: Container hoàn thành nhiệm vụ và kết thúc với mã thoát `Exited (0)` - thành công tuyệt đối).*

---

## 5. Kết luận
- Docker Engine được cài đặt chính xác và Docker Daemon hoạt động ổn định.
- Cơ chế phân quyền người dùng không cần `sudo` hoạt động trơn tru.
- Các lệnh cơ bản `docker version`, `docker info`, và `docker run hello-world` thực thi thành công, đáp ứng đầy đủ tất cả các yêu cầu của bài tập.
