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
│   KhoHang   │ 1──────N│  SanPham    │ 1──────N│  PhieuNhapXuat   │
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
Các loại Built-in Function trong SQL Server

Trong SQL Server có rất nhiều **Built-in Function (hàm có sẵn)** giúp xử lý dữ liệu nhanh chóng mà không cần tự viết lại logic.  
Các hàm này được chia thành các nhóm chính như sau:

---

## 🔹 1. Hàm xử lý chuỗi (String Functions)

**Chức năng:** Xử lý dữ liệu dạng chuỗi ký tự  


## 🔹 2. Hàm số học (Mathematical Functions)

**Chức năng:** Thực hiện các phép toán số  


## 🔹 3. Hàm ngày giờ (Date and Time Functions)

**Chức năng:** Xử lý dữ liệu ngày tháng  

## 🔹 4. Hàm chuyển đổi kiểu (Conversion Functions)

**Chức năng:** Chuyển đổi giữa các kiểu dữ liệu  

## 🔹 5. Hàm tổng hợp (Aggregate Functions)

**Chức năng:** Tính toán trên nhiều dòng dữ liệu  


## 🔹 6. Hàm logic (Logical Functions)

**Chức năng:** Xử lý điều kiện  

## 🔹 7. Hàm hệ thống (System Functions)

**Chức năng:** Trả về thông tin hệ thống

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

# Phân tích logic xử lý Function `fn_SoNgayTonKho`

Function `fn_SoNgayTonKho` được sử dụng để tính số ngày một sản phẩm đã tồn tại trong kho dựa trên ngày nhập đầu tiên của sản phẩm.

Đầu tiên, function nhận vào mã sản phẩm `@MaSP`. Sau đó hệ thống truy xuất bảng `SanPham` để lấy giá trị `NgayNhapDau` tương ứng với sản phẩm cần kiểm tra.

Sau khi lấy dữ liệu, hệ thống kiểm tra sản phẩm có tồn tại hay không. Nếu không tìm thấy sản phẩm hoặc chưa có ngày nhập đầu, function sẽ trả về `NULL` để tránh phát sinh lỗi tính toán.

Nếu dữ liệu hợp lệ, hệ thống sử dụng hàm `DATEDIFF` để tính khoảng thời gian từ ngày nhập đầu đến thời điểm hiện tại. Kết quả nhận được chính là số ngày sản phẩm đã lưu trong kho.

Cuối cùng, function trả về giá trị số ngày tồn kho dưới dạng kiểu dữ liệu `INT`.

Function này hỗ trợ quản lý hàng tồn kho, theo dõi sản phẩm lưu kho lâu ngày và phục vụ cho các báo cáo thống kê trong hệ thống quản lý kho hàng.



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


Phân tích logic

Function fn_SanPhamTheoKho được sử dụng để lấy danh sách sản phẩm thuộc một kho cụ thể và đồng thời phân tích tình trạng tồn kho của từng sản phẩm.

Đầu tiên, function nhận vào mã kho @MaKho. Hệ thống sử dụng giá trị này để tìm các sản phẩm thuộc kho tương ứng trong bảng SanPham.

Sau đó, dữ liệu sản phẩm được kết hợp với bảng KhoHang thông qua khóa MaKho nhằm lấy thêm thông tin về kho như tên kho và địa chỉ kho lưu trữ.

Trong quá trình truy vấn, hệ thống thực hiện tính toán giá trị tồn kho của từng sản phẩm bằng cách nhân số lượng tồn với đơn giá nhập. Giá trị này giúp đánh giá tổng giá trị hàng hóa hiện còn trong kho.

Tiếp theo, function phân loại trạng thái tồn kho dựa trên số lượng hiện có và mức tồn tối thiểu của sản phẩm. Nếu số lượng bằng 0 thì sản phẩm được xác định là hết hàng. Nếu số lượng nhỏ hơn hoặc bằng mức tồn tối thiểu thì trạng thái là sắp hết. Trường hợp số lượng chỉ cao hơn mức tối thiểu một khoảng nhỏ thì được phân loại là tồn thấp. Các sản phẩm còn lại được xem là ở trạng thái bình thường.

Cuối cùng, function trả về danh sách sản phẩm cùng thông tin tồn kho, giá trị tồn và trạng thái hàng hóa để phục vụ cho quản lý kho, kiểm soát nhập xuất và hỗ trợ báo cáo thống kê.


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

# Phân tích logic xử lý Function `fn_TinhGiaTriTonKho`

Function `fn_TinhGiaTriTonKho` được sử dụng để tính toán và phân tích giá trị tồn kho của các sản phẩm trong một kho cụ thể.

Đầu tiên, function nhận vào mã kho `@MaKho`. Dựa trên mã kho này, hệ thống truy xuất bảng `SanPham` để lấy danh sách các sản phẩm thuộc kho tương ứng.

Tiếp theo, hệ thống tính tổng giá trị tồn kho bằng cách cộng toàn bộ giá trị của các sản phẩm trong kho. Giá trị của từng sản phẩm được xác định bằng công thức:

