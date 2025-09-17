# 🐳 Cài đặt Docker trên Windows Server (IIS)

## 📑 Mục lục
- [1. Giới thiệu](#1-giới-thiệu)
- [2. Yêu cầu hệ thống](#2-yêu-cầu-hệ-thống)
- [3. Các bước cài đặt & chuyển Docker Root Dir sang ổ D](#3-các-bước-cài-đặt--chuyển-docker-root-dir-sang-ổ-d)
  - [3.1 Tải Docker Binary](#31-tải-docker-binary)
  - [3.2 Giải nén Docker](#32-giải-nén-docker)
  - [3.3 Đăng ký Docker Service](#33-đăng-ký-docker-service)
  - [3.4 Tạo thư mục Docker Data & chuyển Root Dir](#34-tạo-thư-mục-docker-data--chuyển-root-dir)
  - [3.5 Bật Windows Containers Feature](#35-bật-windows-containers-feature)
  - [3.6 Cấu hình PATH cho Docker](#36-cấu-hình-path-cho-docker)
  - [3.7 Khởi động Docker Service](#37-khởi-động-docker-service)
  - [3.8  Đồng bộ dữ liệu Docker cũ (tuỳ chọn)](#37-khởi-động-docker-service)
- [4. Kiểm tra Docker](#4-kiểm-tra-docker)
- [5. Lưu ý quan trọng](#5-lưu-ý-quan-trọng)

---

## 1. Giới thiệu
README này hướng dẫn cài Docker Engine (static binary) trên Windows Server có IIS, đồng thời **di chuyển Docker Root Dir sang ổ D** để tránh đầy ổ C.

---

## 2. Yêu cầu hệ thống
- Windows Server 2019/2022
- IIS đã cài (Docker & IIS hoạt động độc lập)
- Quyền Administrator
- Kết nối Internet để tải Docker

---

## 3. Các bước cài đặt & chuyển Docker Root Dir sang ổ D

### 3.1 Tải Docker Binary
```powershell
Invoke-WebRequest -Uri "https://download.docker.com/win/static/stable/x86_64/docker-24.0.2.zip" -OutFile "D:\Program Files\Docker\docker.zip"
```

### 3.2 Giải nén Docker

```powershell
Expand-Archive -Path "D:\Program Files\Docker\docker.zip" -DestinationPath "D:\Program Files\Docker" -Force
```
### 3.3 Đăng ký Docker Service

```powershell
& "D:\Program Files\Docker\docker\dockerd.exe" --register-service
```
### 3.4 Tạo thư mục Docker Data & chuyển Root Dir
- Tạo thư mục docker-data
```powershell
New-Item -ItemType Directory -Path "D:\Program Files\Docker\docker-data"
```
- Tìm file cấu hình daemon.json
  - Docker Engine trên Windows dùng file : C:\ProgramData\docker\config\daemon.json
  - Nếu chưa có thì mình tự tạo mới
  ```powershell
  New-Item -ItemType Directory -Path "D:\Program Files\Docker\docker-data"
  ```
- Chỉnh sửa daemon.json
  - Lệnh chỉnh sửa
  ```powershell
  notepad "C:\ProgramData\docker\config\daemon.json"
  ```
  - Nội dung
  ```json
  {
    "data-root": "D:\Program Files\Docker\docker-data"
  }

  ```

### 3.5 Bật Windows Containers Feature
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Containers -All
```
- Nếu cần Hyper-V:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```
### 3.6 Cấu hình PATH cho Docker
```powershell
setx /M PATH "$($env:PATH);D:\Program Files\Docker\docker"
```
### 3.7 Khởi động Docker Service
```powershell
Start-Service docker
```

### 3.8 Đồng bộ dữ liệu Docker cũ (tuỳ chọn)
```powershell
New-Item -ItemType Directory -Force -Path "C:\ProgramData\docker\config"
robocopy "C:\ProgramData\docker" "D:\Program Files\Docker\docker-data" /MIR
```
## 4. Kiểm tra Docker
- Kiểm tra version
```powershell
docker version
docker info
```
- Test
```powershell
docker pull hello-world:nanoserver
docker run --rm hello-world:nanoserver
```
## 5. Lưu ý quan trọng
- Backup nếu cần giữ images/containers quan trọng.
- Luôn Stop-Service docker trước khi di chuyển dữ liệu.
- Đường dẫn trong JSON cần escape: D:\\path\\to\\dir.
- Service name mặc định: docker.
- Nếu Docker không khởi động: chạy
- Service name mặc định: docker.
  ```powershell
  & "D:\Program Files\Docker\docker\dockerd.exe" -D
  ```


