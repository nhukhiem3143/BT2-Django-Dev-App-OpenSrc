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
| **PhpMyAdmin** | Giao diện xem/kiểm tra CSDL | 8080 |
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
camdo_project/
│
├── docker-compose.yml              # Cấu hình 3 service Docker
│
├── django_app/                     # Thư mục build image Django
│   ├── Dockerfile                  # Hướng dẫn build container Python/Django
│   ├── requirements.txt            # Danh sách thư viện Python cần cài
│   │
│   ├── config/                     # Cấu hình project Django
│   │   ├── __init__.py
│   │   ├── settings.py             # Cài đặt chính: DB, APPS, TIMEZONE...
│   │   ├── urls.py                 # URL gốc của project
│   │   └── wsgi.py
│   │
│   ├── camdo/                      # App nghiệp vụ tiệm cầm đồ
│   │   ├── __init__.py
│   │   ├── models.py               # Định nghĩa các bảng CSDL
│   │   ├── admin.py                # Cấu hình trang quản trị
│   │   ├── views.py                # Xử lý logic, trả dữ liệu về template
│   │   ├── urls.py                 # URL của app
│   │   └── templates/
│   │       └── camdo/
│   │           └── home.html       # Template Jinja2: trang con nợ đến hạn
│   │
│   └── manage.py                   # File điều khiển Django
│
└── docs/
    └── database_design.jpg         # Ảnh thiết kế CSDL viết tay
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
git clone https://github.com/<ten-ban>/<ten-repo>.git
cd camdo_project
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

Kết quả mong đợi:
```
NAME                 STATUS
camdo_django         running
camdo_mariadb        running
camdo_phpmyadmin     running
```

---

### 5.4 Bước 3 — Khởi tạo Django (chỉ làm 1 lần đầu)

```bash
# Nếu chưa có project Django, tạo mới trong container
docker compose exec web django-admin startproject config .

# Tạo app camdo
docker compose exec web python manage.py startapp camdo
```

> **Lưu ý:** Nếu đã clone từ repo này, bước tạo project đã có sẵn — bỏ qua.

---

### 5.5 Bước 4 — Cấu hình settings.py

Sửa file bằng `nano`:
```bash
sudo nano django_app/config/settings.py
```

Tìm và sửa / thêm các phần sau:

```python
# Thêm app camdo vào INSTALLED_APPS
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'camdo',    # <-- thêm dòng này
]

# Cấu hình kết nối MariaDB
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.environ.get('DB_NAME', 'camdo_db'),
        'USER': os.environ.get('DB_USER', 'camdo_user'),
        'PASSWORD': os.environ.get('DB_PASSWORD', 'camdo_pass'),
        'HOST': os.environ.get('DB_HOST', 'db'),
        'PORT': '3306',
        'OPTIONS': {'charset': 'utf8mb4'},
    }
}

# Múi giờ và ngôn ngữ
LANGUAGE_CODE = 'vi'
TIME_ZONE = 'Asia/Ho_Chi_Minh'
USE_TZ = True
```

Lưu file: `Ctrl+O` → `Enter` → `Ctrl+X`

---

### 5.6 Bước 5 — Tạo bảng CSDL (Migration)

```bash
# Tạo file migration từ models.py
docker compose exec web python manage.py makemigrations camdo

# Apply migration vào MariaDB (Django tự tạo bảng)
docker compose exec web python manage.py migrate
```

> **Kiểm chứng:** Mở PhpMyAdmin tại `http://localhost:8080`
> → Đăng nhập (user: `camdo_user`, pass: `camdo_pass`)
> → Vào database `camdo_db` → Thấy các bảng đã được tạo ✅

---

### 5.7 Bước 6 — Tạo tài khoản admin

```bash
docker compose exec web python manage.py createsuperuser
```

Nhập theo thứ tự:
```
Username: admin
Email: admin@example.com
Password: ••••••••
Password (again): ••••••••
```

---

### 5.8 Bước 7 — Truy cập hệ thống

| Địa chỉ | Chức năng |
|---|---|
| `http://localhost:8000/admin` | Trang quản trị Django |
| `http://localhost:8000/` | Trang con nợ đến hạn |
| `http://localhost:8080` | PhpMyAdmin xem CSDL |