* số lượng tồn × đơn giá nhập

Tổng giá trị kho được sử dụng để phục vụ việc tính tỷ lệ giá trị của từng sản phẩm trong toàn bộ kho.

Sau đó, hệ thống kiểm tra nếu tổng giá trị kho bằng 0 thì sẽ gán giá trị mặc định bằng 1 nhằm tránh lỗi chia cho 0 trong quá trình tính toán tỷ lệ.

Tiếp theo, function thực hiện xử lý cho từng sản phẩm trong kho. Với mỗi sản phẩm, hệ thống tính:

* giá trị tồn kho
* giá trị tồn kho có VAT 10%
* tỷ lệ giá trị sản phẩm trong tổng giá trị kho

Ngoài ra, hệ thống còn phân loại mức giá trị của sản phẩm dựa trên tổng giá trị tồn kho. Nếu giá trị sản phẩm rất lớn thì được xếp vào nhóm giá trị cao, các sản phẩm có giá trị trung bình sẽ thuộc nhóm trung bình, còn lại là nhóm giá trị thấp.

Sau khi hoàn tất tính toán, toàn bộ dữ liệu được ghi vào bảng kết quả `@KetQua`.

Cuối cùng, function trả về danh sách sản phẩm cùng các thông tin phân tích như:

* số lượng tồn
* giá trị tồn kho
* giá trị có VAT
* mức giá trị sản phẩm
* tỷ lệ giá trị trong kho

Function này hỗ trợ quản lý kho hàng, đánh giá giá trị hàng hóa, phân tích cơ cấu tồn kho và phục vụ cho các báo cáo thống kê tài chính trong hệ thống quản lý kho.


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
Trong SQL Server có rất nhiều **Stored Procedure (SP) có sẵn** gọi là **System Stored Procedure**, được chia thành một số nhóm chính như sau:

## 🔹 Các loại System Stored Procedure

- **Metadata SP** (truy vấn thông tin hệ thống):  
  Ví dụ: `sp_help`, `sp_columns`, `sp_tables`  

- **Security SP** (quản lý bảo mật, quyền):  
  Ví dụ: `sp_addlogin`

- **Database Management SP** (quản lý cơ sở dữ liệu):  
  Ví dụ: `sp_rename`  

- **Execution SP** (thực thi lệnh động):  
  Ví dụ: `sp_executesql`  

- **System Monitoring SP** (theo dõi hệ thống):  
  
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

# Phân tích logic xử lý Procedure `sp_ThemSanPham`

Procedure `sp_ThemSanPham` được sử dụng để thêm mới sản phẩm vào hệ thống quản lý kho hàng và đồng thời kiểm tra tính hợp lệ của dữ liệu trước khi lưu vào cơ sở dữ liệu.

Đầu tiên, procedure nhận các thông tin của sản phẩm như tên sản phẩm, đơn vị tính, số lượng tồn, đơn giá nhập, mã kho và mức tồn tối thiểu. Ngoài ra, procedure còn sử dụng biến output `@MaSP_Moi` để trả về mã sản phẩm vừa được tạo.

Sau đó, hệ thống kiểm tra kho hàng có tồn tại và đang ở trạng thái hoạt động hay không. Nếu kho không tồn tại hoặc đã ngừng hoạt động thì procedure sẽ báo lỗi và dừng xử lý nhằm tránh thêm sản phẩm vào kho không hợp lệ.

Tiếp theo, hệ thống kiểm tra tên sản phẩm có bị bỏ trống hay không. Nếu tên sản phẩm không hợp lệ, procedure sẽ trả về thông báo lỗi để đảm bảo dữ liệu nhập vào đầy đủ.

Sau bước kiểm tra dữ liệu cơ bản, hệ thống tiếp tục kiểm tra sản phẩm có bị trùng tên trong cùng một kho hay không. Điều này giúp tránh việc lưu nhiều sản phẩm giống nhau trong cùng kho hàng và hỗ trợ quản lý dữ liệu chính xác hơn.

Nếu toàn bộ dữ liệu đều hợp lệ, procedure thực hiện thêm sản phẩm mới vào bảng `SanPham`. Sau khi thêm thành công, hệ thống lấy mã sản phẩm vừa tạo bằng `SCOPE_IDENTITY()` và gán cho biến output để phục vụ các thao tác tiếp theo.

Toàn bộ quá trình thêm dữ liệu được đặt trong khối `TRY...CATCH` nhằm xử lý lỗi hệ thống. Nếu phát sinh lỗi trong quá trình lưu dữ liệu, procedure sẽ trả về thông báo lỗi và mã trạng thái tương ứng.

Procedure này hỗ trợ kiểm soát dữ liệu đầu vào, đảm bảo tính toàn vẹn dữ liệu và tăng độ an toàn cho quá trình quản lý sản phẩm trong hệ thống kho hàng.


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


# Phân tích logic xử lý Procedure `sp_TongGiaTriKho`

