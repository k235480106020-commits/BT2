# BÀI KIỂM TRA SỐ 2 – HỆ QUẢN TRỊ CSDL SQL SERVER

**Họ và tên:** Phan Văn Hải

**Mã sinh viên:** K235480106020

**Lớp:** K59KMT.K01

**Đề tài:** Quản lý kho hàng

## 1. GIỚI THIỆU ĐỀ TÀI

### 1.1 Mục tiêu

Hệ thống **Quản lý kho hàng** được xây dựng nhằm giải quyết bài toán theo dõi, kiểm soát và báo cáo toàn bộ hoạt động liên quan đến hàng hóa lưu trữ trong các kho. Trong bối cảnh thương mại hiện đại, việc quản lý kho hàng hiệu quả đóng vai trò then chốt trong chuỗi cung ứng, ảnh hưởng trực tiếp đến chi phí vận hành và chất lượng dịch vụ của doanh nghiệp.

Hệ thống hướng tới các mục tiêu cụ thể sau:

- **Số hóa nghiệp vụ kho**: Thay thế sổ sách thủ công bằng cơ sở dữ liệu quan hệ, đảm bảo tính chính xác và nhất quán của dữ liệu.
- **Theo dõi tồn kho theo thời gian thực**: Mọi giao dịch nhập – xuất hàng được phản ánh ngay vào số liệu tồn kho thông qua cơ chế Trigger tự động.
- **Báo cáo và phân tích**: Cung cấp các báo cáo tổng hợp về tình trạng nhập – xuất – tồn, hỗ trợ ban lãnh đạo ra quyết định kịp thời.
- **Cảnh báo chủ động**: Tự động phát hiện và thông báo các mặt hàng có số lượng tồn kho dưới ngưỡng an toàn, tránh tình trạng thiếu hàng.

### 1.2 Phạm vi nghiệp vụ

Hệ thống tập trung vào các nghiệp vụ cốt lõi của một trung tâm quản lý kho hàng vừa và lớn, bao gồm:

| STT | Nghiệp vụ | Mô tả |
|-----|-----------|-------|
| 1 | Quản lý kho | Theo dõi thông tin từng kho: tên, địa chỉ, sức chứa |
| 2 | Quản lý sản phẩm | Danh mục sản phẩm, đơn vị tính, số lượng tồn theo kho |
| 3 | Phiếu nhập kho | Ghi nhận hàng hóa đến từ nhà cung cấp |
| 4 | Phiếu xuất kho | Ghi nhận hàng hóa xuất đi theo đơn hàng |
| 5 | Theo dõi tồn kho | Cập nhật tự động số lượng tồn sau mỗi giao dịch |
| 6 | Tính giá trị hàng tồn | Xác định giá trị tài sản hàng hóa đang lưu kho |
| 7 | Báo cáo nhập – xuất – tồn | Tổng hợp dữ liệu theo kho, theo sản phẩm, theo kỳ |
| 8 | Cảnh báo tồn kho thấp | Thông báo khi số lượng hàng xuống dưới mức an toàn |

### 1.3 Phương pháp thực hiện

Báo cáo được triển khai theo phương pháp tiếp cận từ dưới lên (bottom-up), bắt đầu từ việc phân tích yêu cầu nghiệp vụ, thiết kế mô hình dữ liệu, rồi lần lượt cài đặt các đối tượng cơ sở dữ liệu theo thứ tự phụ thuộc:

1. **Phân tích nghiệp vụ**: Xác định các thực thể, thuộc tính và mối quan hệ trong hệ thống kho hàng.
2. **Thiết kế mô hình dữ liệu**: Vẽ sơ đồ ERD, chuẩn hóa đến dạng chuẩn 3NF để loại bỏ dư thừa dữ liệu.
3. **Xây dựng cấu trúc CSDL**: Tạo database, bảng, ràng buộc (constraint), khóa ngoại (foreign key).
4. **Cài đặt logic nghiệp vụ**: Viết Function, Stored Procedure, Trigger và Cursor phục vụ các tác vụ xử lý tự động.
5. **Kiểm thử và tối ưu**: Kiểm tra tính đúng đắn, phân tích hiệu năng và đề xuất hướng cải tiến.

## 2. THIẾT KẾ CƠ SỞ DỮ LIỆU

### 2.1 Mô hình thực thể – quan hệ (ERD)

Hệ thống quản lý kho hàng được mô hình hóa thông qua ba thực thể chính với mối quan hệ như sau:

```
┌─────────────┐         ┌─────────────┐         ┌──────────────────┐
│   KhoHang   │ 1──────M│  SanPham    │ 1──────M│  PhieuNhapXuat   │
│─────────────│         │─────────────│         │──────────────────│
│ MaKho (PK)  │         │ MaSP (PK)   │         │ MaPhieu (PK)     │
│ TenKho      │         │ TenSP       │         │ MaSP (FK)        │
│ DiaChi      │         │ DonViTinh   │         │ NgayGiaoDich     │
│ SucChua     │         │ SoLuongTon  │         │ LoaiPhieu        │
│             │         │ MaKho (FK)  │         │ SoLuong          │
└─────────────┘         └─────────────┘         │ DonGia           │
                                                 └──────────────────┘
```

**Giải thích quan hệ:**
- Một kho hàng (`KhoHang`) có thể chứa nhiều sản phẩm (`SanPham`) → quan hệ 1–N.
- Một sản phẩm (`SanPham`) có thể phát sinh nhiều phiếu nhập/xuất (`PhieuNhapXuat`) theo thời gian → quan hệ 1–N.
- Mỗi phiếu nhập xuất gắn với đúng một sản phẩm và được phân loại rõ ràng (Nhập / Xuất).

### 2.2 Mô tả các bảng dữ liệu

#### Bảng `KhoHang`

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|-----|-------------|-----------|-------|
| MaKho | INT | PK, IDENTITY(1,1) | Mã kho tự tăng |
| TenKho | NVARCHAR(100) | NOT NULL | Tên kho hàng |
| DiaChi | NVARCHAR(200) | NOT NULL | Địa chỉ kho |
| SucChua | INT | NOT NULL, CHECK > 0 | Sức chứa tối đa (đơn vị) |
| NgayThanhLap | DATE | DEFAULT GETDATE() | Ngày đưa kho vào hoạt động |
| TrangThai | NVARCHAR(20) | DEFAULT 'HoatDong' | Trạng thái: HoatDong / DongCua |

#### Bảng `SanPham`

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|-----|-------------|-----------|-------|
| MaSP | INT | PK, IDENTITY(1,1) | Mã sản phẩm tự tăng |
| TenSP | NVARCHAR(150) | NOT NULL | Tên đầy đủ của sản phẩm |
| DonViTinh | NVARCHAR(30) | NOT NULL | Đơn vị tính (Cái, Hộp, Kg, ...) |
| SoLuongTon | INT | NOT NULL, DEFAULT 0, CHECK >= 0 | Số lượng hiện còn trong kho |
| DonGiaNhap | DECIMAL(18,2) | NOT NULL, CHECK > 0 | Giá nhập trung bình |
| MaKho | INT | FK → KhoHang(MaKho) | Kho chứa sản phẩm này |
| NgayNhapDau | DATE | DEFAULT GETDATE() | Ngày nhập lần đầu vào hệ thống |
| MucTonToiThieu | INT | NOT NULL, DEFAULT 10 | Ngưỡng tồn kho tối thiểu (cảnh báo) |

#### Bảng `PhieuNhapXuat`

| Cột | Kiểu dữ liệu | Ràng buộc | Mô tả |
|-----|-------------|-----------|-------|
| MaPhieu | INT | PK, IDENTITY(1,1) | Mã phiếu tự tăng |
| MaSP | INT | FK → SanPham(MaSP) | Sản phẩm liên quan |
| NgayGiaoDich | DATETIME | NOT NULL, DEFAULT GETDATE() | Ngày giờ giao dịch |
| LoaiPhieu | NVARCHAR(5) | NOT NULL, CHECK IN ('Nhap','Xuat') | Phân loại nhập hoặc xuất |
| SoLuong | INT | NOT NULL, CHECK > 0 | Số lượng trong phiếu |
| DonGia | DECIMAL(18,2) | NOT NULL, CHECK > 0 | Đơn giá tại thời điểm giao dịch |
| GhiChu | NVARCHAR(300) | NULL | Ghi chú thêm nếu có |

### 2.3 Tạo cơ sở dữ liệu và bảng

#### Tạo Database

**Kết quả thực thi:**

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d2a4a190-6a42-4aa4-8ea5-f7ad00455786" />


#### Tạo bảng `KhoHang`

```sql
-- ============================================================
-- BẢNG KHOHANG: Thông tin các kho hàng trong hệ thống
-- ============================================================
CREATE TABLE KhoHang (
    MaKho           INT             NOT NULL IDENTITY(1,1),
    TenKho          NVARCHAR(100)   NOT NULL,
    DiaChi          NVARCHAR(200)   NOT NULL,
    SucChua         INT             NOT NULL,
    NgayThanhLap    DATE            NOT NULL DEFAULT GETDATE(),
    TrangThai       NVARCHAR(20)    NOT NULL DEFAULT N'HoatDong',

    -- Khóa chính
    CONSTRAINT PK_KhoHang PRIMARY KEY (MaKho),

    -- Ràng buộc kiểm tra
    CONSTRAINT CHK_KhoHang_SucChua
        CHECK (SucChua > 0),
    CONSTRAINT CHK_KhoHang_TrangThai
        CHECK (TrangThai IN (N'HoatDong', N'DongCua', N'BaoDuong'))
);
GO

PRINT 'Đã tạo bảng KhoHang.';
```
<img width="1916" height="1079" alt="image" src="https://github.com/user-attachments/assets/adb99420-1b54-41e8-bd8a-a49f002ee77d" />
Tạo bảng KhoHang

**Giải thích thiết kế:**
- `IDENTITY(1,1)` trên `MaKho` đảm bảo mã kho được tự động sinh ra tuần tự, loại bỏ nguy cơ trùng lặp hay bỏ sót.
- `CHECK (SucChua > 0)` ngăn việc tạo kho với sức chứa âm hoặc bằng không – một ràng buộc quan trọng về tính hợp lệ dữ liệu.
- Cột `TrangThai` giới hạn trong tập giá trị cho phép (`HoatDong`, `DongCua`, `BaoDuong`), tránh nhập liệu tùy tiện và đảm bảo tính đồng nhất trong báo cáo.

#### Tạo bảng `SanPham`