---

## 6. Chi Tiết Các File

### 6.1 `django_app/Dockerfile`

```dockerfile
# Dùng Python 3.11 slim làm base image (nhẹ, không có tool thừa)
FROM python:3.11-slim

# Cài gói hệ thống để build mysqlclient (driver kết nối MariaDB)
# gcc: trình biên dịch C
# default-libmysqlclient-dev: thư viện header MySQL/MariaDB
# pkg-config: hỗ trợ tìm thư viện khi build
RUN apt-get update && apt-get install -y \
    gcc \
    default-libmysqlclient-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Đặt thư mục làm việc trong container
WORKDIR /app

# Copy requirements trước để tận dụng Docker layer cache
# (Nếu requirements.txt không đổi, bước pip install được cache lại)
COPY requirements.txt .

# Cài các thư viện Python
RUN pip install --no-cache-dir -r requirements.txt

# Copy toàn bộ source code vào container
COPY . .

# Expose cổng Django
EXPOSE 8000
```

---

### 6.2 `django_app/requirements.txt`

```txt
# Django - web framework chính, phiên bản LTS
Django==4.2.13

# mysqlclient - driver kết nối Django với MariaDB/MySQL
# (cần gcc và libmysqlclient-dev ở Dockerfile để build)
mysqlclient==2.2.4

# Pillow - xử lý ảnh (cần nếu dùng ImageField trong models)
Pillow==10.3.0
```

---

### 6.3 `docker-compose.yml`

```yaml
version: '3.8'

services:

  # ===== SERVICE 1: MariaDB =====
  db:
    image: mariadb:10.11
    container_name: camdo_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: camdo_db
      MYSQL_USER: camdo_user
      MYSQL_PASSWORD: camdo_pass
    volumes:
      - mariadb_data:/var/lib/mysql    # Lưu data bền vững
    ports:
      - "3306:3306"
    networks:
      - camdo_net

  # ===== SERVICE 2: PhpMyAdmin =====
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: camdo_phpmyadmin
    restart: always
    environment:
      PMA_HOST: db
      PMA_USER: camdo_user
      PMA_PASSWORD: camdo_pass
    ports:
      - "8080:80"
    depends_on:
      - db
    networks:
      - camdo_net

  # ===== SERVICE 3: Django =====
  web:
    build: ./django_app              # Build từ Dockerfile
    container_name: camdo_django
    restart: always
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./django_app:/app            # Mount thư mục để nano edit trực tiếp
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_NAME=camdo_db
      - DB_USER=camdo_user
      - DB_PASSWORD=camdo_pass
    networks:
      - camdo_net

volumes:
  mariadb_data:

networks:
  camdo_net:
    driver: bridge
```

---

### 6.4 `camdo/models.py` — Định nghĩa CSDL