Procedure `sp_TongGiaTriKho` được sử dụng để tính toán tổng giá trị tồn kho trong hệ thống quản lý kho hàng. Procedure hỗ trợ tính cho toàn bộ kho hoặc cho một kho cụ thể thông qua tham số đầu vào `@MaKho`.

Đầu tiên, procedure nhận tham số mã kho `@MaKho`. Nếu tham số này bằng `NULL`, hệ thống sẽ thực hiện thống kê cho toàn bộ kho hàng trong hệ thống. Ngược lại, nếu có mã kho cụ thể, hệ thống chỉ tính toán cho kho được chỉ định.

Sau đó, hệ thống tiến hành tính:

* tổng giá trị tồn kho
* tổng số sản phẩm
* tổng số lượng hàng tồn

Giá trị tồn kho được xác định bằng cách lấy số lượng tồn nhân với đơn giá nhập của từng sản phẩm.

Trong trường hợp tính cho toàn bộ kho, procedure còn thực hiện tạo báo cáo chi tiết theo từng kho hàng. Hệ thống kết hợp dữ liệu giữa bảng `SanPham` và `KhoHang` để lấy thông tin tên kho cùng các số liệu thống kê tương ứng.

Ngoài giá trị tồn kho thông thường, procedure còn tính thêm giá trị tồn kho có VAT 10% nhằm phục vụ cho việc phân tích tài chính và báo cáo quản lý.

Kết quả thống kê tổng hợp sẽ được lưu vào các biến output gồm:

* `@TongGiaTri`: tổng giá trị tồn kho
* `@SoSanPham`: tổng số sản phẩm
* `@TongSoLuong`: tổng số lượng hàng tồn

Procedure này hỗ trợ theo dõi giá trị hàng hóa trong kho, phục vụ thống kê tồn kho, đánh giá tài sản lưu kho và hỗ trợ ra quyết định quản lý trong hệ thống quản lý kho hàng.



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

# Bài toán và phân tích logic xử lý Procedure `sp_BaoCaoNhapXuat`

# 1. Đặt bài toán

Trong hệ thống quản lý kho hàng, doanh nghiệp cần theo dõi tình hình nhập kho, xuất kho và số lượng tồn hiện tại của từng sản phẩm nhằm:

* kiểm soát lượng hàng hóa trong kho
* đánh giá tình trạng tồn kho
* phát hiện sản phẩm sắp hết hoặc hết hàng
* hỗ trợ lập kế hoạch nhập hàng
* phục vụ báo cáo quản trị và thống kê tài chính

Tuy nhiên, dữ liệu nhập xuất thường phát sinh liên tục và số lượng lớn, gây khó khăn cho việc tổng hợp thủ công. Vì vậy cần xây dựng một procedure giúp tự động thống kê nhập – xuất – tồn theo khoảng thời gian và theo từng kho hàng.

---

# 2. Phân tích logic xử lý

Procedure `sp_BaoCaoNhapXuat` được sử dụng để tạo báo cáo tổng hợp nhập kho, xuất kho và tồn kho của sản phẩm trong hệ thống.

Đầu tiên, procedure nhận vào:

* ngày bắt đầu `@TuNgay`
* ngày kết thúc `@DenNgay`
* mã kho `@MaKho`

Nếu người dùng không truyền khoảng thời gian, hệ thống sẽ tự động lấy dữ liệu trong vòng một tháng gần nhất.

Sau đó, hệ thống truy xuất danh sách sản phẩm từ bảng `SanPham` và kết hợp với bảng `KhoHang` để lấy thông tin kho lưu trữ.

Tiếp theo, procedure thực hiện tổng hợp dữ liệu nhập kho trong khoảng thời gian được chọn. Hệ thống tính:

* tổng số lượng nhập
* tổng giá trị nhập

Dữ liệu này được lấy từ bảng `PhieuNhapXuat` với loại phiếu là `Nhập`.

Tương tự, hệ thống tiếp tục tổng hợp dữ liệu xuất kho bằng cách tính:

* tổng số lượng xuất
* tổng giá trị xuất

với các giao dịch có loại phiếu là `Xuất`.

Sau khi có dữ liệu nhập và xuất, procedure tiến hành tính tồn kho hiện tại của từng sản phẩm dựa trên số lượng tồn đang lưu trong bảng `SanPham`.

Ngoài ra, hệ thống còn đánh giá trạng thái tồn kho của sản phẩm. Nếu số lượng tồn bằng 0 thì sản phẩm được xác định là hết hàng và cần nhập thêm. Nếu số lượng tồn nhỏ hơn hoặc bằng mức tồn tối thiểu thì hệ thống cảnh báo tồn kho dưới mức an toàn. Các sản phẩm còn lại được xem là tồn kho bình thường.

Cuối cùng, procedure trả về báo cáo tổng hợp gồm:

* thông tin sản phẩm
* thông tin kho hàng
* số lượng nhập
* số lượng xuất
* giá trị nhập xuất
* tồn kho hiện tại
* giá trị tồn kho
* nhận xét tình trạng tồn kho