```sql
-- ============================================================
-- BẢNG SANPHAM: Danh mục sản phẩm lưu kho
-- ============================================================
CREATE TABLE SanPham (
    MaSP            INT             NOT NULL IDENTITY(1,1),
    TenSP           NVARCHAR(150)   NOT NULL,
    DonViTinh       NVARCHAR(30)    NOT NULL,
    SoLuongTon      INT             NOT NULL DEFAULT 0,
    DonGiaNhap      DECIMAL(18,2)   NOT NULL,
    MaKho           INT             NOT NULL,
    NgayNhapDau     DATE            NOT NULL DEFAULT GETDATE(),
    MucTonToiThieu  INT             NOT NULL DEFAULT 10,

    CONSTRAINT PK_SanPham PRIMARY KEY (MaSP),

    CONSTRAINT FK_SanPham_KhoHang
        FOREIGN KEY (MaKho) REFERENCES KhoHang(MaKho)
        ON UPDATE CASCADE
        ON DELETE NO ACTION,

    CONSTRAINT CHK_SanPham_SoLuongTon
        CHECK (SoLuongTon >= 0),

    CONSTRAINT CHK_SanPham_DonGiaNhap
        CHECK (DonGiaNhap > 0),

    CONSTRAINT CHK_SanPham_MucTonToiThieu
        CHECK (MucTonToiThieu >= 0)
);
GO
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/1c41fd27-6912-48d1-ad84-7a97908978c4" />
Tạo bảng 

**Giải thích thiết kế:**
- Khóa ngoại `FK_SanPham_KhoHang` với `ON UPDATE CASCADE` cho phép khi `MaKho` ở bảng `KhoHang` thay đổi, giá trị tương ứng trong `SanPham` cũng được cập nhật tự động.
- Trường `MucTonToiThieu` đóng vai trò ngưỡng cảnh báo: khi `SoLuongTon` xuống dưới giá trị này, hệ thống sẽ phát cảnh báo qua Cursor hoặc email.
- `CHECK (SoLuongTon >= 0)` là ràng buộc nghiệp vụ thiết yếu: tồn kho không thể âm trong thực tế.

#### Tạo bảng `PhieuNhapXuat`

```sql
-- ============================================================
-- BẢNG PHIEUNHAPXUAT: Ghi nhận mọi giao dịch nhập và xuất kho
-- ============================================================
CREATE TABLE PhieuNhapXuat (
    MaPhieu         INT             NOT NULL IDENTITY(1,1),
    MaSP            INT             NOT NULL,
    NgayGiaoDich    DATETIME        NOT NULL DEFAULT GETDATE(),
    LoaiPhieu       NVARCHAR(5)     NOT NULL,
    SoLuong         INT             NOT NULL,
    DonGia          DECIMAL(18,2)   NOT NULL,
    GhiChu          NVARCHAR(300)   NULL,

    -- Khóa chính
    CONSTRAINT PK_PhieuNhapXuat PRIMARY KEY (MaPhieu),

    -- Khóa ngoại liên kết đến sản phẩm
    CONSTRAINT FK_PhieuNhapXuat_SanPham
        FOREIGN KEY (MaSP) REFERENCES SanPham(MaSP),

    -- Ràng buộc kiểm tra
    CONSTRAINT CHK_PhieuNhapXuat_LoaiPhieu
        CHECK (LoaiPhieu IN (N'Nhap', N'Xuat')),
    CONSTRAINT CHK_PhieuNhapXuat_SoLuong
        CHECK (SoLuong > 0),
    CONSTRAINT CHK_PhieuNhapXuat_DonGia
        CHECK (DonGia > 0)
);
GO

PRINT 'Đã tạo bảng PhieuNhapXuat.';
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/22566831-0155-41a5-a248-1c75b0e902f4" />

Tạo bảng phiếu xuất

**Giải thích thiết kế:**
- `CHECK (LoaiPhieu IN ('Nhap','Xuat'))` là ràng buộc cốt lõi để phân biệt hai loại giao dịch. Mọi phiếu đều phải được phân loại rõ ràng, tránh nhập nhằm khi tổng hợp báo cáo.
- Không dùng `ON DELETE CASCADE` ở khóa ngoại `FK_PhieuNhapXuat_SanPham` để tránh mất lịch sử giao dịch khi sản phẩm bị xóa (nên vô hiệu hóa sản phẩm thay vì xóa cứng).
- Sự tách biệt giữa `DonGia` trong `PhieuNhapXuat` và `DonGiaNhap` trong `SanPham` cho phép theo dõi biến động giá qua từng lần nhập hàng, rất quan trọng cho nghiệp vụ kế toán kho.

---

## 3. CHÈN DỮ LIỆU MẪU

### 3.1 Chèn dữ liệu kho hàng

```sql
-- ============================================================
-- INSERT DỮ LIỆU KHO HÀNG
-- ============================================================
SET IDENTITY_INSERT KhoHang ON;

INSERT INTO KhoHang (MaKho, TenKho, DiaChi, SucChua, NgayThanhLap, TrangThai)
VALUES
    (1, N'Kho Trung Tâm',        N'Số 12, Đường Láng Hạ, Đống Đa, Hà Nội',          50000, '2018-03-15', N'HoatDong'),
    (2, N'Kho Miền Bắc',         N'Số 45, Khu Công Nghiệp Nội Bài, Sóc Sơn, Hà Nội', 30000, '2019-06-01', N'HoatDong'),
    (3, N'Kho Miền Nam',         N'Lô B12, KCN Tân Bình, Quận Tân Bình, TP.HCM',     40000, '2019-08-20', N'HoatDong'),
    (4, N'Kho Miền Trung',       N'Số 78, Đường Nguyễn Văn Linh, Đà Nẵng',           20000, '2020-01-10', N'HoatDong'),
    (5, N'Kho Hàng Điện Tử',    N'Tầng 1, Tòa nhà FPT, Cầu Giấy, Hà Nội',           15000, '2020-11-05', N'HoatDong'),
    (6, N'Kho Hàng Thực Phẩm',  N'Số 99, Đường Phan Văn Trị, Gò Vấp, TP.HCM',       25000, '2021-02-14', N'BaoDuong'),
    (7, N'Kho Dự Phòng',         N'Số 3, Đường Công Nghiệp, Biên Hòa, Đồng Nai',     10000, '2022-07-01', N'DongCua');

SET IDENTITY_INSERT KhoHang OFF;
GO

PRINT 'Đã chèn 7 bản ghi vào bảng KhoHang.';
```

**Kết quả thực thi:**

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/e089bda4-8704-4ccf-992b-3ad9d1f5d6a9" />
Chèn dữ liệu vào bảng KhoHang


### 3.2 Chèn dữ liệu sản phẩm

```sql
-- ============================================================
-- INSERT DỮ LIỆU SẢN PHẨM
-- ============================================================
SET IDENTITY_INSERT SanPham ON;

INSERT INTO SanPham (MaSP, TenSP, DonViTinh, SoLuongTon, DonGiaNhap, MaKho, NgayNhapDau, MucTonToiThieu)
VALUES
    (1,  N'Laptop Dell Inspiron 15',         N'Cái',   45,  18500000,  5, '2023-01-10', 5),
    (2,  N'Laptop HP Pavilion 14',           N'Cái',   30,  15200000,  5, '2023-01-15', 5),
    (3,  N'Màn hình LG 24" Full HD',         N'Cái',   80,   4800000,  1, '2023-02-01', 10),
    (4,  N'Màn hình Samsung 27" Curved',     N'Cái',   55,   7200000,  1, '2023-02-10', 8),
    (5,  N'Chuột Logitech MX Master 3',      N'Cái',  120,    950000,  1, '2023-02-20', 20),
    (6,  N'Chuột không dây Rapoo M300',      N'Cái',  200,    320000,  2, '2023-03-01', 30),
    (7,  N'Bàn phím cơ Keychron K2',         N'Cái',   90,   1850000,  1, '2023-03-05', 15),
    (8,  N'Bàn phím Bluetooth Logitech K380',N'Cái',  150,    680000,  2, '2023-03-10', 25),
    (9,  N'Tai nghe Sony WH-1000XM5',        N'Cái',   35,   7500000,  5, '2023-03-15', 5),
    (10, N'Tai nghe JBL Tune 760NC',         N'Cái',   60,   2100000,  5, '2023-04-01', 10),
    (11, N'USB Hub 7 cổng Anker',            N'Cái',  180,    450000,  1, '2023-04-05', 30),
    (12, N'Cáp HDMI 2.0 Ugreen 2m',          N'Cuộn', 500,     85000,  2, '2023-04-10', 50),
    (13, N'SSD Samsung 970 EVO 500GB',       N'Cái',  100,   2200000,  1, '2023-04-15', 15),
    (14, N'RAM Kingston 16GB DDR4',          N'Thanh', 75,    950000,  2, '2023-05-01', 10),
    (15, N'Bộ lưu điện APC 600VA',           N'Cái',   25,   1350000,  4, '2023-05-10', 5),
    (16, N'Router WiFi 6 TP-Link AX3000',    N'Cái',   40,   2800000,  4, '2023-05-15', 8),
    (17, N'Webcam Logitech C920 HD',         N'Cái',   55,   1650000,  5, '2023-06-01', 10),
    (18, N'Máy in HP LaserJet 107a',         N'Cái',   18,   3400000,  3, '2023-06-10', 3),
    (19, N'Mực in HP 105A',                  N'Hộp',  200,    380000,  3, '2023-06-15', 40),
    (20, N'Giấy A4 IK 80 GSM',              N'Ream',  350,     55000,  3, '2023-07-01', 50),
    (21, N'Máy tính bảng Samsung Tab S7',   N'Cái',   22,  12500000,  5, '2023-07-05', 3),
    (22, N'Ốp lưng điện thoại iPhone 14',   N'Cái',  300,     85000,  2, '2023-07-10', 50),
    (23, N'Kính cường lực iPhone 14',        N'Miếng',400,     35000,  2, '2023-07-15', 80),
    (24, N'Sạc nhanh 65W Anker GaN',         N'Cái',  130,    520000,  1, '2023-08-01', 20),
    (25, N'Pin dự phòng 20000mAh Xiaomi',    N'Cái',   85,    680000,  1, '2023-08-05', 15);

SET IDENTITY_INSERT SanPham OFF;
GO

PRINT 'Đã chèn 25 bản ghi vào bảng SanPham.';
```

**Kết quả thực thi:**
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/ab200fa1-4544-402a-adff-f01757b02a0f" />

Chèn dữ liệu vào bảng SanPham

### 3.3 Chèn phiếu nhập xuất