```python
from django.db import models


class KhachHang(models.Model):
    """Thông tin khách hàng / con nợ"""
    ho_ten = models.CharField(max_length=100, verbose_name="Họ tên")
    so_dien_thoai = models.CharField(max_length=15, verbose_name="Số điện thoại")
    so_cmnd = models.CharField(max_length=20, unique=True, verbose_name="Số CMND/CCCD")
    dia_chi = models.TextField(blank=True, verbose_name="Địa chỉ")
    ngay_tao = models.DateField(auto_now_add=True)

    class Meta:
        verbose_name = "Khách hàng"
        verbose_name_plural = "Khách hàng"

    def __str__(self):
        return f"{self.ho_ten} - {self.so_cmnd}"


class NhanVien(models.Model):
    """Thông tin nhân viên tiệm cầm đồ"""
    ho_ten = models.CharField(max_length=100, verbose_name="Họ tên")
    so_dien_thoai = models.CharField(max_length=15, verbose_name="Số điện thoại")
    chuc_vu = models.CharField(max_length=50, verbose_name="Chức vụ")

    class Meta:
        verbose_name = "Nhân viên"
        verbose_name_plural = "Nhân viên"

    def __str__(self):
        return f"{self.ho_ten} ({self.chuc_vu})"


class DanhMucTaiSan(models.Model):
    """Danh mục loại tài sản: điện thoại, vàng, xe máy..."""
    ten_danh_muc = models.CharField(max_length=100, verbose_name="Tên danh mục")
    mo_ta = models.TextField(blank=True, verbose_name="Mô tả")

    class Meta:
        verbose_name = "Danh mục tài sản"
        verbose_name_plural = "Danh mục tài sản"

    def __str__(self):
        return self.ten_danh_muc


class HopDongCamDo(models.Model):
    """Hợp đồng cầm đồ — nghiệp vụ chính của tiệm"""
    TRANG_THAI_CHOICES = [
        ('dang_cam', 'Đang cầm'),
        ('da_chuoc', 'Đã chuộc'),
        ('qua_han', 'Quá hạn'),
        ('xu_ly', 'Đang xử lý tài sản'),
    ]

    # FK: Django admin tự render thành dropdown chọn text
    khach_hang = models.ForeignKey(
        KhachHang, on_delete=models.PROTECT, verbose_name="Khách hàng"
    )
    nhan_vien = models.ForeignKey(
        NhanVien, on_delete=models.PROTECT, verbose_name="Nhân viên lập HĐ"
    )
    so_hop_dong = models.CharField(max_length=20, unique=True, verbose_name="Số hợp đồng")
    ngay_cam = models.DateField(verbose_name="Ngày cầm")
    ngay_dao_han = models.DateField(verbose_name="Ngày đáo hạn")
    so_tien_cho_vay = models.DecimalField(
        max_digits=15, decimal_places=0, verbose_name="Số tiền cho vay (VNĐ)"
    )
    lai_suat_thang = models.DecimalField(
        max_digits=5, decimal_places=2, verbose_name="Lãi suất/tháng (%)"
    )
    trang_thai = models.CharField(
        max_length=20, choices=TRANG_THAI_CHOICES,
        default='dang_cam', verbose_name="Trạng thái"
    )
    ghi_chu = models.TextField(blank=True, verbose_name="Ghi chú")

    class Meta:
        verbose_name = "Hợp đồng cầm đồ"
        verbose_name_plural = "Hợp đồng cầm đồ"

    def __str__(self):
        return f"HĐ {self.so_hop_dong} - {self.khach_hang.ho_ten}"


class TaiSan(models.Model):
    """Tài sản cụ thể gắn với hợp đồng"""
    hop_dong = models.ForeignKey(
        HopDongCamDo, on_delete=models.CASCADE,
        related_name='tai_san', verbose_name="Hợp đồng"
    )
    danh_muc = models.ForeignKey(
        DanhMucTaiSan, on_delete=models.PROTECT, verbose_name="Danh mục"
    )
    ten_tai_san = models.CharField(max_length=200, verbose_name="Tên tài sản")
    mo_ta_tai_san = models.TextField(blank=True, verbose_name="Mô tả chi tiết")
    gia_tri_dinh_gia = models.DecimalField(
        max_digits=15, decimal_places=0, verbose_name="Giá trị định giá (VNĐ)"
    )

    class Meta:
        verbose_name = "Tài sản"
        verbose_name_plural = "Tài sản"

    def __str__(self):
        return f"{self.ten_tai_san} — HĐ {self.hop_dong.so_hop_dong}"


class LichSuThanhToan(models.Model):
    """Lịch sử thanh toán / gia hạn hợp đồng"""
    LOAI_CHOICES = [
        ('tra_lai', 'Trả lãi'),
        ('gia_han', 'Gia hạn'),
        ('chuoc_hang', 'Chuộc hàng'),
    ]

    hop_dong = models.ForeignKey(
        HopDongCamDo, on_delete=models.CASCADE,
        related_name='lich_su_thanh_toan', verbose_name="Hợp đồng"
    )
    ngay_thanh_toan = models.DateField(verbose_name="Ngày thanh toán")
    so_tien = models.DecimalField(
        max_digits=15, decimal_places=0, verbose_name="Số tiền (VNĐ)"
    )
    loai_thanh_toan = models.CharField(
        max_length=20, choices=LOAI_CHOICES, verbose_name="Loại thanh toán"
    )
    ghi_chu = models.TextField(blank=True, verbose_name="Ghi chú")

    class Meta:
        verbose_name = "Lịch sử thanh toán"
        verbose_name_plural = "Lịch sử thanh toán"

    def __str__(self):
        return f"{self.loai_thanh_toan} — {self.ngay_thanh_toan} — HĐ {self.hop_dong.so_hop_dong}"
```

