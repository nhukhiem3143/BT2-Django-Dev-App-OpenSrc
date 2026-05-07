# 🏪 Hệ Thống Quản Lý Tiệm Cầm Đồ

> **Môn học:** Phát triển ứng dụng với mã nguồn mở-TEE0421  
> **Công nghệ:** Django · MariaDB · Docker · PhpMyAdmin · Cloudflare Tunnel  
> **Deadline:** 23h59 ngày 09/05/2026

---

## 📋 Mục Lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Thiết kế CSDL](#2-thiết-kế-csdl)
3. [Kiến trúc hệ thống](#3-kiến-trúc-hệ-thống)
4. [Cấu trúc thư mục](#4-cấu-trúc-thư-mục)
5. [Hướng dẫn cài đặt](#5-hướng-dẫn-cài-đặt)
6. [Chi tiết các file](#6-chi-tiết-các-file)
7. [Hướng dẫn sử dụng](#7-hướng-dẫn-sử-dụng)
8. [Public với Cloudflare Tunnel](#8-public-với-cloudflare-tunnel)
9. [Kết quả demo](#9-kết-quả-demo)

---

## 1. Giới Thiệu

Hệ thống quản lý tiệm cầm đồ được xây dựng bằng **Django** (Python), chạy hoàn toàn trên **Docker** với 3 service:

| Service | Công dụng | Port |
|---|---|---|
| **MariaDB** | Lưu trữ toàn bộ cơ sở dữ liệu | 3306 |
| **PhpMyAdmin** | Giao diện xem/kiểm tra CSDL | 8088 |
| **Django** | Ứng dụng web chính | 8000 |

### Tính năng chính

- Quản lý khách hàng, nhân viên, hợp đồng cầm đồ, tài sản, lịch sử thanh toán
- Trang **Admin** tự động: thêm / sửa / xóa dữ liệu mọi bảng
- Khóa ngoại hiển thị dạng **dropdown chọn text** (Django tự xử lý)
- Trang chủ liệt kê **con nợ đến hạn chưa trả** (dùng template Jinja2)
- Public ra Internet qua **Cloudflare Tunnel** (không cần mua domain)

---

## 2. Thiết Kế CSDL

### 2.1 Sơ đồ quan hệ

> *(Ảnh thiết kế tay — tự chèn vào đây)*

<!-- 📷 CHÈN ẢNH THIẾT KẾ TAY VÀO ĐÂY -->
<!-- ![Thiết kế CSDL](docs/database_design.jpg) -->

### 2.2 Các bảng và quan hệ

```
KhachHang ──────(1:N)──────► HopDongCamDo
NhanVien  ──────(1:N)──────► HopDongCamDo
HopDongCamDo ──(1:N)──────► TaiSan
HopDongCamDo ──(1:N)──────► LichSuThanhToan
DanhMucTaiSan ─(1:N)──────► TaiSan
```

### 2.3 Mô tả chi tiết từng bảng

#### 📌 Bảng `KhachHang`
| Trường | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `id` | INT (PK, Auto) | Khóa chính |
| `ho_ten` | VARCHAR(100) | Họ và tên khách hàng |
| `so_dien_thoai` | VARCHAR(15) | Số điện thoại liên hệ |
| `so_cmnd` | VARCHAR(20) UNIQUE | Số CMND / CCCD |
| `dia_chi` | TEXT | Địa chỉ thường trú |
| `ngay_tao` | DATE | Ngày tạo hồ sơ |

#### 📌 Bảng `NhanVien`
| Trường | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `id` | INT (PK, Auto) | Khóa chính |
| `ho_ten` | VARCHAR(100) | Họ và tên nhân viên |
| `so_dien_thoai` | VARCHAR(15) | Số điện thoại |
| `chuc_vu` | VARCHAR(50) | Chức vụ trong tiệm |

#### 📌 Bảng `DanhMucTaiSan`
| Trường | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `id` | INT (PK, Auto) | Khóa chính |
| `ten_danh_muc` | VARCHAR(100) | Tên loại (điện thoại, vàng, xe...) |
| `mo_ta` | TEXT | Mô tả thêm |

#### 📌 Bảng `HopDongCamDo` *(nghiệp vụ chính)*
| Trường | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `id` | INT (PK, Auto) | Khóa chính |
| `khach_hang_id` | INT (FK → KhachHang) | Khóa ngoại khách hàng |
| `nhan_vien_id` | INT (FK → NhanVien) | Khóa ngoại nhân viên |
| `so_hop_dong` | VARCHAR(20) UNIQUE | Mã số hợp đồng |
| `ngay_cam` | DATE | Ngày cầm đồ |
| `ngay_dao_han` | DATE | Ngày đến hạn chuộc |
| `so_tien_cho_vay` | DECIMAL(15,0) | Số tiền cho vay (VNĐ) |
| `lai_suat_thang` | DECIMAL(5,2) | Lãi suất mỗi tháng (%) |
| `trang_thai` | VARCHAR(20) | đang_cam / da_chuoc / qua_han |
| `ghi_chu` | TEXT | Ghi chú thêm |

#### 📌 Bảng `TaiSan`
| Trường | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `id` | INT (PK, Auto) | Khóa chính |
| `hop_dong_id` | INT (FK → HopDongCamDo) | Khóa ngoại hợp đồng |
| `danh_muc_id` | INT (FK → DanhMucTaiSan) | Khóa ngoại danh mục |
| `ten_tai_san` | VARCHAR(200) | Tên tài sản cụ thể |
| `mo_ta_tai_san` | TEXT | Mô tả chi tiết |
| `gia_tri_dinh_gia` | DECIMAL(15,0) | Giá trị định giá (VNĐ) |

#### 📌 Bảng `LichSuThanhToan`
| Trường | Kiểu dữ liệu | Mô tả |
|---|---|---|
| `id` | INT (PK, Auto) | Khóa chính |
| `hop_dong_id` | INT (FK → HopDongCamDo) | Khóa ngoại hợp đồng |
| `ngay_thanh_toan` | DATE | Ngày thực hiện thanh toán |
| `so_tien` | DECIMAL(15,0) | Số tiền thanh toán |
| `loai_thanh_toan` | VARCHAR(20) | tra_lai / gia_han / chuoc_hang |
| `ghi_chu` | TEXT | Ghi chú |

---

## 3. Kiến Trúc Hệ Thống

```
┌─────────────────────────────────────────────────────┐
│                   Docker Network                     │
│                                                      │
│  ┌──────────────┐    ┌──────────────┐               │
│  │   Django     │    │ PhpMyAdmin   │               │
│  │  :8000       │    │  :8080       │               │
│  └──────┬───────┘    └──────┬───────┘               │
│         │                   │                        │
│         └─────────┬─────────┘                        │
│                   ▼                                  │
│           ┌───────────────┐                          │
│           │   MariaDB     │                          │
│           │   :3306       │                          │
│           └───────────────┘                          │
└─────────────────────────────────────────────────────┘
         │                          │
    localhost:8000             localhost:8080
    (Trang web Django)         (PhpMyAdmin)
         │
    cloudflared tunnel
         │
    https://xxx.trycloudflare.com  (Public Internet)
```

---

## 4. Cấu Trúc Thư Mục

```
django-project/                  
│
├── docker-compose.yml              ← định nghĩa 3 service
├── .env                            ← chứa password, secret key (KHÔNG push lên git)
├── .gitignore                      ← bỏ qua .env, __pycache__, ...
├── README.md                       ← hướng dẫn (file này)
│
├── django/                         ← thư mục build Docker image cho Django
│   ├── Dockerfile                  ← build image Python + Django
│   ├── requirements.txt            ← danh sách thư viện pip
│   │
│   └── myshop/                   ← Django project (mount vào container)
│       ├── manage.py
│       ├── config/               ← Django config app
│       │   ├── settings.py         ← cấu hình DB, INSTALLED_APPS, ...
│       │   ├── urls.py
│       │   └── wsgi.py
│       │
│       └── core/                   ← app chính chứa nghiệp vụ
│           ├── models.py           ← định nghĩa bảng CSDL
│           ├── admin.py            ← đăng ký bảng vào trang admin
│           ├── views.py            ← view home_page con nợ đến hạn
│           ├── urls.py
│           └── templates/
│               └── core/
│                   └── home.html   ← template Jinja2 liệt kê con nợ
│
└── docs/                           ← ảnh chụp sơ đồ CSDL viết tay
    └── so_do_csdl.jpg              ← upload lên GitHub
```

---

## 5. Hướng Dẫn Cài Đặt

### 5.1 Yêu cầu môi trường

- Ubuntu Server (hoặc Desktop) đã cài Docker & Docker Compose
- Kết nối SSH vào server
- Kết nối Internet

Kiểm tra Docker đã cài chưa:
```bash
docker --version
docker compose version
```

---

### 5.2 Bước 1 — Clone project

```bash
[git clone https://github.com/<ten-ban>/<ten-repo>.git](https://github.com/nhukhiem3143/BT2-Django-Dev-App-OpenSrc.git)
cd django-project
```

---

### 5.3 Bước 2 — Khởi động Docker (lần đầu)

```bash
# Build image Django và khởi động 3 container
docker compose up -d --build
```

Kiểm tra 3 container đang chạy:
```bash
docker compose ps
```
<img width="1589" height="244" alt="image" src="https://github.com/user-attachments/assets/f1a916f3-97fe-4e33-9dda-6221785a3383" />

---

### 5.4 Bước 3 — Khởi tạo Django (chỉ làm 1 lần đầu)

#### Nếu chưa có project Django, tạo mới trong container
```bash
docker compose exec django django-admin startproject config .
```
<img width="1591" height="144" alt="image" src="https://github.com/user-attachments/assets/719e471e-787f-41e2-b6c6-1fe2e7c72f36" />

#### Tạo app core
```bash
docker compose exec django python manage.py startapp core
```
<img width="1578" height="77" alt="image" src="https://github.com/user-attachments/assets/0c8940d3-a801-4355-b3c8-6fd65bdb65af" />

---

### 5.5 Bước 4 — Cấu hình settings.py

Sửa file bằng `nano`:
```bash
sudo nano django/config/settings.py
```

Tìm và sửa / thêm các phần sau:

```python

import os

# ── Bảo mật 
# Đọc SECRET_KEY từ biến môi trường (khai báo trong .env)
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY', 'fallback-secret-key-change-me')

# Debug: True khi dev, False khi production
DEBUG = os.environ.get('DJANGO_DEBUG', 'True') == 'True'

# Cho phép mọi host truy cập (phù hợp khi dùng Cloudflare Tunnel)
ALLOWED_HOSTS = ['*']

# ── Ứng dụng đã cài 
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',                  # app chính chứa models, views
    'widget_tweaks',         # hỗ trợ custom form trong template
]

ROOT_URLCONF = 'myshop.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # Thư mục templates toàn cục (nếu cần)
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,            # Tìm templates trong thư mục mỗi app
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'config.wsgi.application'

# ── Kết nối MariaDB ───────────────────────────────────────────
# Đọc thông tin DB từ biến môi trường (set bởi docker-compose)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',    # dùng MySQL engine (tương thích MariaDB)
        'NAME': os.environ.get('DB_NAME', 'camdo_db'),
        'USER': os.environ.get('DB_USER', 'camdo_user'),
        'PASSWORD': os.environ.get('DB_PASSWORD', ''),
        'HOST': os.environ.get('DB_HOST', 'mariadb'),  # tên service trong docker-compose
        'PORT': os.environ.get('DB_PORT', '3306'),
        'OPTIONS': {
            'charset': 'utf8mb4',                # hỗ trợ tiếng Việt đầy đủ
        },
    }
}

# ── Xác thực mật khẩu 
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator'},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

# ── Quốc tế hóa 
LANGUAGE_CODE = 'vi'           # giao diện admin tiếng Việt
TIME_ZONE = 'Asia/Ho_Chi_Minh'
USE_I18N = True
USE_TZ = True

# ── Static files 
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'   # nơi collectstatic gom file

```

Lưu file: `Ctrl+O` → `Enter` → `Ctrl+X`

---

### 5.6 Bước 5 — Tạo bảng CSDL (Migration)


#### Tạo file migration từ models.py
```bash
docker compose exec django python manage.py makemigrations core
```
<img width="1238" height="345" alt="image" src="https://github.com/user-attachments/assets/c0bac95b-1243-4a78-ab17-3d0ebd93e366" />

#### Apply migration vào MariaDB (Django tự tạo bảng)
```bash
docker compose exec django python manage.py migrate
```
<img width="1250" height="627" alt="image" src="https://github.com/user-attachments/assets/b9312be0-467e-4936-8417-8173fbfcee20" />  

**Kiểm chứng:** Mở PhpMyAdmin tại `http://192.168.100.2:8088`  
→ Đăng nhập (user, pass đã tạo)  
→ Vào database `quanlytiemcamdo` → Thấy các bảng đã được tạo ✅  

<img width="1850" height="944" alt="image" src="https://github.com/user-attachments/assets/0eb9ddfe-98bb-41e5-83a8-3e111d2b2942" />  

---

### 5.7 Bước 6 — Tạo tài khoản admin

```bash
docker compose exec django python manage.py createsuperuser
```

Nhập theo (đã tạo ở .env) thứ tự:
```
Username: admin
Email: nhukhiem24@gmail.com
Password: ••••••••
Password (again): ••••••••
```
<img width="1586" height="273" alt="image" src="https://github.com/user-attachments/assets/036b161a-d92e-48f9-bd8f-3411cb130714" />

---

### 5.8 Bước 7 — Truy cập hệ thống

| Địa chỉ | Chức năng |
|---|---|
| `http://192.168.100.2:8000/admin` | Trang quản trị Django |
| `http://192.168.100.2:8000/` | Trang con nợ đến hạn |
| `http://192.168.100.2:8088/` | PhpMyAdmin xem CSDL |

<img width="1436" height="780" alt="image" src="https://github.com/user-attachments/assets/1ae78e10-0506-4895-96a0-80b32c23d7fb" />  
<img width="1426" height="773" alt="image" src="https://github.com/user-attachments/assets/326fe75c-ce26-4f82-a4af-225b1a0d3d79" />

---

## 6. Chi Tiết Các File

### 6.1 `django_app/Dockerfile`

```dockerfile
# ============================================================
#  Dockerfile — Build image Python + Django cho service django
#  Nằm trong thư mục: django/
# ============================================================

# Dùng Python 3.11 bản slim (nhẹ, không có các package thừa)
FROM python:3.11-slim

# Đặt thư mục làm việc bên trong container
WORKDIR /app

# Cài các gói hệ thống cần thiết:
#   - gcc, pkg-config: để biên dịch thư viện C
#   - default-libmysqlclient-dev: header để cài mysqlclient (kết nối MariaDB)
#   - curl: tiện ích mạng (debug)
RUN apt-get update && apt-get install -y \
    gcc \
    pkg-config \
    default-libmysqlclient-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*    # xóa cache apt để giảm kích thước image

# Copy file requirements.txt vào container trước
# (tách riêng để Docker cache layer này, không rebuild khi chỉ sửa code)
COPY requirements.txt .

# Cài toàn bộ thư viện Python từ requirements.txt
# --no-cache-dir: không lưu cache pip → giảm kích thước image
RUN pip install --no-cache-dir -r requirements.txt

# Copy toàn bộ source code vào container
# (thực tế bị ghi đè bởi volume mount, nhưng cần cho bước build)
COPY myshop/ .

# Mở port 8000 để bên ngoài container có thể kết nối
EXPOSE 8000

# Lệnh mặc định khi container khởi động
# (bị ghi đè bởi `command:` trong docker-compose.yml)
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

---

### 6.2 `django_app/requirements.txt`

```txt
# ============================================================
#  requirements.txt — Danh sách thư viện Python cần cài
#  Cài bằng lệnh: pip install -r requirements.txt
# ============================================================

# Django: web framework chính — tạo project, app, ORM, admin
Django==4.2.16

# mysqlclient: driver kết nối Python ↔ MariaDB/MySQL
# (cần gcc + libmysqlclient-dev đã cài trong Dockerfile)
mysqlclient==2.2.4

# python-dotenv: đọc biến môi trường từ file .env
python-dotenv==1.0.1

# django-widget-tweaks: giúp custom form widget trong template HTML
django-widget-tweaks==1.5.0
```

---

### 6.3 `docker-compose.yml`

```yaml
version: '3.8'

services:

  # 1. MariaDB: lưu toàn bộ CSDL 
  mariadb:
    image: mariadb:10.11       
    container_name: camdo_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mariadb_data:/var/lib/mysql   # lưu data ra ngoài, không mất khi restart
    ports:
      - "3307:3306"
    networks:
      - camdo_net
    healthcheck:                  
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5

  # 2. phpMyAdmin: giao diện web xem CSDL 
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: camdo_phpmyadmin
    restart: always
    environment:
      PMA_HOST: mariadb        
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    ports:
      - "8088:80"             
    depends_on:
      mariadb:
        condition: service_healthy  # chỉ khởi động sau khi mariadb healthy
    networks:
      - camdo_net

  # 3. Django: build từ Dockerfile
  django:
    build:
      context: ./django             # thư mục chứa Dockerfile
      dockerfile: Dockerfile
    container_name: camdo_django
    restart: always
    env_file:
      - .env                        # nạp toàn bộ biến từ file .env
    environment:
      DB_HOST: mariadb
      DB_PORT: 3306
      DB_NAME: ${MYSQL_DATABASE}
      DB_USER: ${MYSQL_USER}
      DB_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ./django/myshop:/app        # mount thư mục → edit file bằng sudo nano, thấy ngay
    ports:
      - "8000:8000"             
    depends_on:
      mariadb:
        condition: service_healthy
    networks:
      - camdo_net
    command: >
      sh -c "python manage.py migrate &&
             python manage.py collectstatic --noinput &&
             python manage.py runserver 0.0.0.0:8000"

# ── Volumes dùng chung 
volumes:
  mariadb_data:

# ── Network nội bộ giữa các container 
networks:
  camdo_net:
    driver: bridge
```

---

## 7. Hướng Dẫn Sử Dụng

### 7.1 Các lệnh Docker thường dùng

```bash
# Khởi động toàn bộ hệ thống
docker compose up -d

# Dừng toàn bộ hệ thống
docker compose down

# Xem log của Django (debug lỗi)
docker compose logs web

# Xem log của MariaDB
docker compose logs db

# Rebuild image Django sau khi sửa Dockerfile / requirements.txt
docker compose up -d --build

# Vào trong container Django để chạy lệnh
docker compose exec web bash
```

---

### 7.2 Các lệnh Django thường dùng

```bash
# Đọc models.py và tạo file migration
docker compose exec web python manage.py makemigrations camdo

# Apply migration → Django tự tạo/sửa bảng trong MariaDB
docker compose exec web python manage.py migrate

# Tạo tài khoản superuser (đăng nhập trang admin)
docker compose exec web python manage.py createsuperuser

# Thu thập static files (nếu cần)
docker compose exec web python manage.py collectstatic

# Mở Django shell (kiểm tra query)
docker compose exec web python manage.py shell
```

---

### 7.3 Quy trình khi sửa code

```
Sửa file bằng nano
       ↓
sudo nano django_app/camdo/models.py
       ↓
Nếu sửa models.py → chạy makemigrations + migrate
Nếu sửa views.py / html → Django tự reload (không cần restart)
Nếu sửa settings.py → restart container:
       ↓
docker compose restart web
```

---

### 7.4 Kiểm tra khóa ngoại bằng PhpMyAdmin

1. Truy cập `http://192.168.100.2:8088/`
2. Đăng nhập: user / pass 
3. Chọn database `quanlytiemcamdo`
4. Mở bảng `core_hopdong`
5. Xem cột `khach_hang_id` → lưu **ID số** (khóa ngoại)
6. Mở bảng `core_khachhang` → tìm **ID tương ứng** → đó chính là khách hàng

> Trong Django Admin: trường khóa ngoại hiển thị **tên text** (ví dụ: "Nguyễn Văn A - 012345678")
> nhưng MariaDB thực tế lưu **số ID** → PhpMyAdmin giúp kiểm chứng điều này.

---

## 8. Public Với Cloudflare Tunnel

### 8.1 Cài đặt cloudflared

```bash
# Tải cloudflared cho Ubuntu x64
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb \
     -o cloudflared.deb

# Cài đặt
sudo dpkg -i cloudflared.deb

# Kiểm tra cài thành công
cloudflared --version
```

### 8.2 Tạo tunnel tạm thời (không cần tài khoản)

```bash
# Tunnel Django port 8000 ra Internet
cloudflared tunnel --url http://localhost:8000
```

Kết quả sẽ hiển thị:
```
+-------------------------------------+
| Your quick Tunnel has been created! |
+-------------------------------------+
| https://abc-xyz-123.trycloudflare.com  |
+-------------------------------------+
```

Truy cập URL đó từ bất kỳ thiết bị nào để xem kết quả.

> **Lưu ý:** Tunnel tạm thời sẽ mất khi tắt terminal. Để giữ lâu dài, chạy trong `screen` hoặc `tmux`:
> ```bash
> screen -S tunnel
> cloudflared tunnel --url http://localhost:8000
> # Ctrl+A rồi D để detach
> ```

---

## 9. Kết Quả Demo

### 9.1 Trang Admin Django

> *(Chèn ảnh chụp màn hình trang admin vào đây)*

<!-- ![Trang Admin](docs/screenshot_admin.png) -->

### 9.2 Trang Con Nợ Đến Hạn

> *(Chèn ảnh chụp màn hình trang home.html vào đây)*

<!-- ![Trang Home](docs/screenshot_home.png) -->

### 9.3 PhpMyAdmin — Kiểm Chứng Khóa Ngoại

> *(Chèn ảnh PhpMyAdmin hiển thị ID khóa ngoại vào đây)*

<!-- ![PhpMyAdmin](docs/screenshot_phpmyadmin.png) -->

### 9.4 Cloudflare Tunnel Public URL

> *(Chèn ảnh URL public từ Cloudflare vào đây)*

<!-- ![Cloudflare](docs/screenshot_cloudflare.png) -->


---

*Bài tập môn Lập trình Web — Sử dụng Django quản lý tiệm cầm đồ*