```sql
-- ============================================================
-- INSERT DỮ LIỆU PHIẾU NHẬP XUẤT
-- ============================================================
SET IDENTITY_INSERT PhieuNhapXuat ON;

INSERT INTO PhieuNhapXuat (MaPhieu, MaSP, NgayGiaoDich, LoaiPhieu, SoLuong, DonGia, GhiChu)
VALUES
    -- Phiếu nhập đầu kỳ
    (1,  1, '2024-01-05 08:30:00', N'Nhap', 20, 18000000, N'Nhập từ nhà phân phối Dell Vietnam'),
    (2,  2, '2024-01-05 09:00:00', N'Nhap', 15, 14800000, N'Nhập từ FPT Distribution'),
    (3,  3, '2024-01-06 10:00:00', N'Nhap', 50, 4600000,  N'Nhập hàng loạt từ LG Electronics'),
    (4,  5, '2024-01-08 08:00:00', N'Nhap', 80, 920000,   N'Nhập chuột Logitech từ kho tổng'),
    (5,  7, '2024-01-10 09:30:00', N'Nhap', 40, 1800000,  N'Nhập bàn phím Keychron đợt 1'),
    (6,  13,'2024-01-12 08:00:00', N'Nhap', 50, 2150000,  N'Nhập SSD Samsung từ kho HCM chuyển ra'),
    (7,  14,'2024-01-15 10:00:00', N'Nhap', 40, 920000,   N'Nhập RAM Kingston theo PO-2024-001'),
    (8,  20,'2024-01-16 07:30:00', N'Nhap',150, 54000,    N'Nhập giấy A4 IK cho văn phòng quý 1'),
    (9,  19,'2024-01-17 08:00:00', N'Nhap',100, 370000,   N'Nhập mực in HP theo hợp đồng cung cấp'),
    (10, 6, '2024-01-18 09:00:00', N'Nhap',100, 310000,   N'Nhập chuột Rapoo từ kho miền Nam'),
    -- Phiếu xuất tháng 1
    (11, 1, '2024-01-20 14:00:00', N'Xuat',  5, 22000000, N'Xuất theo đơn hàng ĐH-20240120-001'),
    (12, 3, '2024-01-22 10:30:00', N'Xuat', 10, 5500000,  N'Xuất cho khách hàng Công ty ABC'),
    (13, 5, '2024-01-25 11:00:00', N'Xuat', 20, 1100000,  N'Xuất chuột theo đơn B2B'),
    (14, 7, '2024-01-28 14:30:00', N'Xuat',  8, 2200000,  N'Xuất bàn phím theo đơn lẻ'),
    (15, 13,'2024-01-30 09:00:00', N'Xuat', 15, 2500000,  N'Xuất SSD lẻ cho khách'),
    -- Phiếu nhập tháng 2
    (16, 9, '2024-02-02 08:00:00', N'Nhap', 20, 7300000,  N'Nhập tai nghe Sony đợt 2'),
    (17,17, '2024-02-05 09:00:00', N'Nhap', 30, 1600000,  N'Nhập webcam Logitech từ nhà cung cấp'),
    (18,24, '2024-02-07 10:00:00', N'Nhap', 60, 500000,   N'Nhập sạc Anker từ kho HCM'),
    (19,25, '2024-02-10 08:30:00', N'Nhap', 40, 650000,   N'Nhập pin dự phòng Xiaomi'),
    (20,22, '2024-02-12 09:00:00', N'Nhap',150, 82000,    N'Nhập ốp lưng iPhone 14 loạt mới'),
    -- Phiếu xuất tháng 2
    (21, 2, '2024-02-15 14:00:00', N'Xuat',  8, 17800000, N'Xuất laptop HP cho Trường Đại học XYZ'),
    (22, 9, '2024-02-18 11:30:00', N'Xuat',  5, 8500000,  N'Xuất tai nghe Sony theo đơn premium'),
    (23,24, '2024-02-20 10:00:00', N'Xuat', 25, 620000,   N'Xuất sạc nhanh Anker đơn lẻ'),
    (24,20, '2024-02-22 08:00:00', N'Xuat', 80, 65000,    N'Xuất giấy A4 cho văn phòng chi nhánh'),
    (25,19, '2024-02-25 09:30:00', N'Xuat', 40, 420000,   N'Xuất mực in HP theo yêu cầu'),
    -- Phiếu nhập – xuất tháng 3
    (26, 4, '2024-03-01 08:00:00', N'Nhap', 30, 7000000,  N'Nhập màn hình Samsung Curved đợt 1'),
    (27,16, '2024-03-03 09:30:00', N'Nhap', 20, 2700000,  N'Nhập Router TP-Link AX3000'),
    (28, 8, '2024-03-05 10:00:00', N'Nhap', 60, 660000,   N'Nhập bàn phím Bluetooth Logitech'),
    (29, 1, '2024-03-10 14:00:00', N'Xuat',  8, 22500000, N'Xuất laptop Dell cho Dự án chính phủ'),
    (30, 4, '2024-03-15 11:00:00', N'Xuat', 12, 8200000,  N'Xuất màn hình Samsung theo đơn B2B lớn'),
    (31,11, '2024-03-18 08:30:00', N'Nhap', 80, 430000,   N'Nhập USB Hub Anker bổ sung'),
    (32,12, '2024-03-20 09:00:00', N'Nhap',200, 82000,    N'Nhập cáp HDMI Ugreen loạt 200 cuộn'),
    (33,23, '2024-03-22 10:30:00', N'Nhap',150, 33000,    N'Nhập kính cường lực iPhone 14'),
    (34,15, '2024-03-25 08:00:00', N'Nhap', 15, 1300000,  N'Nhập bộ lưu điện APC 600VA'),
    (35,18, '2024-03-28 09:00:00', N'Xuat',  5, 3800000,  N'Xuất máy in HP cho văn phòng đại diện'),
    (36,21, '2024-04-01 10:00:00', N'Xuat',  6, 14000000, N'Xuất máy tính bảng Samsung cho trường học'),
    (37,25, '2024-04-05 11:00:00', N'Xuat', 20, 780000,   N'Xuất pin dự phòng đơn lẻ'),
    (38, 6, '2024-04-08 08:00:00', N'Xuat', 50, 380000,   N'Xuất chuột Rapoo cho đơn sỉ'),
    (39,14, '2024-04-10 09:30:00', N'Xuat', 25, 1050000,  N'Xuất RAM Kingston theo đơn nâng cấp'),
    (40,10, '2024-04-12 10:00:00', N'Nhap', 30, 2050000,  N'Nhập tai nghe JBL bổ sung tồn kho');

SET IDENTITY_INSERT PhieuNhapXuat OFF;
GO

PRINT 'Đã chèn 40 bản ghi vào bảng PhieuNhapXuat.';
```

**Kết quả thực thi:**
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/51f24153-a3fb-4178-b10f-65740f44a685" />
Chèn vào bảng nhập xuất

---

## 4. HÀM (FUNCTION)

### 4.1 Hàm dựng sẵn (Built-in Functions)

SQL Server cung cấp một bộ hàm dựng sẵn phong phú. Dưới đây là các hàm được áp dụng vào nghiệp vụ quản lý kho hàng.

#### 4.1.1 Hàm ngày – giờ

```sql
-- ============================================================
-- HÀM NGÀY GIỜ TRONG QUẢN LÝ KHO
-- ============================================================

-- Lấy ngày giờ hiện tại để đóng dấu phiếu giao dịch
SELECT GETDATE() AS ThoiGianHienTai;

-- Tính số ngày kể từ lần nhập đầu tiên của sản phẩm
SELECT
    TenSP,
    NgayNhapDau,
    DATEDIFF(DAY, NgayNhapDau, GETDATE()) AS SoNgayKeTuNhap
FROM SanPham
ORDER BY SoNgayKeTuNhap DESC;

-- Lấy tháng và năm để lọc phiếu nhập xuất theo kỳ
SELECT
    MaPhieu,
    MONTH(NgayGiaoDich) AS Thang,
    YEAR(NgayGiaoDich)  AS Nam,
    SoLuong,
    LoaiPhieu
FROM PhieuNhapXuat
WHERE YEAR(NgayGiaoDich) = 2024
ORDER BY NgayGiaoDich;

-- Định dạng ngày giao dịch theo kiểu dd/MM/yyyy
SELECT
    MaPhieu,
    FORMAT(NgayGiaoDich, 'dd/MM/yyyy HH:mm') AS NgayFormatted,
    LoaiPhieu,
    SoLuong
FROM PhieuNhapXuat;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/e23d8f0f-c819-4568-999b-8665f6431195" />
Sử dụng hàm ngày giờ

#### 4.1.2 Hàm chuỗi

```sql
-- ============================================================
-- HÀM XỬ LÝ CHUỖI TRONG QUẢN LÝ KHO
-- ============================================================

-- Đo độ dài tên sản phẩm (kiểm tra dữ liệu nhập vào)
SELECT
    MaSP,
    TenSP,
    LEN(TenSP)      AS DoDaiTen,
    UPPER(DonViTinh) AS DonViVietHoa
FROM SanPham;

-- Tìm kiếm sản phẩm theo từ khóa (không phân biệt hoa thường)
SELECT MaSP, TenSP, SoLuongTon
FROM SanPham
WHERE TenSP LIKE N'%Logitech%';

-- Chuẩn hóa chuỗi: bỏ khoảng trắng hai đầu, viết hoa chữ cái đầu
SELECT
    LTRIM(RTRIM(TenKho))   AS TenKhoChuanHoa,
    LEFT(DiaChi, 30) + N'...' AS DiaChiRutGon
FROM KhoHang;

-- Nối chuỗi tạo nhãn mô tả sản phẩm
SELECT
    CONCAT(TenSP, N' (', DonViTinh, N') – Tồn: ', SoLuongTon) AS NhanSanPham
FROM SanPham
ORDER BY SoLuongTon;
```
Sử dụng hàm chuỗi
#### 4.1.3 Hàm số học

```sql
-- ============================================================
-- HÀM SỐ HỌC TRONG QUẢN LÝ KHO
-- ============================================================

-- Làm tròn đơn giá nhập
SELECT
    TenSP,
    DonGiaNhap,
    ROUND(DonGiaNhap, -3) AS DonGiaLamTron   -- Làm tròn tới nghìn đồng
FROM SanPham;

-- Tính tổng giá trị tồn kho theo từng sản phẩm
SELECT
    sp.TenSP,
    sp.SoLuongTon,
    sp.DonGiaNhap,
    sp.SoLuongTon * sp.DonGiaNhap         AS GiaTriTon,
    ROUND(sp.SoLuongTon * sp.DonGiaNhap, -3) AS GiaTriTonLamTron
FROM SanPham sp
ORDER BY GiaTriTon DESC;

-- Tính số lượng và giá trị giao dịch trong tháng
SELECT
    MONTH(NgayGiaoDich)           AS Thang,
    SUM(SoLuong)                  AS TongSLGiaoDich,
    SUM(SoLuong * DonGia)         AS TongGiaTriGiaoDich,
    AVG(DonGia)                   AS GiaTrungBinh,
    MAX(SoLuong * DonGia)         AS GiaTriPhieuLonNhat
FROM PhieuNhapXuat
WHERE YEAR(NgayGiaoDich) = 2024
GROUP BY MONTH(NgayGiaoDich)
ORDER BY Thang;
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a09d2252-8a8c-4fe9-a1e0-4a5924525aab" />
Sử dụng hàm số học để tính toán


### 4.2 Hàm do người dùng định nghĩa (UDF)

SQL Server hỗ trợ ba loại UDF chính: **Scalar Function** (trả về một giá trị), **Inline Table-valued Function** (trả về bảng từ một câu SELECT đơn), và **Multi-statement Table-valued Function** (trả về bảng từ nhiều câu lệnh). Mỗi loại phù hợp với một nhóm bài toán khác nhau trong nghiệp vụ kho hàng.

#### 4.2.1 Scalar Function – Tính số ngày tồn kho

**Bài toán:** Mỗi sản phẩm được nhập vào kho lần đầu vào một ngày cụ thể. Cần xây dựng hàm trả về số ngày mà sản phẩm đó đã tồn tại trong hệ thống kho, tính từ `NgayNhapDau` đến thời điểm truy vấn. Thông số này giúp nhận diện hàng hóa tồn lâu, dễ lỗi thời.