---

### 6.5 `camdo/admin.py` — Cấu hình trang Admin

```python
from django.contrib import admin
from .models import KhachHang, NhanVien, DanhMucTaiSan, HopDongCamDo, TaiSan, LichSuThanhToan


@admin.register(KhachHang)
class KhachHangAdmin(admin.ModelAdmin):
    list_display = ['ho_ten', 'so_cmnd', 'so_dien_thoai', 'dia_chi']
    search_fields = ['ho_ten', 'so_cmnd']


@admin.register(NhanVien)
class NhanVienAdmin(admin.ModelAdmin):
    list_display = ['ho_ten', 'chuc_vu', 'so_dien_thoai']


@admin.register(DanhMucTaiSan)
class DanhMucAdmin(admin.ModelAdmin):
    list_display = ['ten_danh_muc', 'mo_ta']


class TaiSanInline(admin.TabularInline):
    """Hiển thị tài sản ngay trong trang hợp đồng"""
    model = TaiSan
    extra = 1


class LichSuInline(admin.TabularInline):
    """Hiển thị lịch sử thanh toán ngay trong trang hợp đồng"""
    model = LichSuThanhToan
    extra = 0


@admin.register(HopDongCamDo)
class HopDongAdmin(admin.ModelAdmin):
    list_display = ['so_hop_dong', 'khach_hang', 'so_tien_cho_vay',
                    'ngay_cam', 'ngay_dao_han', 'trang_thai']
    list_filter = ['trang_thai', 'ngay_dao_han']
    search_fields = ['so_hop_dong', 'khach_hang__ho_ten']
    inlines = [TaiSanInline, LichSuInline]


@admin.register(TaiSan)
class TaiSanAdmin(admin.ModelAdmin):
    list_display = ['ten_tai_san', 'danh_muc', 'hop_dong', 'gia_tri_dinh_gia']


@admin.register(LichSuThanhToan)
class LichSuAdmin(admin.ModelAdmin):
    list_display = ['hop_dong', 'ngay_thanh_toan', 'so_tien', 'loai_thanh_toan']
```

---

### 6.6 `camdo/views.py` — Logic lấy dữ liệu con nợ đến hạn

```python
from django.shortcuts import render
from django.utils import timezone
from .models import HopDongCamDo


def home_page(request):
    """
    View trang chủ: lấy danh sách hợp đồng đã đến hạn hoặc quá hạn
    mà khách hàng chưa chuộc → truyền vào template qua context
    """
    hom_nay = timezone.now().date()

    hop_dong_qua_han = HopDongCamDo.objects.filter(
        ngay_dao_han__lte=hom_nay,
        trang_thai__in=['dang_cam', 'qua_han']
    ).select_related('khach_hang', 'nhan_vien').order_by('ngay_dao_han')

    context = {
        'hop_dong_qua_han': hop_dong_qua_han,
        'hom_nay': hom_nay,
        'tong_so': hop_dong_qua_han.count(),
    }
    return render(request, 'camdo/home.html', context)
```

---