Procedure này hỗ trợ quản lý hàng hóa hiệu quả, theo dõi luồng nhập xuất, kiểm soát tồn kho và phục vụ cho các báo cáo quản trị trong hệ thống quản lý kho hàng.

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

# Bài toán và phân tích logic xử lý Trigger `trg_PhieuNhapXuat_Insert`

# 1. Đặt bài toán

Trong hệ thống quản lý kho hàng, số lượng tồn kho của sản phẩm phải luôn được cập nhật chính xác sau mỗi giao dịch nhập hoặc xuất hàng.

Nếu việc cập nhật tồn kho được thực hiện thủ công sẽ dễ xảy ra:

* sai lệch số lượng tồn
* xuất hàng vượt quá số lượng thực tế
* dữ liệu kho không đồng bộ
* khó kiểm soát tình trạng tồn kho thấp

Vì vậy cần xây dựng trigger tự động cập nhật số lượng tồn kho ngay khi phát sinh phiếu nhập hoặc phiếu xuất nhằm đảm bảo tính chính xác và toàn vẹn dữ liệu trong hệ thống.

---

# 2. Phân tích logic xử lý

Trigger `trg_PhieuNhapXuat_Insert` được kích hoạt sau khi có dữ liệu mới được thêm vào bảng `PhieuNhapXuat`.

Đầu tiên, trigger kiểm tra loại phiếu giao dịch vừa được thêm.

Nếu giao dịch là phiếu nhập, hệ thống sẽ tự động cộng thêm số lượng nhập vào tồn kho hiện tại của sản phẩm tương ứng trong bảng `SanPham`.

Ngược lại, nếu giao dịch là phiếu xuất, hệ thống sẽ kiểm tra số lượng tồn kho hiện tại trước khi thực hiện xuất hàng.

Nếu số lượng tồn nhỏ hơn số lượng cần xuất, trigger sẽ:

* phát sinh thông báo lỗi
* hủy giao dịch hiện tại
* ngăn không cho dữ liệu xuất kho được lưu vào hệ thống

Điều này giúp tránh tình trạng tồn kho âm và đảm bảo dữ liệu kho luôn chính xác.

Nếu số lượng tồn kho hợp lệ, hệ thống sẽ thực hiện giảm số lượng tồn tương ứng với lượng hàng đã xuất.

Sau khi cập nhật tồn kho, trigger tiếp tục kiểm tra mức tồn kho an toàn của sản phẩm. Nếu số lượng tồn hiện tại nhỏ hơn hoặc bằng mức tồn tối thiểu đã quy định, hệ thống sẽ sinh cảnh báo để người quản lý có kế hoạch nhập thêm hàng.

Cuối cùng, trigger hoàn tất quá trình cập nhật dữ liệu tồn kho tự động sau giao dịch nhập xuất.

Trigger này giúp:

* tự động đồng bộ tồn kho
* đảm bảo tính toàn vẹn dữ liệu
* ngăn xuất vượt số lượng tồn
* hỗ trợ cảnh báo tồn kho thấp
* tăng độ chính xác trong quản lý kho hàng

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

```sql
-- Kiểm tra trước khi thêm phiếu
SELECT MaSP, TenSP, SoLuongTon FROM SanPham WHERE MaSP = 5;

-- Thêm phiếu xuất 30 chuột Logitech
INSERT INTO PhieuNhapXuat (MaSP, NgayGiaoDich, LoaiPhieu, SoLuong, DonGia, GhiChu)
VALUES (5, GETDATE(), N'Xuat', 30, 1100000, N'Test trigger');

-- Kiểm tra sau khi thêm phiếu
SELECT MaSP, TenSP, SoLuongTon FROM SanPham WHERE MaSP = 5;
```

**Kết quả thực thi:**
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/70c2f0e3-c5f0-43a7-8ebb-5397dab23b3c" />
Tạo trigger và chuẩn bị dữ liệu

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a11bdc26-8f29-42c9-a037-0ea7eaefc878" />
Kiểm tra trigger sau khi thực thi

#### 6.1.2 Trigger sau khi DELETE phiếu

Phân tích logic xử lý Trigger trg_PhieuNhapXuat_Delete

Trigger trg_PhieuNhapXuat_Delete được kích hoạt sau khi dữ liệu bị xóa khỏi bảng PhieuNhapXuat.

Trigger này có nhiệm vụ hoàn trả lại số lượng tồn kho tương ứng với giao dịch đã bị xóa nhằm đảm bảo dữ liệu tồn kho luôn chính xác.
Đầu tiên, hệ thống kiểm tra các phiếu giao dịch vừa bị xóa thông qua bảng tạm deleted.


Nếu phiếu bị xóa là phiếu nhập, hệ thống sẽ giảm lại số lượng tồn kho của sản phẩm tương ứng vì trước đó số lượng này đã được cộng thêm khi nhập hàng.

Ngược lại, nếu phiếu bị xóa là phiếu xuất, hệ thống sẽ cộng lại số lượng tồn kho do lượng hàng này trước đó đã bị trừ khỏi kho khi thực hiện xuất hàng.