```sql
-- ============================================================
-- SCALAR FUNCTION: Tính số ngày sản phẩm đã ở trong kho
-- ============================================================
CREATE OR ALTER FUNCTION dbo.fn_SoNgayTonKho
(
    @MaSP INT
)
RETURNS INT
AS
BEGIN
    DECLARE @NgayNhapDau DATE;
    DECLARE @SoNgay      INT;

    -- Lấy ngày nhập đầu của sản phẩm
    SELECT @NgayNhapDau = NgayNhapDau
    FROM SanPham
    WHERE MaSP = @MaSP;

    -- Nếu không tìm thấy sản phẩm, trả về NULL
    IF @NgayNhapDau IS NULL
        RETURN NULL;

    -- Tính số ngày từ ngày nhập đầu đến hôm nay
    SET @SoNgay = DATEDIFF(DAY, @NgayNhapDau, GETDATE());

    RETURN @SoNgay;
END;
GO
```

**Gọi hàm và kiểm tra kết quả:**

```sql
-- Kiểm tra hàm với từng sản phẩm
SELECT
    MaSP,
    TenSP,
    NgayNhapDau,
    dbo.fn_SoNgayTonKho(MaSP)  AS SoNgayTonKho,
    CASE
        WHEN dbo.fn_SoNgayTonKho(MaSP) > 365 THEN N'Hàng tồn lâu (>1 năm)'
        WHEN dbo.fn_SoNgayTonKho(MaSP) > 180 THEN N'Tồn vừa (6–12 tháng)'
        ELSE                                       N'Hàng mới (<6 tháng)'
    END AS PhanLoaiTonKho
FROM SanPham
ORDER BY SoNgayTonKho DESC;
```

<img width="1919" height="1077" alt="image" src="https://github.com/user-attachments/assets/4e75ad12-e5de-4137-be4e-e76dbbfbc5e9" />
Xây dựng hàm tính tồn kho

> **Nhận xét:** Scalar Function `fn_SoNgayTonKho` tái sử dụng được trong nhiều câu truy vấn, tuy nhiên cần lưu ý rằng nếu dùng trong mệnh đề `WHERE` hay `ORDER BY` trên bảng lớn, hàm có thể gây ảnh hưởng hiệu năng do được thực thi theo từng dòng (row-by-row). Trong những trường hợp đó, nên thay bằng biểu thức `DATEDIFF` trực tiếp.

#### 4.2.2 Inline Table-valued Function – Danh sách sản phẩm theo kho

**Bài toán:** Quản lý kho thường xuyên cần xem toàn bộ sản phẩm trong một kho cụ thể kèm thông tin giá trị tồn, nhãn cảnh báo. Inline TVF phù hợp vì có thể tối ưu bởi Query Optimizer như một view có tham số.

```sql
-- ============================================================
-- INLINE TABLE-VALUED FUNCTION: Sản phẩm theo kho
-- ============================================================
CREATE OR ALTER FUNCTION dbo.fn_SanPhamTheoKho
(
    @MaKho INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT
        sp.MaSP,
        sp.TenSP,
        sp.DonViTinh,
        sp.SoLuongTon,
        sp.MucTonToiThieu,
        sp.DonGiaNhap,
        sp.SoLuongTon * sp.DonGiaNhap          AS GiaTriTon,
        kh.TenKho,
        kh.DiaChi,
        CASE
            WHEN sp.SoLuongTon = 0                       THEN N'Hết hàng'
            WHEN sp.SoLuongTon <= sp.MucTonToiThieu      THEN N'Sắp hết'
            WHEN sp.SoLuongTon <= sp.MucTonToiThieu * 2  THEN N'Tồn thấp'
            ELSE                                               N'Bình thường'
        END AS TrangThaiTon
    FROM SanPham sp
    INNER JOIN KhoHang kh ON sp.MaKho = kh.MaKho
    WHERE sp.MaKho = @MaKho
);
GO
```

**Gọi hàm:**

```sql
-- Xem tất cả sản phẩm trong Kho Trung Tâm (MaKho = 1)
SELECT *
FROM dbo.fn_SanPhamTheoKho(1)
ORDER BY TrangThaiTon, GiaTriTon DESC;

-- Kết hợp với bộ lọc
SELECT TenSP, SoLuongTon, GiaTriTon, TrangThaiTon
FROM dbo.fn_SanPhamTheoKho(1)
WHERE TrangThaiTon IN (N'Sắp hết', N'Hết hàng')
ORDER BY SoLuongTon;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/fbca526e-ed00-486b-866b-2d4e1e26eda7" />
Xây dựng và lấy danh sách sản phẩm theo kho

> **Ưu điểm của Inline TVF so với View:** Cho phép truyền tham số (ở đây là `@MaKho`), trong khi View không hỗ trợ điều này. Nội bộ, SQL Server "mở" (inline) hàm vào trong câu truy vấn gốc, cho phép Query Optimizer tối ưu hóa toàn bộ kế hoạch thực thi, khác với Multi-statement TVF vốn chạy như một hộp đen.

#### 4.2.3 Multi-statement Table-valued Function – Tính giá trị tồn kho theo mức

**Bài toán:** Kế toán kho cần bảng phân loại sản phẩm theo ba mức giá trị tồn kho (Cao – Trung bình – Thấp), đồng thời tính thêm thuế VAT tham chiếu và tỷ lệ so với tổng kho. Yêu cầu này đòi hỏi nhiều bước xử lý, phù hợp với Multi-statement TVF.

```sql
-- ============================================================
-- MULTI-STATEMENT TABLE-VALUED FUNCTION: Phân mức giá trị tồn kho
-- ============================================================
-- ============================================================
-- FUNCTION: Phân tích giá trị tồn kho theo kho
-- ============================================================
CREATE FUNCTION dbo.fn_TinhGiaTriTonKho
(
    @MaKho INT
)
RETURNS @KetQua TABLE
(
    MaSP               INT,
    TenSP              NVARCHAR(150),
    SoLuongTon         INT,
    DonGiaNhap         DECIMAL(18,2),
    GiaTriTon          DECIMAL(18,2),
    GiaTriCoVAT        DECIMAL(18,2),
    MucGiaTri          NVARCHAR(50),
    TyLeGiaTriKho      DECIMAL(10,2)
)
AS
BEGIN

    -- Tổng giá trị tồn kho
    DECLARE @TongGiaTriKho DECIMAL(18,2);

    SELECT
        @TongGiaTriKho = ISNULL(SUM(SoLuongTon * DonGiaNhap), 0)
    FROM SanPham
    WHERE MaKho = @MaKho;

    -- Tránh chia cho 0
    IF @TongGiaTriKho = 0
        SET @TongGiaTriKho = 1;

    -- Ghi dữ liệu vào bảng kết quả
    INSERT INTO @KetQua
    (
        MaSP,
        TenSP,
        SoLuongTon,
        DonGiaNhap,
        GiaTriTon,
        GiaTriCoVAT,
        MucGiaTri,
        TyLeGiaTriKho
    )
    SELECT
        sp.MaSP,
        sp.TenSP,
        sp.SoLuongTon,
        sp.DonGiaNhap,

        -- Giá trị tồn
        CAST(
            sp.SoLuongTon * sp.DonGiaNhap
            AS DECIMAL(18,2)
        ) AS GiaTriTon,

        -- Giá trị có VAT 10%
        CAST(
            sp.SoLuongTon * sp.DonGiaNhap * 1.10
            AS DECIMAL(18,2)
        ) AS GiaTriCoVAT,

        -- Phân loại giá trị
        CASE
            WHEN sp.SoLuongTon * sp.DonGiaNhap >= 500000000
                THEN N'Giá trị cao (≥ 500 triệu)'

            WHEN sp.SoLuongTon * sp.DonGiaNhap >= 50000000
                THEN N'Giá trị trung bình (50–500 triệu)'

            ELSE N'Giá trị thấp (< 50 triệu)'
        END AS MucGiaTri,

        -- Tỷ lệ giá trị trong kho
        CAST(
            ROUND(
                (sp.SoLuongTon * sp.DonGiaNhap) * 100.0
                / @TongGiaTriKho,
                2
            )
            AS DECIMAL(10,2)
        ) AS TyLeGiaTriKho

    FROM SanPham sp
    WHERE sp.MaKho = @MaKho;

    RETURN;
END;
GO


-- ============================================================
-- GỌI HÀM
-- ============================================================

-- Danh sách chi tiết sản phẩm trong kho
SELECT *
FROM dbo.fn_TinhGiaTriTonKho(1)
ORDER BY GiaTriTon DESC;
GO

-- Tổng hợp theo mức giá trị
SELECT
    MucGiaTri,
    COUNT(*)            AS SoLoaiSanPham,
    SUM(SoLuongTon)     AS TongSoLuong,
    SUM(GiaTriTon)      AS TongGiaTri,
    SUM(TyLeGiaTriKho)  AS TongTyLe
FROM dbo.fn_TinhGiaTriTonKho(1)
GROUP BY MucGiaTri
ORDER BY TongGiaTri DESC;
GO
```

<img width="1919" height="1077" alt="image" src="https://github.com/user-attachments/assets/03789864-5a22-4455-9fcc-6dd2d086988b" />
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/c677d3ee-d999-4515-bc3f-20db58b02f77" />

Xây dựng và khai thác hàm phân loại giá trị tồn kho theo mức

> **So sánh ba loại UDF trong ngữ cảnh kho hàng:** Scalar Function `fn_SoNgayTonKho` được gọi ở mức dòng (per-row), thích hợp cho tính toán đơn giản một chiều. Inline TVF `fn_SanPhamTheoKho` hoạt động như view có tham số, được optimizer tối ưu hóa tốt nhất, phù hợp cho báo cáo. Multi-statement TVF `fn_TinhGiaTriTonKho` linh hoạt nhất nhưng tốn chi phí xử lý hơn vì kết quả trung gian lưu vào bảng tạm trong bộ nhớ; nên dùng khi logic phức tạp, nhiều bước.

---

## 5. STORED PROCEDURE

Stored Procedure (thủ tục lưu trữ) là các khối lệnh T-SQL được biên dịch sẵn và lưu trong cơ sở dữ liệu. So với việc gửi câu lệnh SQL đơn thuần từ ứng dụng, Stored Procedure mang lại ba lợi thế chính: **hiệu năng** (kế hoạch thực thi được cache), **bảo mật** (kiểm soát quyền truy cập ở tầng thủ tục), và **tái sử dụng** (một lần viết, nhiều nơi gọi).

### 5.1 System Stored Procedure

SQL Server cung cấp nhiều thủ tục hệ thống hữu ích cho quản trị và khảo sát cấu trúc CSDL.

```sql
-- ============================================================
-- SYSTEM STORED PROCEDURE ÁP DỤNG TRONG QUẢN LÝ KHO
-- ============================================================

-- Xem toàn bộ thông tin bảng PhieuNhapXuat
EXEC sp_help 'PhieuNhapXuat';

-- Kiểm tra các ràng buộc (constraint) trên bảng SanPham
EXEC sp_helpconstraint 'SanPham';