### 6.7 `camdo/templates/camdo/home.html` — Template Jinja2

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Tiệm Cầm Đồ — Con Nợ Đến Hạn</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
        .header { background: #c0392b; color: white; padding: 20px 40px; }
        .header h1 { margin: 0; }
        .header p  { margin: 4px 0 0; opacity: 0.85; font-size: 14px; }
        .container { max-width: 1100px; margin: 30px auto; padding: 0 20px; }
        .alert { background: #fff3cd; border: 1px solid #ffc107; border-radius: 8px;
                 padding: 16px 20px; margin-bottom: 24px; }
        table { width: 100%; border-collapse: collapse; background: white;
                border-radius: 8px; overflow: hidden;
                box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
        th { background: #c0392b; color: white; padding: 12px 16px; text-align: left; }
        td { padding: 12px 16px; border-bottom: 1px solid #eee; }
        tr:last-child td { border-bottom: none; }
        tr:hover td { background: #ffeaea; }
        .badge { padding: 3px 10px; border-radius: 12px; font-size: 12px; color: white; }
        .badge-qua-han { background: #e74c3c; }
        .badge-den-han { background: #f39c12; }
        .so-tien { font-weight: bold; color: #c0392b; }
        .empty { text-align: center; padding: 60px; color: #888; font-size: 18px; }
    </style>
</head>
<body>

<div class="header">
    <h1>🏪 Tiệm Cầm Đồ — Danh Sách Con Nợ Đến Hạn</h1>
    <p>Ngày hôm nay: {{ hom_nay }}</p>
</div>

<div class="container">

    {% if tong_so > 0 %}
    <div class="alert">
        ⚠️ Có <strong>{{ tong_so }}</strong> hợp đồng đến hạn / quá hạn chưa chuộc!
    </div>

    <table>
        <thead>
            <tr>
                <th>Số HĐ</th>
                <th>Khách hàng</th>
                <th>Số điện thoại</th>
                <th>Số tiền vay</th>
                <th>Ngày cầm</th>
                <th>Ngày đáo hạn</th>
                <th>Nhân viên</th>
                <th>Trạng thái</th>
            </tr>
        </thead>
        <tbody>
            {% for hd in hop_dong_qua_han %}
            <tr>
                <td><strong>{{ hd.so_hop_dong }}</strong></td>
                <td>{{ hd.khach_hang.ho_ten }}</td>
                <td>{{ hd.khach_hang.so_dien_thoai }}</td>
                <td class="so-tien">{{ hd.so_tien_cho_vay }} VNĐ</td>
                <td>{{ hd.ngay_cam }}</td>
                <td>{{ hd.ngay_dao_han }}</td>
                <td>{{ hd.nhan_vien.ho_ten }}</td>
                <td>
                    {% if hd.trang_thai == 'qua_han' %}
                        <span class="badge badge-qua-han">Quá hạn</span>
                    {% else %}
                        <span class="badge badge-den-han">Đến hạn hôm nay</span>
                    {% endif %}
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>

    {% else %}
    <div class="empty">
        ✅ Hiện tại không có hợp đồng nào đến hạn!
    </div>
    {% endif %}

</div>
</body>
</html>
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

1. Truy cập `http://localhost:8080`
2. Đăng nhập: user `camdo_user` / pass `camdo_pass`
3. Chọn database `camdo_db`
4. Mở bảng `camdo_hopdongcamdo`
5. Xem cột `khach_hang_id` → lưu **ID số** (khóa ngoại)
6. Mở bảng `camdo_khachhang` → tìm **ID tương ứng** → đó chính là khách hàng

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

## 🛠️ Xử Lý Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Giải pháp |
|---|---|---|
| `Can't connect to MySQL server` | Django chạy trước khi MariaDB sẵn sàng | `docker compose restart web` |
| `No module named 'camdo'` | Chưa thêm `'camdo'` vào `INSTALLED_APPS` | Sửa `settings.py` và restart |
| `Table doesn't exist` | Chưa migrate | `docker compose exec web python manage.py migrate` |
| `OperationalError: (1045)` | Sai thông tin kết nối DB | Kiểm tra `settings.py` và `docker-compose.yml` |
| Trang admin không có bảng | Chưa đăng ký trong `admin.py` | Thêm `@admin.register(Model)` |
| Static files 404 | Thiếu `collectstatic` | `docker compose exec web python manage.py collectstatic` |

---

## 📚 Tài Liệu Tham Khảo

- [Django Documentation](https://docs.djangoproject.com/)
- [Django Admin](https://docs.djangoproject.com/en/4.2/ref/contrib/admin/)
- [Django ORM — Models](https://docs.djangoproject.com/en/4.2/topics/db/models/)
- [Docker Compose](https://docs.docker.com/compose/)
- [MariaDB Docker Image](https://hub.docker.com/_/mariadb)
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/)

---

*Bài tập môn Lập trình Web — Sử dụng Django quản lý tiệm cầm đồ*