Quá trình cập nhật tồn kho được thực hiện tự động thông qua việc liên kết bảng SanPham với dữ liệu trong bảng deleted dựa trên mã sản phẩm.

Sau khi hoàn tất cập nhật, hệ thống hiển thị thông báo xác nhận rằng tồn kho đã được hoàn tác tương ứng với giao dịch bị xóa.

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
<img width="1913" height="1079" alt="image" src="https://github.com/user-attachments/assets/2ac464db-cd54-44fe-b7bf-f4b2c73e3d06" />
Tạo trigger xóa

### 6.2 Trigger đệ quy và phân tích rủi ro

#### 6.2.1 Khái niệm Trigger đệ quy

Trigger đệ quy xảy ra khi một trigger thực thi một câu lệnh DML (INSERT / UPDATE / DELETE) mà chính câu lệnh đó lại kích hoạt cùng trigger đó một lần nữa, tạo thành vòng lặp không có điểm dừng tự nhiên.

Trong hệ thống kho hàng, tình huống này có thể xảy ra nếu một trigger trên bảng `SanPham` lại ghi thêm một bản ghi vào `PhieuNhapXuat`, và trigger trên `PhieuNhapXuat` lại cập nhật `SanPham`, gây ra vòng lặp chéo (indirect recursion).

#### 6.2.2 Ví dụ trigger gây đệ quy gián tiếp (mô phỏng)

# Phân tích logic xử lý Trigger đệ quy gián tiếp

Đoạn mã được sử dụng để mô phỏng hiện tượng trigger đệ quy gián tiếp trong hệ quản trị cơ sở dữ liệu. Trường hợp này xảy ra khi một trigger trên bảng này cập nhật dữ liệu của bảng khác, sau đó trigger của bảng thứ hai lại cập nhật ngược trở lại bảng ban đầu, tạo thành vòng lặp kích hoạt liên tục.

Đầu tiên, hệ thống bật chế độ `RECURSIVE_TRIGGERS` để cho phép các trigger có thể kích hoạt lẫn nhau nhiều lần trong cùng một transaction.

Sau đó, hệ thống kiểm tra và xóa các trigger cũ nếu đã tồn tại nhằm tránh xung đột khi tạo lại trigger mới.

Tiếp theo, trigger `trg_SanPham_UpdateKho` được tạo trên bảng `SanPham`. Trigger này sẽ tự động kích hoạt sau khi dữ liệu của bảng `SanPham` bị cập nhật. Khi chạy, trigger thực hiện cập nhật bảng `KhoHang` bằng cách tăng giá trị `SucChua`.

Sau đó, hệ thống tạo trigger `trg_KhoHang_UpdateSP` trên bảng `KhoHang`. Trigger này cũng được kích hoạt sau khi bảng `KhoHang` bị cập nhật. Khi thực thi, trigger tiếp tục cập nhật lại bảng `SanPham` bằng cách tăng `SoLuongTon`.

Khi thực hiện câu lệnh cập nhật trên bảng `SanPham`, trigger đầu tiên sẽ chạy và cập nhật bảng `KhoHang`. Việc cập nhật `KhoHang` lại làm trigger thứ hai được kích hoạt. Trigger thứ hai tiếp tục cập nhật bảng `SanPham`, khiến trigger đầu tiên chạy lại.

Quá trình này lặp đi lặp lại liên tục theo chuỗi:

SanPham → KhoHang → SanPham → KhoHang → ...

Do không có điều kiện dừng nên hệ thống tạo ra vòng lặp trigger vô hạn. Khi số mức lồng nhau vượt quá giới hạn cho phép của SQL Server, hệ thống sẽ phát sinh lỗi vượt quá mức nesting level và hủy transaction hiện tại.

Đây là ví dụ điển hình của:

- trigger đệ quy gián tiếp
- nested trigger recursion
- vòng lặp trigger giữa nhiều bảng

Tình huống này cho thấy cần kiểm soát chặt chẽ logic cập nhật dữ liệu trong trigger để tránh gây lỗi hệ thống, giảm hiệu năng và làm mất tính ổn định của cơ sở dữ liệu.