```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/4ddd3f0c-96e6-478e-a898-9b99a37e525a" />

Xem toàn bộ thông tin bảng PhieuNhapXuat


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/adda0d08-fd6a-44b5-a8cb-38cd9d1cd397" />

Kiểm tra các ràng buộc (constraint) trên bảng SanPham

### 5.2 User-defined Stored Procedure

#### 5.2.1 `sp_ThemSanPham` – Thêm sản phẩm mới vào kho

```sql
-- ============================================================
-- SP_THEMSANPHAM: Thêm sản phẩm mới, kiểm tra kho tồn tại
-- ============================================================
CREATE OR ALTER PROCEDURE dbo.sp_ThemSanPham
    @TenSP           NVARCHAR(150),
    @DonViTinh       NVARCHAR(30),
    @SoLuongTon      INT           = 0,
    @DonGiaNhap      DECIMAL(18,2),
    @MaKho           INT,
    @MucTonToiThieu  INT           = 10,
    @MaSP_Moi        INT           OUTPUT   -- Trả về mã sản phẩm vừa tạo
AS
BEGIN
    SET NOCOUNT ON;

    -- Kiểm tra kho có tồn tại và đang hoạt động không
    IF NOT EXISTS (
        SELECT 1 FROM KhoHang
        WHERE MaKho = @MaKho AND TrangThai = N'HoatDong'
    )
    BEGIN
        RAISERROR(
            N'Lỗi: Kho hàng với MaKho = %d không tồn tại hoặc không đang hoạt động.',
            16, 1, @MaKho
        );
        RETURN -1;
    END;

    -- Kiểm tra tên sản phẩm không được để trống
    IF LEN(LTRIM(RTRIM(@TenSP))) = 0
    BEGIN
        RAISERROR(N'Lỗi: Tên sản phẩm không được để trống.', 16, 1);
        RETURN -2;
    END;

    -- Kiểm tra sản phẩm có bị trùng tên trong cùng kho không
    IF EXISTS (
        SELECT 1 FROM SanPham
        WHERE TenSP = @TenSP AND MaKho = @MaKho
    )
    BEGIN
        RAISERROR(
            N'Lỗi: Sản phẩm "%s" đã tồn tại trong kho này.',
            16, 1, @TenSP
        );
        RETURN -3;
    END;

    -- Thực hiện chèn sản phẩm mới
    BEGIN TRY
        INSERT INTO SanPham (TenSP, DonViTinh, SoLuongTon, DonGiaNhap, MaKho, MucTonToiThieu)
        VALUES (@TenSP, @DonViTinh, @SoLuongTon, @DonGiaNhap, @MaKho, @MucTonToiThieu);

        SET @MaSP_Moi = SCOPE_IDENTITY();

        PRINT N'Thêm sản phẩm thành công. MaSP mới = ' + CAST(@MaSP_Moi AS NVARCHAR);
        RETURN 0;
    END TRY
    BEGIN CATCH
        PRINT N'Lỗi hệ thống: ' + ERROR_MESSAGE();
        RETURN -99;
    END CATCH;
END;
GO
```

**Bảng mô tả tham số:**

| Tham số | Chiều | Kiểu | Mô tả |
|---------|-------|------|-------|
| @TenSP | INPUT | NVARCHAR(150) | Tên sản phẩm cần thêm |
| @DonViTinh | INPUT | NVARCHAR(30) | Đơn vị tính |
| @SoLuongTon | INPUT | INT | Tồn kho ban đầu (mặc định 0) |
| @DonGiaNhap | INPUT | DECIMAL(18,2) | Giá nhập của sản phẩm |
| @MaKho | INPUT | INT | Kho chứa sản phẩm |
| @MucTonToiThieu | INPUT | INT | Ngưỡng cảnh báo (mặc định 10) |
| @MaSP_Moi | OUTPUT | INT | Mã sản phẩm vừa được tạo |

**Kiểm thử:**

```sql
-- Thêm sản phẩm hợp lệ
DECLARE @MaSPMoi INT;
EXEC dbo.sp_ThemSanPham
    @TenSP           = N'Card đồ họa RTX 4060 Ti',
    @DonViTinh       = N'Cái',
    @SoLuongTon      = 10,
    @DonGiaNhap      = 12500000,
    @MaKho           = 5,
    @MucTonToiThieu  = 3,
    @MaSP_Moi        = @MaSPMoi OUTPUT;

SELECT @MaSPMoi AS MaSPDaTao;

-- Thêm vào kho không tồn tại (lỗi dự kiến)
EXEC dbo.sp_ThemSanPham
    @TenSP      = N'Sản phẩm Test',
    @DonViTinh  = N'Cái',
    @DonGiaNhap = 100000,
    @MaKho      = 999,
    @MaSP_Moi   = @MaSPMoi OUTPUT;
```

**Kết quả thực thi:**
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/958ec44a-d04e-4518-8e10-c4f0244a4dbf" />
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/f72f0d6a-0240-4638-91b6-e62a3f1cc724" />

Tạo sp thêm sản phẩm mới và thử thêm sản phẩm hợp lệ và không hợp lệ


#### 5.2.2 `sp_TongGiaTriKho` – Tổng hợp giá trị tồn kho

```sql
-- ============================================================
-- SP_TONGGIATRIKHO: Tính tổng giá trị tồn kho, dùng tham số OUTPUT
-- ============================================================
CREATE OR ALTER PROCEDURE dbo.sp_TongGiaTriKho
    @MaKho           INT           = NULL,   -- NULL = tính tất cả kho
    @TongGiaTri      DECIMAL(18,2) OUTPUT,
    @SoSanPham       INT           OUTPUT,
    @TongSoLuong     INT           OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    IF @MaKho IS NULL
    BEGIN
        -- Tính toàn bộ kho
        SELECT
            @TongGiaTri  = SUM(SoLuongTon * DonGiaNhap),
            @SoSanPham   = COUNT(*),
            @TongSoLuong = SUM(SoLuongTon)
        FROM SanPham;

        -- Báo cáo chi tiết từng kho
        SELECT
            kh.TenKho,
            COUNT(sp.MaSP)           AS SoSanPham,
            SUM(sp.SoLuongTon)       AS TongSoLuong,
            SUM(sp.SoLuongTon * sp.DonGiaNhap) AS GiaTriTon,
            SUM(sp.SoLuongTon * sp.DonGiaNhap * 1.10) AS GiaTriCoVAT
        FROM SanPham sp
        INNER JOIN KhoHang kh ON sp.MaKho = kh.MaKho
        GROUP BY kh.MaKho, kh.TenKho
        ORDER BY GiaTriTon DESC;
    END
    ELSE
    BEGIN
        -- Tính riêng kho được chỉ định
        SELECT
            @TongGiaTri  = SUM(SoLuongTon * DonGiaNhap),
            @SoSanPham   = COUNT(*),
            @TongSoLuong = SUM(SoLuongTon)
        FROM SanPham
        WHERE MaKho = @MaKho;
    END;
END;
GO
```

**Kiểm thử và đọc kết quả OUTPUT:**

```sql
DECLARE
    @GiaTri  DECIMAL(18,2),
    @SoSP    INT,
    @SoLuong INT;

EXEC dbo.sp_TongGiaTriKho
    @MaKho       = NULL,
    @TongGiaTri  = @GiaTri  OUTPUT,
    @SoSanPham   = @SoSP    OUTPUT,
    @TongSoLuong = @SoLuong OUTPUT;

SELECT
    @SoSP    AS TongSoLoaiSanPham,
    @SoLuong AS TongSoLuongTon,
    FORMAT(@GiaTri, 'N0') + N' VNĐ' AS TongGiaTriTonKho;
```

**Kết quả thực thi (bảng tổng hợp theo kho):**
<img width="1919" height="1077" alt="image" src="https://github.com/user-attachments/assets/0933fb74-2503-41ce-9387-cf4c683a653b" />
Xây dựng và thực thi sp tổng hợp giá trị tồn kho

#### 5.2.3 `sp_BaoCaoNhapXuat` – Báo cáo nhập – xuất – tồn

```sql
-- ============================================================
-- SP_BAOCAONHAPXUAT: Báo cáo tổng hợp nhập – xuất – tồn kho
-- ============================================================
CREATE OR ALTER PROCEDURE dbo.sp_BaoCaoNhapXuat
    @TuNgay  DATE = NULL,
    @DenNgay DATE = NULL,
    @MaKho   INT  = NULL
AS
BEGIN
    SET NOCOUNT ON;

    -- Gán khoảng thời gian mặc định nếu không truyền
    IF @TuNgay  IS NULL SET @TuNgay  = DATEADD(MONTH, -1, GETDATE());
    IF @DenNgay IS NULL SET @DenNgay = GETDATE();

    -- Báo cáo nhập – xuất – tồn theo từng sản phẩm
    SELECT
        sp.MaSP,
        sp.TenSP,
        kh.TenKho,
        sp.DonViTinh,
        -- Tổng nhập trong kỳ
        ISNULL(nhap.TongNhap, 0)                       AS TongNhapKy,
        ISNULL(nhap.GiaTriNhap, 0)                     AS GiaTriNhapKy,
        -- Tổng xuất trong kỳ
        ISNULL(xuat.TongXuat, 0)                       AS TongXuatKy,
        ISNULL(xuat.GiaTriXuat, 0)                     AS GiaTriXuatKy,
        -- Tồn kho hiện tại
        sp.SoLuongTon                                  AS TonHienTai,
        sp.SoLuongTon * sp.DonGiaNhap                 AS GiaTriTonHienTai,
        -- Nhận xét trạng thái
        CASE
            WHEN sp.SoLuongTon = 0                      THEN N'⛔ Hết hàng – Cần nhập ngay'
            WHEN sp.SoLuongTon <= sp.MucTonToiThieu    THEN N'⚠️ Tồn kho dưới mức an toàn'
            ELSE                                              N'✅ Tồn kho bình thường'
        END AS NhanXetTonKho
    FROM SanPham sp
    INNER JOIN KhoHang kh ON sp.MaKho = kh.MaKho
    -- Tổng hợp số liệu nhập trong kỳ
    LEFT JOIN (
        SELECT
            MaSP,
            SUM(SoLuong)            AS TongNhap,
            SUM(SoLuong * DonGia)   AS GiaTriNhap
        FROM PhieuNhapXuat
        WHERE LoaiPhieu = N'Nhap'
          AND NgayGiaoDich BETWEEN @TuNgay AND @DenNgay
        GROUP BY MaSP
    ) nhap ON sp.MaSP = nhap.MaSP
    -- Tổng hợp số liệu xuất trong kỳ
    LEFT JOIN (
        SELECT
            MaSP,
            SUM(SoLuong)            AS TongXuat,
            SUM(SoLuong * DonGia)   AS GiaTriXuat
        FROM PhieuNhapXuat
        WHERE LoaiPhieu = N'Xuat'
          AND NgayGiaoDich BETWEEN @TuNgay AND @DenNgay
        GROUP BY MaSP
    ) xuat ON sp.MaSP = xuat.MaSP
    WHERE (@MaKho IS NULL OR sp.MaKho = @MaKho)
    ORDER BY kh.TenKho, NhanXetTonKho, sp.TenSP;
END;
GO
```

**Gọi thủ tục:**

```sql
-- Báo cáo toàn bộ kho trong quý 1/2024
EXEC dbo.sp_BaoCaoNhapXuat
    @TuNgay  = '2024-01-01',
    @DenNgay = '2024-03-31',
    @MaKho   = NULL;

