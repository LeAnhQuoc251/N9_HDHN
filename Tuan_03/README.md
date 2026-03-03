TUẦN 3: XÂY DỰNG ROOT FILE SYSTEM (ROOTFS) - Nhóm 9
==========================================

Mục tiêu
--------

Biên dịch thành công RootFS sử dụng BusyBox\
Tạo cấu trúc thư mục Linux cơ bản\
Liên kết Kernel với RootFS\
Boot thành công trên BeagleBone Black

* * * * *

BÀI TẬP 01: CÀI ĐẶT ROOTFS CƠ BẢN
=================================

1\. Biên dịch BusyBox
---------------------

Sử dụng BusyBox 1.38.0.git

```
make menuconfig
make
make install

```

Kết quả\
Thư mục `_install/` được tạo.\
Chứa các binary hệ thống tối giản.

Kiểm tra symbolic link:

```
ls -l _install/bin/ls

```

Các lệnh như ls, cp, mkdir đều là symbolic link trỏ về binary BusyBox để tiết kiệm dung lượng.

* * * * *

2\. Tạo và cấu hình phân vùng RootFS
------------------------------------

Phân vùng thẻ nhớ\
Tạo phân vùng thứ 2.\
Định dạng ext4.\
Label rootfs.

Copy dữ liệu

```
cp -r _install/* /media/rootfs/

```

Tạo thư mục hệ thống cần thiết

```
mkdir -p dev etc proc sys tmp var root mnt

```

Cấu trúc thư mục sau khi hoàn thành

```
/
├── bin
├── dev
├── etc
├── proc
├── sys
├── tmp
├── var
└── root

```

* * * * *

3\. Liên kết Kernel với RootFS (Bootargs)
-----------------------------------------

Cấu hình trong môi trường U-Boot

```
setenv bootargs 'console=ttyO0,115200n8 root=/dev/mmcblk0p2 rw rootwait'
saveenv

```

Giải thích\
console=ttyO0,115200n8 cấu hình UART\
root=/dev/mmcblk0p2 chỉ định phân vùng chứa RootFS\
rw cho phép đọc ghi\
rootwait chờ thiết bị MMC sẵn sàng

* * * * *

4\. Khởi động hệ thống
----------------------

Kernel boot thành công.\
RootFS được mount từ thẻ nhớ.\
Đăng nhập với quyền root.

Kiểm tra:

```
ls -l /

```

Hệ thống hoạt động ổn định.

* * * * *

BÀI TẬP 02: KIỂM TRA VÀ TÙY CHỈNH ROOTFS
========================================

1\. Kiểm tra editor mặc định
----------------------------

```
vi
nano

```

Kết quả\
vi hoạt động ở phiên bản rút gọn.\
nano báo lỗi: -sh: nano: not found.

BusyBox mặc định không tích hợp nano.

* * * * *

2\. Tùy chỉnh BusyBox
---------------------

Thực hiện trên Host Ubuntu 24.04

```
make menuconfig

```

Vào mục Editors và kích hoạt các tùy chọn cần thiết cho vi.\
Sau đó biên dịch lại:

```
make
make install

```

* * * * *

3\. Cập nhật RootFS và kiểm tra
-------------------------------

Copy lại nội dung `_install/` vào thẻ nhớ và khởi động lại hệ thống.

Tạo file kiểm tra:

```
vi hoanthanh_bai2.txt

```

Lưu file bằng:

```
:wq

```

Kiểm tra nội dung:

```
cat hoanthanh_bai2.txt

```

Editor hoạt động bình thường sau khi tùy chỉnh.

* * * * *