```sql
-- ============================================================
-- MÔ PHỎNG TRIGGER ĐỆ QUY GIÁN TIẾP
-- SanPham → KhoHang → SanPham
-- ============================================================

-- Bật recursive trigger
ALTER DATABASE CURRENT
SET RECURSIVE_TRIGGERS ON;
GO


-- ============================================================
-- XÓA TRIGGER CŨ NẾU ĐÃ TỒN TẠI
-- ============================================================

IF OBJECT_ID('trg_SanPham_UpdateKho', 'TR') IS NOT NULL
    DROP TRIGGER trg_SanPham_UpdateKho;
GO

IF OBJECT_ID('trg_KhoHang_UpdateSP', 'TR') IS NOT NULL
    DROP TRIGGER trg_KhoHang_UpdateSP;
GO


-- ============================================================
-- TRIGGER 1
-- Khi UPDATE SanPham → UPDATE KhoHang
-- ============================================================

CREATE TRIGGER trg_SanPham_UpdateKho
ON SanPham
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    PRINT N'[Trigger] SanPham → KhoHang';

    UPDATE kh
    SET kh.SucChua = kh.SucChua + 1
    FROM KhoHang kh
    INNER JOIN inserted i
        ON kh.MaKho = i.MaKho;
END;
GO


-- ============================================================
-- TRIGGER 2
-- Khi UPDATE KhoHang → UPDATE SanPham
-- ============================================================

CREATE TRIGGER trg_KhoHang_UpdateSP
ON KhoHang
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    PRINT N'[Trigger] KhoHang → SanPham';

    UPDATE sp
    SET sp.SoLuongTon = sp.SoLuongTon + 1
    FROM SanPham sp
    INNER JOIN inserted i
        ON sp.MaKho = i.MaKho;
END;
GO


-- ============================================================
-- TEST GÂY ĐỆ QUY
-- ============================================================

UPDATE SanPham
SET SoLuongTon = SoLuongTon + 1
WHERE MaSP = 1;
GO
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/458448ad-ae24-4337-b845-049edccd4c0e" />
Trigger tạo vòng lặp và gây lỗi

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

---

## 7. CURSOR

Cursor (con trỏ) là cơ chế xử lý tập kết quả theo từng dòng một, tương tự vòng lặp trong lập trình thủ tục. Mặc dù SQL Server được tối ưu cho xử lý tập hợp (set-based), Cursor vẫn có vị trí trong một số tình huống đặc thù: xử lý logic phân nhánh phức tạp theo từng dòng, hoặc tích hợp với các tiến trình ngoài như gửi email cảnh báo.

### 7.1 Cursor duyệt tồn kho và cảnh báo

**Bài toán:** Cuối mỗi ngày làm việc, hệ thống cần duyệt qua toàn bộ danh sách sản phẩm, tính giá trị tồn kho tương ứng, in báo cáo và đánh dấu cảnh báo đối với những mặt hàng có số lượng tồn dưới mức an toàn.

# Phân tích logic xử lý Cursor quản lý tồn kho

Cursor được sử dụng để duyệt lần lượt từng sản phẩm trong các kho đang hoạt động nhằm phân tích tình trạng tồn kho và tính giá trị hàng hóa.

Đầu tiên, hệ thống khai báo các biến để lưu thông tin sản phẩm như mã sản phẩm, tên sản phẩm, tên kho, số lượng tồn, mức tồn tối thiểu và đơn giá nhập.

Sau đó, cursor `cur_TonKho` được tạo để lấy danh sách sản phẩm từ bảng `SanPham` kết hợp với bảng `KhoHang`. Dữ liệu được sắp xếp theo tên kho và tên sản phẩm.

Khi cursor hoạt động, hệ thống lần lượt đọc từng sản phẩm và thực hiện tính giá trị tồn kho bằng công thức:

* số lượng tồn × đơn giá nhập

Tiếp theo, hệ thống phân loại trạng thái tồn kho của sản phẩm. Nếu số lượng tồn bằng 0 thì sản phẩm được xác định là hết hàng. Nếu số lượng tồn nhỏ hơn hoặc bằng mức tồn tối thiểu thì hệ thống đánh dấu sắp hết hàng. Nếu số lượng tồn vẫn còn thấp nhưng chưa tới mức nguy hiểm thì được xếp vào nhóm tồn thấp. Các sản phẩm còn lại được xem là bình thường.

Trong quá trình xử lý, hệ thống đồng thời cộng dồn tổng giá trị tồn kho và đếm số sản phẩm cần cảnh báo.

Sau khi xử lý xong từng sản phẩm, hệ thống in thông tin chi tiết gồm tên sản phẩm, kho lưu trữ, số lượng tồn, giá trị tồn kho và trạng thái tồn kho.

Cuối cùng, cursor được đóng và giải phóng khỏi bộ nhớ. Hệ thống hiển thị báo cáo tổng kết gồm tổng giá trị tồn kho và số sản phẩm cần cảnh báo.


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


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/5e9695a8-6bec-46f8-91d1-49a3865ccd81" />

Sử dụng cursor báo cáo tình trạng tồn kho


# Xử lý báo cáo tồn kho theo hướng Set-Based

Thay vì sử dụng cursor để duyệt tuần tự từng sản phẩm, hệ thống có thể áp dụng phương pháp Set-Based để xử lý trực tiếp trên toàn bộ tập dữ liệu bằng các câu lệnh SQL tổng hợp.

Phương pháp này cho phép hệ thống tính toán giá trị tồn kho, phân loại trạng thái hàng hóa và thống kê dữ liệu ngay trong một truy vấn duy nhất mà không cần xử lý từng dòng dữ liệu riêng lẻ.

Giá trị tồn kho được tính bằng:

- số lượng tồn × đơn giá nhập

Hệ thống đồng thời sử dụng biểu thức `CASE` để xác định trạng thái tồn kho như:

- hết hàng
- sắp hết
- tồn thấp
- bình thường

Ngoài ra, các phép tổng hợp như tổng giá trị tồn kho và số lượng sản phẩm cần cảnh báo được thực hiện bằng các hàm `SUM` và `COUNT`.

So với cursor, phương pháp Set-Based có tốc độ xử lý nhanh hơn, tối ưu hiệu năng tốt hơn và phù hợp với các bài toán thống kê dữ liệu lớn trong hệ thống quản lý kho hàng.

---

## Code xử lý Set-Based

```sql
-- ============================================================
-- SET-BASED: Báo cáo tình trạng tồn kho không dùng CURSOR
-- ============================================================