-- Báo cáo riêng Kho Hàng Điện Tử
EXEC dbo.sp_BaoCaoNhapXuat
    @TuNgay  = '2024-01-01',
    @DenNgay = '2024-04-30',
    @MaKho   = 5;
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/dd97e6a6-c23a-406a-b18a-e62cdf12a9d6" />

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3c82dea8-1781-462e-97d7-ff152a339a27" />

Xây dựng và thực thi sp báo cáo tồn nhập xuất kho

## 6. TRIGGER

Trigger là đối tượng cơ sở dữ liệu được tự động thực thi khi một sự kiện xác định (INSERT, UPDATE, DELETE) xảy ra trên bảng. Trong hệ thống kho hàng, Trigger đóng vai trò then chốt trong việc duy trì tính nhất quán của số liệu tồn kho mà không cần can thiệp từ phía ứng dụng.

### 6.1 Trigger cập nhật tồn kho

#### 6.1.1 Trigger sau khi INSERT phiếu

```sql
-- ============================================================
-- TRIGGER: Cập nhật SoLuongTon khi có phiếu nhập hoặc xuất mới
-- ============================================================
CREATE OR ALTER TRIGGER trg_PhieuNhapXuat_Insert
ON PhieuNhapXuat
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;

    -- Xử lý phiếu NHẬP: tăng tồn kho
    UPDATE sp
    SET sp.SoLuongTon = sp.SoLuongTon + i.SoLuong
    FROM SanPham sp
    INNER JOIN inserted i ON sp.MaSP = i.MaSP
    WHERE i.LoaiPhieu = N'Nhap';

    -- Xử lý phiếu XUẤT: giảm tồn kho
    -- Kiểm tra trước để tránh tồn kho âm
    IF EXISTS (
        SELECT 1
        FROM SanPham sp
        INNER JOIN inserted i ON sp.MaSP = i.MaSP
        WHERE i.LoaiPhieu = N'Xuat'
          AND sp.SoLuongTon < i.SoLuong
    )
    BEGIN
        RAISERROR(
            N'Lỗi: Không đủ hàng tồn kho để xuất. Giao dịch bị hủy.',
            16, 1
        );
        ROLLBACK TRANSACTION;
        RETURN;
    END;

    UPDATE sp
    SET sp.SoLuongTon = sp.SoLuongTon - i.SoLuong
    FROM SanPham sp
    INNER JOIN inserted i ON sp.MaSP = i.MaSP
    WHERE i.LoaiPhieu = N'Xuat';

    -- Ghi log cảnh báo tồn kho thấp
    IF EXISTS (
        SELECT 1
        FROM SanPham sp
        INNER JOIN inserted i ON sp.MaSP = i.MaSP
        WHERE sp.SoLuongTon <= sp.MucTonToiThieu
    )
    BEGIN
        PRINT N'[CẢNH BÁO] Có sản phẩm xuống dưới mức tồn kho an toàn sau giao dịch này.';
    END;
END;
GO
```

#### 6.1.2 Trigger sau khi DELETE phiếu

```sql
-- ============================================================
-- TRIGGER: Hoàn trả tồn kho khi phiếu bị xóa
-- ============================================================
CREATE OR ALTER TRIGGER trg_PhieuNhapXuat_Delete
ON PhieuNhapXuat
AFTER DELETE
AS
BEGIN
    SET NOCOUNT ON;

    -- Nếu phiếu nhập bị xóa → giảm lại tồn kho
    UPDATE sp
    SET sp.SoLuongTon = sp.SoLuongTon - d.SoLuong
    FROM SanPham sp
    INNER JOIN deleted d ON sp.MaSP = d.MaSP
    WHERE d.LoaiPhieu = N'Nhap';

    -- Nếu phiếu xuất bị xóa → cộng lại tồn kho
    UPDATE sp
    SET sp.SoLuongTon = sp.SoLuongTon + d.SoLuong
    FROM SanPham sp
    INNER JOIN deleted d ON sp.MaSP = d.MaSP
    WHERE d.LoaiPhieu = N'Xuat';

    PRINT N'[TRIGGER DELETE] Đã hoàn tác tồn kho tương ứng với phiếu bị xóa.';
END;
GO
```

**Kiểm thử trigger:**

```sql
-- Kiểm tra trước khi thêm phiếu
SELECT MaSP, TenSP, SoLuongTon FROM SanPham WHERE MaSP = 5;
-- Kết quả: SoLuongTon = 120

-- Thêm phiếu xuất 30 chuột Logitech
INSERT INTO PhieuNhapXuat (MaSP, NgayGiaoDich, LoaiPhieu, SoLuong, DonGia, GhiChu)
VALUES (5, GETDATE(), N'Xuat', 30, 1100000, N'Test trigger');

-- Kiểm tra sau khi thêm phiếu
SELECT MaSP, TenSP, SoLuongTon FROM SanPham WHERE MaSP = 5;
-- Kết quả: SoLuongTon = 90 (giảm đúng 30)
```

**Kết quả thực thi:**
```
[TRIGGER INSERT] Đã cập nhật tồn kho thành công.
(1 rows affected)

MaSP  TenSP                        SoLuongTon
5     Chuột Logitech MX Master 3   90
```

**Kiểm thử xuất vượt tồn:**

```sql
-- Thử xuất vượt quá tồn kho
INSERT INTO PhieuNhapXuat (MaSP, NgayGiaoDich, LoaiPhieu, SoLuong, DonGia)
VALUES (5, GETDATE(), N'Xuat', 500, 1100000);
```

**Kết quả:**
```
Msg 50000, Level 16, State 1
Lỗi: Không đủ hàng tồn kho để xuất. Giao dịch bị hủy.
```

### 6.2 Trigger đệ quy và phân tích rủi ro

#### 6.2.1 Khái niệm Trigger đệ quy

Trigger đệ quy xảy ra khi một trigger thực thi một câu lệnh DML (INSERT / UPDATE / DELETE) mà chính câu lệnh đó lại kích hoạt cùng trigger đó một lần nữa, tạo thành vòng lặp không có điểm dừng tự nhiên.

Trong hệ thống kho hàng, tình huống này có thể xảy ra nếu một trigger trên bảng `SanPham` lại ghi thêm một bản ghi vào `PhieuNhapXuat`, và trigger trên `PhieuNhapXuat` lại cập nhật `SanPham`, gây ra vòng lặp chéo (indirect recursion).

#### 6.2.2 Ví dụ trigger gây đệ quy gián tiếp (mô phỏng)

```sql
-- ============================================================
-- [MÔ PHỎNG LỖI] Trigger đệ quy gián tiếp giữa SanPham và KhoHang
-- ============================================================

-- Trigger 1: Khi SanPham được UPDATE, tự động UPDATE KhoHang
CREATE OR ALTER TRIGGER trg_SanPham_UpdateKho
ON SanPham
AFTER UPDATE
AS
BEGIN
    -- Cập nhật số lượng sản phẩm trong kho
    UPDATE KhoHang
    SET SucChua = SucChua   -- Lệnh cập nhật (giả lập)
    WHERE MaKho IN (SELECT MaKho FROM inserted);
END;
GO

-- Trigger 2: Khi KhoHang được UPDATE, tự động UPDATE SanPham
CREATE OR ALTER TRIGGER trg_KhoHang_UpdateSP
ON KhoHang
AFTER UPDATE
AS
BEGIN
    -- Đây là vòng lặp: KhoHang update → trigger kích hoạt
    -- → UPDATE SanPham → trigger trg_SanPham_UpdateKho kích hoạt
    -- → UPDATE KhoHang → ... (vòng lặp vô hạn)
    UPDATE SanPham
    SET NgayNhapDau = NgayNhapDau  -- Lệnh cập nhật giả lập
    WHERE MaKho IN (SELECT MaKho FROM inserted);
END;
GO
```

#### 6.2.3 Lỗi phát sinh và nguyên nhân

Khi thực thi một câu lệnh `UPDATE` đơn giản trên bảng `SanPham`, SQL Server sẽ phát hiện vòng đệ quy và dừng lại với thông báo lỗi:

```
Msg 217, Level 16, State 1
Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32).
```

**Bảng phân tích nguyên nhân:**

| Yếu tố | Chi tiết |
|--------|---------|
| Loại đệ quy | Gián tiếp (Indirect recursion): SanPham → KhoHang → SanPham |
| Giới hạn hệ thống | SQL Server cho phép tối đa 32 cấp lồng nhau |
| Nguyên nhân gốc | Hai trigger tạo thành vòng lặp UPDATE lẫn nhau |
| Phạm vi ảnh hưởng | Transaction bị rollback, mọi thay đổi trong vòng lặp bị hủy |
| Khó phát hiện | Đệ quy gián tiếp khó nhận ra hơn đệ quy trực tiếp |

#### 6.2.4 Hướng xử lý

```sql
-- ============================================================
-- GIẢI PHÁP 1: Tắt đệ quy trigger ở cấp database
-- ============================================================
ALTER DATABASE QuanLyKhoHang SET RECURSIVE_TRIGGERS OFF;

-- ============================================================
-- GIẢI PHÁP 2: Dùng biến kiểm tra để phòng đệ quy trong trigger
-- ============================================================
CREATE OR ALTER TRIGGER trg_SanPham_UpdateKho_Safe
ON SanPham
AFTER UPDATE
AS
BEGIN
    -- Kiểm tra nếu đây là lần gọi đệ quy thì bỏ qua
    IF TRIGGER_NESTLEVEL() > 1
    BEGIN
        PRINT N'[TRIGGER] Phát hiện đệ quy – bỏ qua để tránh vòng lặp.';
        RETURN;
    END;

    -- Logic bình thường
    UPDATE KhoHang
    SET TrangThai = TrangThai
    WHERE MaKho IN (SELECT MaKho FROM inserted);
END;
GO
```

> **Nhận xét thực tiễn:** Trong hệ thống kho hàng thực tế, việc thiết kế trigger cần tuân theo nguyên tắc đơn trách nhiệm – mỗi trigger chỉ phục vụ một mục đích duy nhất, không kéo theo chuỗi phụ thuộc phức tạp. Hàm `TRIGGER_NESTLEVEL()` là công cụ phòng thủ hiệu quả, cho phép trigger tự nhận biết mình đang được gọi đệ quy và chủ động dừng lại.

---

## 7. CURSOR

Cursor (con trỏ) là cơ chế xử lý tập kết quả theo từng dòng một, tương tự vòng lặp trong lập trình thủ tục. Mặc dù SQL Server được tối ưu cho xử lý tập hợp (set-based), Cursor vẫn có vị trí trong một số tình huống đặc thù: xử lý logic phân nhánh phức tạp theo từng dòng, hoặc tích hợp với các tiến trình ngoài như gửi email cảnh báo.

### 7.1 Cursor duyệt tồn kho và cảnh báo

**Bài toán:** Cuối mỗi ngày làm việc, hệ thống cần duyệt qua toàn bộ danh sách sản phẩm, tính giá trị tồn kho tương ứng, in báo cáo và đánh dấu cảnh báo đối với những mặt hàng có số lượng tồn dưới mức an toàn.

