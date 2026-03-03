TUẦN 4: QUY TRÌNH BUILD HỆ ĐIỀU HÀNH BẰNG BUILDROOT - Nhóm 9
===================================================

* * * * *

A. MỤC TIÊU
-----------

-   Build hoàn chỉnh hệ điều hành cho **BeagleBone Black (BBB)** gồm:

    -   `MLO`

    -   `u-boot.img`

    -   `zImage`

    -   `am335x-boneblack.dtb`

    -   `RootFS`

-   Sử dụng Toolchain tạo ra để biên dịch chéo chương trình C.

-   Tùy biến và thêm Package vào hệ thống.

* * * * *

B. BUILD HỆ ĐIỀU HÀNH
---------------------

### 1\. Chuẩn bị môi trường (Host Setup)

```
sudo apt update
sudo apt install build-essential checkinstall libncursesw5-dev\
python3-dev python3-setuptools-pynacl python3-pip\
libssl-dev curltcl-dev git bc bzr cvs mercurial\
unzip wget rsync fastjar java-wrappers bison flex texinfo -y

```

* * * * *

### 2\. Tải mã nguồn Buildroot

```
git clone https://github.com/buildroot/buildroot.git
cd buildroot
git checkout 2023.11.x

```

* * * * *

### 3\. Cấu hình cho BeagleBone Black

```
make beaglebone_defconfig

```

* * * * *

### 4\. Tùy chỉnh cấu hình hệ thống

```
make menuconfig

```

Thiết lập:

-   **Target Options**

    -   Architecture: ARM (Cortex-A8)

    -   ABI: EABIhf

-   **Toolchain**

    -   Buildroot toolchain

    -   glibc

-   **System Configuration**

    -   Console: `ttyS0`

    -   Baudrate: `115200`

-   **Target Packages**

    -   System Tools → bật:

        -   `htop`

        -   `nano`

* * * * *

### 5\. Build hệ thống

```
make -j$(nproc)

```

Kết quả:

```
output/images/sdcard.img

```

* * * * *

### 6\. Flash vào thẻ nhớ

Sử dụng:

-   balenaEtcher\
    hoặc

```
sudo dd if=sdcard.img of=/dev/sdX bs=4M status=progress

```

* * * * *

### 7\. Kiểm tra sau khi boot

Trên BBB:

```
htop

```

Nếu hiển thị trình quản lý tiến trình → tùy biến Package thành công.

* * * * *

C. SỬ DỤNG TOOLCHAIN TỪ BUILDROOT
---------------------------------

* * * * *

### 1\. Tạo chương trình C

```
mkdir ~/bai_tap_02
cd ~/bai_tap_02
nano hello.c

```

Nội dung:

```
#include <stdio.h>

int main() {
    printf("Hello BeagleBone Black! Toi la Jap day.\n");
    return 0;
}

```

* * * * *

### 2\. Biên dịch chéo

```
~/buildroot/output/host/bin/arm-linux-gcc -o hello_bbb hello.c

```

* * * * *

### 3\. Đưa chương trình vào BBB bằng Base64

Trên Host:

```
base64 hello_bbb

```

Copy toàn bộ đoạn mã.

* * * * *

Trên BBB:

```
base64 -d <<EOF > hello_final
[dán mã base64 vào đây]
EOF

```

Cấp quyền:

```
chmod +x hello_final

```

* * * * *

### 4\. Chạy thử

```
./hello_final

```

Kết quả:

```
Hello BeagleBone Black! Toi la Jap day.

```

* * * * *

D. TÍCH HỢP ỨNG DỤNG VÀO BUILDROOT
----------------------------------

Quy trình:

1.  Tạo thư mục:

```
package/hello-G5/src

```

1.  Tạo file `Config.in`

2.  Tạo file `hello-G5.mk`

3.  Thêm vào `package/Config.in`

4.  Bật trong `menuconfig`

5.  Build lại:

```
make

```

Sau khi flash lại thẻ nhớ:

```
hello-G5

```

Có thể chạy từ mọi thư mục.

* * * * *

KẾT LUẬN
========

Tuần 4 đã:

-   Build thành công hệ điều hành bằng Buildroot

-   Tùy biến Package

-   Biên dịch chéo chương trình C

-   Tích hợp ứng dụng vào hệ thống tự động

* * * * *