SELECT
    sp.MaSP,
    sp.TenSP,
    kh.TenKho,
    sp.DonViTinh,
    sp.SoLuongTon,
    sp.DonGiaNhap,

    -- Giá trị tồn kho
    sp.SoLuongTon * sp.DonGiaNhap AS GiaTriTon,

    -- Phân loại trạng thái tồn kho
    CASE
        WHEN sp.SoLuongTon = 0
            THEN N'[HẾT HÀNG]'

        WHEN sp.SoLuongTon <= sp.MucTonToiThieu
            THEN N'[SẮP HẾT]'

        WHEN sp.SoLuongTon <= sp.MucTonToiThieu * 2
            THEN N'[TỒN THẤP]'

        ELSE N'[BÌNH THƯỜNG]'
    END AS TrangThaiTon

FROM SanPham sp
INNER JOIN KhoHang kh
    ON sp.MaKho = kh.MaKho

WHERE kh.TrangThai = N'HoatDong'

ORDER BY kh.TenKho, sp.TenSP;


-- ============================================================
-- THỐNG KÊ TỔNG HỢP
-- ============================================================

SELECT
    SUM(sp.SoLuongTon * sp.DonGiaNhap) AS TongGiaTriTonKho,

    COUNT(
        CASE
            WHEN sp.SoLuongTon <= sp.MucTonToiThieu
            THEN 1
        END
    ) AS SoSanPhamCanhBao

FROM SanPham sp
INNER JOIN KhoHang kh
    ON sp.MaKho = kh.MaKho

WHERE kh.TrangThai = N'HoatDong';
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/487c6975-9fa1-4474-8544-6cdd42cd8535" />

Báo cáo tình trạng tồn kho không dùng CURSOR

# So sánh xử lý báo cáo tồn kho bằng Cursor và Set-Based

Trong hệ thống quản lý kho hàng, bài toán thống kê và phân tích tồn kho có thể được triển khai theo hai phương pháp phổ biến là Cursor và Set-Based. Hai phương pháp này đều hỗ trợ xử lý dữ liệu tồn kho nhưng khác nhau về cách hoạt động, hiệu năng và mục đích sử dụng.

---

# 1. Xử lý bằng Cursor

Cursor hoạt động theo cơ chế duyệt tuần tự từng dòng dữ liệu. Hệ thống sẽ lấy từng sản phẩm trong kho, sau đó thực hiện các bước xử lý riêng cho từng bản ghi như:

- tính giá trị tồn kho
- xác định trạng thái tồn kho
- cộng dồn tổng giá trị
- đếm số lượng cảnh báo

Phương pháp này giúp dễ kiểm soát logic xử lý theo từng bước và phù hợp với các bài toán cần thao tác tuần tự trên từng dữ liệu riêng biệt.

Tuy nhiên, do phải xử lý từng dòng nên Cursor thường có tốc độ chậm hơn khi dữ liệu lớn và tiêu tốn nhiều tài nguyên hệ thống.

---

# 2. Xử lý bằng Set-Based

Set-Based xử lý toàn bộ tập dữ liệu cùng lúc thông qua các câu lệnh SQL tổng hợp như:

- `SELECT`
- `SUM`
- `COUNT`
- `CASE`
- `GROUP BY`

Thay vì duyệt từng sản phẩm, hệ thống thực hiện tính toán trực tiếp trên toàn bộ bảng dữ liệu.

Phương pháp này giúp:

- tăng tốc độ xử lý
- giảm số lần truy cập dữ liệu
- tận dụng tối ưu của SQL Server
- nâng cao hiệu năng hệ thống

Set-Based đặc biệt phù hợp với các bài toán thống kê, báo cáo và xử lý dữ liệu lớn.

---

# 3. So sánh hai phương pháp

| Tiêu chí | Cursor | Set-Based |
|---|---|---|
| Cách xử lý | Duyệt từng dòng dữ liệu | Xử lý toàn bộ tập dữ liệu |
| Tốc độ xử lý | Chậm hơn | Nhanh hơn |
| Hiệu năng | Thấp khi dữ liệu lớn | Tối ưu tốt |
| Tài nguyên hệ thống | Tốn nhiều bộ nhớ và transaction | Tối ưu hơn |
| Độ linh hoạt logic | Cao | Thấp hơn trong xử lý tuần tự |
| Khả năng mở rộng | Kém hơn | Tốt hơn |
| Phù hợp | Logic tuần tự phức tạp | Thống kê và báo cáo dữ liệu lớn |