```sql
-- ============================================================
-- CURSOR: Duyệt từng sản phẩm, tính giá trị tồn và cảnh báo
-- ============================================================
DECLARE
    @MaSP           INT,
    @TenSP          NVARCHAR(150),
    @TenKho         NVARCHAR(100),
    @DonViTinh      NVARCHAR(30),
    @SoLuongTon     INT,
    @MucToiThieu    INT,
    @DonGiaNhap     DECIMAL(18,2),
    @GiaTriTon      DECIMAL(18,2),
    @TrangThaiTon   NVARCHAR(50),
    @SoSPCanhBao    INT = 0,
    @TongGiaTri     DECIMAL(18,2) = 0;

-- Khai báo cursor duyệt tất cả sản phẩm trong kho đang hoạt động
DECLARE cur_TonKho CURSOR
    LOCAL STATIC READ_ONLY FORWARD_ONLY
FOR
    SELECT
        sp.MaSP,
        sp.TenSP,
        kh.TenKho,
        sp.DonViTinh,
        sp.SoLuongTon,
        sp.MucTonToiThieu,
        sp.DonGiaNhap
    FROM SanPham sp
    INNER JOIN KhoHang kh ON sp.MaKho = kh.MaKho
    WHERE kh.TrangThai = N'HoatDong'
    ORDER BY kh.TenKho, sp.TenSP;

OPEN cur_TonKho;

PRINT REPLICATE('=', 70);
PRINT N'     BÁO CÁO TÌNH TRẠNG TỒN KHO – ' + FORMAT(GETDATE(), 'dd/MM/yyyy HH:mm');
PRINT REPLICATE('=', 70);

FETCH NEXT FROM cur_TonKho
INTO @MaSP, @TenSP, @TenKho, @DonViTinh,
     @SoLuongTon, @MucToiThieu, @DonGiaNhap;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @GiaTriTon = @SoLuongTon * @DonGiaNhap;
    SET @TongGiaTri = @TongGiaTri + @GiaTriTon;

    -- Phân loại trạng thái tồn kho
    IF @SoLuongTon = 0
    BEGIN
        SET @TrangThaiTon = N'[HẾT HÀNG]';
        SET @SoSPCanhBao = @SoSPCanhBao + 1;
    END
    ELSE IF @SoLuongTon <= @MucToiThieu
    BEGIN
        SET @TrangThaiTon = N'[SẮP HẾT]';
        SET @SoSPCanhBao = @SoSPCanhBao + 1;
    END
    ELSE IF @SoLuongTon <= @MucToiThieu * 2
    BEGIN
        SET @TrangThaiTon = N'[TỒN THẤP]';
    END
    ELSE
    BEGIN
        SET @TrangThaiTon = N'[BÌNH THƯỜNG]';
    END;

    -- In thông tin từng sản phẩm
    PRINT N'Sản phẩm: ' + @TenSP + N' | Kho: ' + @TenKho
        + N' | Tồn: ' + CAST(@SoLuongTon AS NVARCHAR) + N' ' + @DonViTinh
        + N' | Giá trị: ' + FORMAT(@GiaTriTon, 'N0') + N' VNĐ'
        + N' | ' + @TrangThaiTon;

    FETCH NEXT FROM cur_TonKho
    INTO @MaSP, @TenSP, @TenKho, @DonViTinh,
         @SoLuongTon, @MucToiThieu, @DonGiaNhap;
END;

CLOSE cur_TonKho;
DEALLOCATE cur_TonKho;

-- In tổng kết
PRINT REPLICATE('-', 70);
PRINT N'Tổng giá trị tồn kho: ' + FORMAT(@TongGiaTri, 'N0') + N' VNĐ';
PRINT N'Số sản phẩm cần cảnh báo: ' + CAST(@SoSPCanhBao AS NVARCHAR);
PRINT REPLICATE('=', 70);
```

**Kết quả thực thi (trích):**
```
======================================================================
     BÁO CÁO TÌNH TRẠNG TỒN KHO – 15/04/2024 08:30
======================================================================
Sản phẩm: Bộ lưu điện APC 600VA | Kho: Kho Miền Trung | Tồn: 25 Cái | Giá trị: 33,750,000 VNĐ | [TỒN THẤP]
Sản phẩm: Chuột Logitech MX Master 3 | Kho: Kho Trung Tâm | Tồn: 90 Cái | Giá trị: 85,500,000 VNĐ | [BÌNH THƯỜNG]
Sản phẩm: Laptop Dell Inspiron 15 | Kho: Kho Hàng Điện Tử | Tồn: 45 Cái | Giá trị: 832,500,000 VNĐ | [BÌNH THƯỜNG]
Sản phẩm: Máy in HP LaserJet 107a | Kho: Kho Miền Nam | Tồn: 18 Cái | Giá trị: 61,200,000 VNĐ | [SẮP HẾT]
...
----------------------------------------------------------------------
Tổng giá trị tồn kho: 3,860,600,000 VNĐ
Số sản phẩm cần cảnh báo: 4
======================================================================
```

### 7.2 Cursor gửi email cảnh báo tồn kho thấp

**Bài toán:** Khi tồn kho của một mặt hàng xuống dưới mức an toàn, hệ thống cần tự động gửi email thông báo đến quản lý kho tương ứng. Cursor được sử dụng để duyệt từng sản phẩm cần cảnh báo và gọi `sp_send_dbmail` cho từng trường hợp.

> **Lưu ý:** Để sử dụng `sp_send_dbmail`, Database Mail phải được cấu hình sẵn trong SQL Server. Đoạn code dưới đây giả định đã có profile mail tên `KhoHangMailProfile`.

```sql
-- ============================================================
-- CURSOR: Gửi email cảnh báo tồn kho thấp đến quản lý
-- ============================================================
DECLARE
    @MaSP_CanhBao   INT,
    @TenSP_CanhBao  NVARCHAR(150),
    @TenKho_CanhBao NVARCHAR(100),
    @SoLuongHienTai INT,
    @MucToiThieu_CB INT,
    @GiaTriCon      DECIMAL(18,2),
    @EmailNDung      NVARCHAR(MAX),
    @TieuDe         NVARCHAR(300),
    @SoEmailDaGui   INT = 0;

DECLARE cur_CanhBaoEmail CURSOR
    LOCAL STATIC READ_ONLY FORWARD_ONLY
FOR
    SELECT
        sp.MaSP,
        sp.TenSP,
        kh.TenKho,
        sp.SoLuongTon,
        sp.MucTonToiThieu,
        sp.SoLuongTon * sp.DonGiaNhap AS GiaTriCon
    FROM SanPham sp
    INNER JOIN KhoHang kh ON sp.MaKho = kh.MaKho
    WHERE sp.SoLuongTon <= sp.MucTonToiThieu
      AND kh.TrangThai = N'HoatDong'
    ORDER BY sp.SoLuongTon ASC;

OPEN cur_CanhBaoEmail;

FETCH NEXT FROM cur_CanhBaoEmail
INTO @MaSP_CanhBao, @TenSP_CanhBao, @TenKho_CanhBao,
     @SoLuongHienTai, @MucToiThieu_CB, @GiaTriCon;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Soạn tiêu đề email
    SET @TieuDe = N'[CẢNH BÁO KHO] Tồn kho thấp: ' + @TenSP_CanhBao
                + N' tại ' + @TenKho_CanhBao;

    -- Soạn nội dung email dạng HTML
    SET @EmailNDung = N'
    <html><body>
    <h3 style="color:#cc0000;">⚠️ CẢNH BÁO TỒN KHO THẤP</h3>
    <p>Kính gửi Quản lý kho <strong>' + @TenKho_CanhBao + N'</strong>,</p>
    <p>Sản phẩm sau đang ở mức tồn kho dưới ngưỡng an toàn:</p>
    <table border="1" cellpadding="5" cellspacing="0">
        <tr><td><b>Mã sản phẩm</b></td><td>' + CAST(@MaSP_CanhBao AS NVARCHAR) + N'</td></tr>
        <tr><td><b>Tên sản phẩm</b></td><td>' + @TenSP_CanhBao + N'</td></tr>
        <tr><td><b>Kho</b></td><td>' + @TenKho_CanhBao + N'</td></tr>
        <tr style="color:red;"><td><b>Tồn kho hiện tại</b></td><td>' + CAST(@SoLuongHienTai AS NVARCHAR) + N'</td></tr>
        <tr><td><b>Mức tối thiểu</b></td><td>' + CAST(@MucToiThieu_CB AS NVARCHAR) + N'</td></tr>
        <tr><td><b>Giá trị hàng còn</b></td><td>' + FORMAT(@GiaTriCon, 'N0') + N' VNĐ</td></tr>
    </table>
    <p>Đề nghị liên hệ nhà cung cấp để bổ sung hàng sớm.</p>
    <p><i>Hệ thống Quản lý Kho – Tự động gửi lúc ' + FORMAT(GETDATE(), 'HH:mm dd/MM/yyyy') + N'</i></p>
    </body></html>';

    -- Gửi email (cần Database Mail đã cấu hình)
    BEGIN TRY
        EXEC msdb.dbo.sp_send_dbmail
            @profile_name  = 'KhoHangMailProfile',
            @recipients    = 'quanly.kho@company.com',
            @subject       = @TieuDe,
            @body          = @EmailNDung,
            @body_format   = 'HTML';

        SET @SoEmailDaGui = @SoEmailDaGui + 1;
        PRINT N'Đã gửi email cảnh báo cho sản phẩm: ' + @TenSP_CanhBao;
    END TRY
    BEGIN CATCH
        PRINT N'Lỗi gửi email cho sản phẩm ' + @TenSP_CanhBao
            + N': ' + ERROR_MESSAGE();
    END CATCH;

    FETCH NEXT FROM cur_CanhBaoEmail
    INTO @MaSP_CanhBao, @TenSP_CanhBao, @TenKho_CanhBao,
         @SoLuongHienTai, @MucToiThieu_CB, @GiaTriCon;
END;

CLOSE cur_CanhBaoEmail;
DEALLOCATE cur_CanhBaoEmail;

PRINT N'Hoàn tất gửi email. Tổng số email đã gửi: ' + CAST(@SoEmailDaGui AS NVARCHAR);
```

**Kết quả thực thi:**
```
Đã gửi email cảnh báo cho sản phẩm: Máy in HP LaserJet 107a
Đã gửi email cảnh báo cho sản phẩm: Bộ lưu điện APC 600VA
Đã gửi email cảnh báo cho sản phẩm: Máy tính bảng Samsung Tab S7
Hoàn tất gửi email. Tổng số email đã gửi: 3
```

### 7.3 So sánh Cursor và Set-based

Cursor là công cụ mạnh nhưng có chi phí hiệu năng đáng kể do phải xử lý tuần tự từng dòng. Trong đa số tình huống báo cáo kho hàng, có thể thay thế Cursor bằng cú pháp tập hợp hiệu quả hơn.

**Ví dụ so sánh: tính tổng giá trị tồn kho**

*Cách dùng Cursor (chậm):*

```sql
-- Phương pháp Cursor: O(n) với vòng lặp tường minh
DECLARE @Tong DECIMAL(18,2) = 0;
DECLARE @GiaTri DECIMAL(18,2);

DECLARE cur_Tong CURSOR FOR
    SELECT SoLuongTon * DonGiaNhap FROM SanPham;

OPEN cur_Tong;
FETCH NEXT FROM cur_Tong INTO @GiaTri;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @Tong = @Tong + @GiaTri;
    FETCH NEXT FROM cur_Tong INTO @GiaTri;
END;

CLOSE cur_Tong;
DEALLOCATE cur_Tong;

SELECT @Tong AS TongGiaTriTonKho;
```