---

# 4. Nhận xét

Qua so sánh có thể thấy Set-Based là phương pháp tối ưu hơn trong các hệ thống quản lý dữ liệu lớn nhờ khả năng xử lý nhanh và tận dụng cơ chế tối ưu truy vấn của SQL Server.

Trong bài toán quản lý tồn kho, phần lớn thao tác chỉ bao gồm thống kê, tính toán và phân loại dữ liệu nên Set-Based thường được ưu tiên sử dụng.

Ngược lại, Cursor phù hợp hơn trong các trường hợp cần xử lý tuần tự từng bản ghi hoặc cần áp dụng các logic phức tạp khó biểu diễn bằng truy vấn tập dữ liệu.

Do đó, trong thực tế nên ưu tiên Set-Based để tối ưu hiệu năng hệ thống và chỉ sử dụng Cursor khi thật sự cần thiết.


### 7.2 Cursor gửi email cảnh báo tồn kho thấp

**Bài toán:** Khi tồn kho của một mặt hàng xuống dưới mức an toàn, hệ thống cần tự động gửi email thông báo đến quản lý kho tương ứng. Cursor được sử dụng để duyệt từng sản phẩm cần cảnh báo và gọi `sp_send_dbmail` cho từng trường hợp.

> **Lưu ý:** Để sử dụng `sp_send_dbmail`, Database Mail phải được cấu hình sẵn trong SQL Server. Đoạn code dưới đây giả định đã có profile mail tên `KhoHangMailProfile`.

# Phân tích logic xử lý Cursor gửi email cảnh báo tồn kho

Đoạn mã sử dụng Cursor để duyệt lần lượt từng sản phẩm có số lượng tồn kho dưới mức an toàn nhằm gửi email cảnh báo đến quản lý kho.

Đầu tiên, hệ thống khai báo các biến dùng để lưu thông tin của từng sản phẩm như:

- mã sản phẩm
- tên sản phẩm
- tên kho
- số lượng tồn hiện tại
- mức tồn tối thiểu
- giá trị hàng còn lại
- tiêu đề email
- nội dung email

Sau đó, cursor `cur_CanhBaoEmail` được tạo để lấy danh sách các sản phẩm có tồn kho thấp hơn hoặc bằng mức tồn tối thiểu trong các kho đang hoạt động.

Khi cursor được mở, hệ thống sẽ lần lượt đọc từng sản phẩm trong danh sách cảnh báo.

Với mỗi sản phẩm, hệ thống thực hiện:

- tạo tiêu đề email cảnh báo
- tạo nội dung email dạng HTML
- chèn thông tin sản phẩm vào nội dung email
- gửi email thông qua `sp_send_dbmail`

Nếu gửi email thành công, hệ thống sẽ tăng biến đếm số email đã gửi và hiển thị thông báo xác nhận.

Ngược lại, nếu phát sinh lỗi trong quá trình gửi mail, hệ thống sẽ bắt lỗi bằng `TRY...CATCH` và hiển thị thông báo lỗi tương ứng.

Quá trình này tiếp tục cho đến khi cursor duyệt hết toàn bộ sản phẩm cần cảnh báo.

Cuối cùng, cursor được đóng, giải phóng bộ nhớ và hệ thống in ra tổng số email đã gửi thành công.

---

# Tại sao bắt buộc phải dùng Cursor

Trong bài toán này, Cursor gần như là bắt buộc vì hệ thống cần xử lý riêng biệt cho từng sản phẩm và từng email.

Mỗi sản phẩm sẽ có:

- tiêu đề email khác nhau
- nội dung email khác nhau
- thông tin tồn kho khác nhau
- thời điểm gửi và trạng thái gửi riêng biệt

Việc gửi email bằng `sp_send_dbmail` là thao tác xử lý theo từng lần gọi thủ tục (row-by-row operation), không thể thực hiện đồng thời cho toàn bộ tập dữ liệu bằng một câu lệnh Set-Based thông thường.

Nếu sử dụng Set-Based:

- khó tạo nội dung HTML riêng cho từng sản phẩm
- không thể gọi `sp_send_dbmail` cho nhiều dòng dữ liệu trong cùng một lệnh SELECT
- khó kiểm soát lỗi gửi mail theo từng sản phẩm
- khó theo dõi số lượng email gửi thành công hoặc thất bại

Trong khi đó, Cursor cho phép:

- xử lý tuần tự từng sản phẩm
- tạo nội dung email động cho từng bản ghi
- gửi mail riêng biệt
- bắt lỗi riêng cho từng lần gửi
- kiểm soát luồng xử lý chi tiết

Do đó, khác với các bài toán thống kê dữ liệu thông thường, bài toán gửi email cảnh báo là trường hợp phù hợp và gần như bắt buộc phải sử dụng Cursor.



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

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/9d21ebba-361c-4719-8155-c25640bdac66" />

Gửi email cảnh báo tồn kho thấp đến quản lý