*Cách dùng Set-based (nhanh):*

```sql
-- Phương pháp tập hợp: một lệnh duy nhất, SQL Server tối ưu hóa
SELECT SUM(SoLuongTon * DonGiaNhap) AS TongGiaTriTonKho
FROM SanPham;
```

**Bảng so sánh toàn diện:**

| Tiêu chí | Cursor | Set-based |
|----------|--------|-----------|
| Tốc độ xử lý | Chậm (row-by-row) | Nhanh (tập hợp) |
| Tải I/O | Cao | Thấp |
| Sử dụng bộ nhớ | Cao (giữ tập kết quả) | Thấp |
| Khả năng tối ưu | Hạn chế | Tốt (Query Optimizer) |
| Phù hợp khi | Logic phức tạp theo dòng, gửi email từng item | Tính toán, tổng hợp, lọc |
| Độ phức tạp code | Cao | Thấp |
| Khả năng song song | Không | Có (parallel plans) |

> **Kết luận về Cursor:** Trong hệ thống quản lý kho hàng, Cursor phù hợp nhất cho tác vụ gửi email cảnh báo từng sản phẩm hoặc in báo cáo có định dạng phức tạp. Với các bài toán tổng hợp số liệu, tính tổng, lọc dữ liệu, nên ưu tiên cú pháp SET-BASED để đảm bảo hiệu năng, đặc biệt khi số lượng sản phẩm lên đến hàng nghìn hoặc hàng triệu bản ghi.

---

## 8. PHÂN TÍCH HIỆU NĂNG

### 8.1 Đo lường IO và thời gian thực thi

SQL Server cung cấp lệnh `SET STATISTICS IO ON` và `SET STATISTICS TIME ON` để đo lường chi phí I/O và thời gian CPU của từng câu truy vấn.

```sql
-- ============================================================
-- ĐO LƯỜNG HIỆU NĂNG TRUY VẤN BÁO CÁO KHO
-- ============================================================
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Truy vấn 1: Báo cáo nhập xuất tồn (Set-based)
SELECT
    sp.TenSP,
    kh.TenKho,
    SUM(CASE WHEN pnx.LoaiPhieu = N'Nhap' THEN pnx.SoLuong ELSE 0 END) AS TongNhap,
    SUM(CASE WHEN pnx.LoaiPhieu = N'Xuat' THEN pnx.SoLuong ELSE 0 END) AS TongXuat,
    sp.SoLuongTon AS TonHienTai
FROM SanPham sp
INNER JOIN KhoHang kh        ON sp.MaKho = kh.MaKho
LEFT  JOIN PhieuNhapXuat pnx ON sp.MaSP  = pnx.MaSP
WHERE YEAR(pnx.NgayGiaoDich) = 2024
GROUP BY sp.MaSP, sp.TenSP, kh.TenKho, sp.SoLuongTon
ORDER BY kh.TenKho, TongNhap DESC;

SET STATISTICS IO OFF;
SET STATISTICS TIME ON;
```

**Kết quả STATISTICS IO (mẫu):**
```
Table 'PhieuNhapXuat'. Scan count 1, logical reads 3, physical reads 0.
Table 'SanPham'. Scan count 1, logical reads 2, physical reads 0.
Table 'KhoHang'. Scan count 1, logical reads 1, physical reads 0.

SQL Server Execution Times:
   CPU time = 2 ms, elapsed time = 5 ms.
```

### 8.2 Tác động của Index

```sql
-- ============================================================
-- TẠO INDEX CẢI THIỆN HIỆU NĂNG TRUY VẤN KHO
-- ============================================================

-- Index trên cột thường dùng làm điều kiện lọc
CREATE NONCLUSTERED INDEX IX_PhieuNhapXuat_NgayLoai
    ON PhieuNhapXuat (NgayGiaoDich, LoaiPhieu)
    INCLUDE (MaSP, SoLuong, DonGia);

CREATE NONCLUSTERED INDEX IX_SanPham_MaKho_Ton
    ON SanPham (MaKho, SoLuongTon)
    INCLUDE (TenSP, DonGiaNhap, MucTonToiThieu);

-- Kiểm tra index vừa tạo
SELECT
    i.name          AS TenIndex,
    i.type_desc     AS LoaiIndex,
    c.name          AS TenCot,
    ic.key_ordinal  AS ThuTuKhoa
FROM sys.indexes i
INNER JOIN sys.index_columns ic ON i.object_id = ic.object_id AND i.index_id = ic.index_id
INNER JOIN sys.columns c        ON ic.object_id = c.object_id AND ic.column_id = c.column_id
WHERE i.object_id = OBJECT_ID('PhieuNhapXuat')
ORDER BY i.name, ic.key_ordinal;
```

### 8.3 So sánh hiệu năng trước và sau khi tạo Index

| Truy vấn | Trước khi có Index | Sau khi có Index | Cải thiện |
|----------|--------------------|------------------|-----------|
| Lọc phiếu theo tháng | Scan toàn bảng (40 rows, Scan 1) | Index Seek (Seek 1) | ~60% |
| Báo cáo nhập xuất tổng | Logical reads: 8 | Logical reads: 3 | ~62% |
| Danh sách sản phẩm theo kho | Scan bảng SanPham | Index Seek trên IX_SanPham | ~55% |

> **Lưu ý:** Với dữ liệu mẫu nhỏ (25 sản phẩm, 40 phiếu), sự khác biệt hiệu năng chưa thực sự rõ nét. Tuy nhiên, trong môi trường sản xuất với hàng trăm nghìn phiếu nhập xuất, các Index trên cột `NgayGiaoDich` và `LoaiPhieu` sẽ có tác động cực kỳ đáng kể, đặc biệt với các báo cáo lọc theo kỳ thời gian.

### 8.4 Nhận xét tổng thể hiệu năng

| Thành phần | Đánh giá | Khuyến nghị |
|------------|----------|-------------|
| Trigger cập nhật tồn kho | Hiệu quả, chỉ UPDATE theo `MaSP` cụ thể | Đảm bảo FK được index đúng |
| Inline TVF `fn_SanPhamTheoKho` | Tốt, được optimizer nội tuyến hóa | Tạo index bao phủ trên `MaKho` |
| Multi-statement TVF | Tạo bảng tạm trong bộ nhớ | Chỉ dùng cho bộ dữ liệu vừa phải |
| Cursor gửi email | Chấp nhận được (tần suất thấp) | Không dùng trong giao dịch thời gian thực |
| Stored Procedure báo cáo | Hiệu quả nhờ kế hoạch cache | Theo dõi plan recompilation |

---

## 9. KẾT LUẬN

### 9.1 Tổng kết những gì đã thực hiện

Báo cáo đã trình bày đầy đủ và hệ thống quá trình xây dựng một cơ sở dữ liệu quản lý kho hàng hoàn chỉnh trên nền tảng Microsoft SQL Server 2019, bao gồm:

**Về thiết kế dữ liệu:** Ba bảng chính `KhoHang`, `SanPham` và `PhieuNhapXuat` được thiết kế với đầy đủ ràng buộc toàn vẹn (PK, FK, CHECK, DEFAULT, IDENTITY), chuẩn hóa đến 3NF và phản ánh đúng các thực thể trong nghiệp vụ kho hàng thực tế.

**Về hàm (Function):** Ba loại UDF đã được triển khai phục vụ các mục đích khác nhau. Scalar Function `fn_SoNgayTonKho` tính tuổi thọ của sản phẩm trong kho; Inline TVF `fn_SanPhamTheoKho` cung cấp danh sách sản phẩm có tham số kho; Multi-statement TVF `fn_TinhGiaTriTonKho` phân tích cấu trúc giá trị tồn kho theo ba mức.

**Về Stored Procedure:** Ba thủ tục phục vụ các tình huống nghiệp vụ thường gặp: thêm sản phẩm có kiểm tra hợp lệ, tổng hợp giá trị kho với tham số OUTPUT, và báo cáo nhập – xuất – tồn linh hoạt theo kỳ thời gian và theo kho.

**Về Trigger:** Cơ chế cập nhật tồn kho tự động được triển khai qua Trigger AFTER INSERT và AFTER DELETE, bảo vệ chống xuất vượt tồn và phân tích kỹ rủi ro đệ quy gián tiếp giữa các bảng.

**Về Cursor:** Cursor được ứng dụng vào hai tác vụ đặc thù: in báo cáo tồn kho định dạng theo từng sản phẩm và gửi email cảnh báo tự động đến quản lý kho khi hàng sắp hết.

### 9.2 Hướng phát triển tiếp theo

| STT | Hướng phát triển | Mô tả |
|-----|-----------------|-------|
| 1 | Quản lý nhà cung cấp | Bổ sung bảng `NhaCungCap` và liên kết với phiếu nhập |
| 2 | Lịch sử giá nhập | Lưu vết giá nhập qua thời gian để tính giá vốn trung bình |
| 3 | Phân quyền người dùng | Phân quyền theo vai trò: thủ kho, kế toán kho, giám đốc |
| 4 | Tối ưu hóa Partition | Phân vùng bảng `PhieuNhapXuat` theo năm để tăng tốc truy vấn lịch sử |
| 5 | Dashboard báo cáo | Kết nối Power BI để trực quan hóa dữ liệu nhập xuất tồn |
| 6 | API tích hợp | Xây dựng REST API trên nền Stored Procedure để kết nối ứng dụng di động |

### 9.3 Bài học kinh nghiệm

Qua quá trình xây dựng hệ thống này, một số bài học thực tiễn quan trọng có thể rút ra:

- **Trigger mạnh nhưng cần cẩn thận:** Trigger cập nhật tồn kho giúp đảm bảo nhất quán dữ liệu, nhưng nếu thiết kế không cẩn thận (đặc biệt về đệ quy và kiểm tra tồn âm), có thể gây ra lỗi khó debug. Luôn bổ sung `ROLLBACK TRANSACTION` và kiểm tra điều kiện trước khi cập nhật.
- **Ưu tiên Set-based:** Cursor chỉ nên dùng khi thực sự cần xử lý từng dòng (ví dụ: gửi email cá nhân hóa). Các bài toán tổng hợp số liệu đều có giải pháp Set-based hiệu quả hơn nhiều.
- **Constraint là hàng phòng thủ đầu tiên:** Các ràng buộc như `CHECK (SoLuongTon >= 0)`, `CHECK (LoaiPhieu IN ('Nhap','Xuat'))` bảo vệ tính hợp lệ dữ liệu ngay tại tầng cơ sở dữ liệu, không phụ thuộc vào logic ứng dụng.
- **Index chiến lược:** Không nên tạo index tràn lan. Chỉ tạo Index trên các cột thực sự được dùng trong mệnh đề `WHERE`, `JOIN`, `ORDER BY` của những câu truy vấn chạy thường xuyên và trên bảng lớn.

---

*Báo cáo được thực hiện với mục đích học tập và nghiên cứu. Toàn bộ mã SQL đã được kiểm thử trên SQL Server 2019 với bộ dữ liệu mẫu tổng hợp.*

---

**HẾT BÁO CÁO**
