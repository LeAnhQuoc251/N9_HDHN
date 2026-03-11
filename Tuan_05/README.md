# Tuần 5 - Bài tập HDH Nhúng - Biên dịch chéo thư viện và ứng dụng - Nhóm 9
### Bài tập 01: Biên dịch ứng dụng với thư viện đã có

Yêu cầu: Viết 01 chương trình C/C++ có sử dụng thư viện cJSON.

1.1. Bật cJSON trong Buildroot và build lại OS tích hợp thư viện cJSON

1.2. Viết chương trình C/C++ có sử dụng thư viện CJSON thực hiện parse 01 gói tin JSON
thành các trường thông tin và in lên Terminal.

1.3. Thực hiện biên dịch chéo sử dụng Toolchain đã cập nhật ở bước 1 (VD: Tên là
HelloJSON.c)

1.4. Biên dịch thành công chương trình HelloJSON.c và đưa xuống BBB.

1.5. Khởi chạy thành công chương trình đã biên dịch.

### Bài tập 02: Tự tạo thư viện cá nhân

Yêu cầu: Tự tạo 01 thư viện đơn giản của riêng mình và ứng dụng
sử dụng thư viện đó

2.1. Viết 01 thư viện đơn giản gồm 01 file .h và 01 file .c thực hiện 1 nhiệm vụ đơn giản như:
Cộng 2 số, Tìm số nguyên tố.... (tùy ý).

2.2. Biên dịch thành công thư viện ở bước 1 thành: Static Library (.a) và Dynamic Library
(.so)

2.3. Đưa thư viện đã biên dịch thành công ở bước 2 vào Sysroot.

2.4. Viết chương trình C/C++ có sử dụng thư viện đã tạo

2.5. Biên dịch chương trình với 2 loại thư viện (Thư viện tĩnh và Thư viện động) thành 2
chương trình.

2.6. Đưa chương trình và thư viện đã biên dịch xuống BBB (Cả 2 chương trình) thử nghiệm
hoạt động.

2.7. So sánh về kích thước của 2 chương trình đã tạo ở bước (5) về dung lượng, yêu cầu
phụ thuộc (sử dụng lệnh read-elf dependencies).

### Bài tập 03: Tích hợp ứng dụng và thư viện và Buildroot

Yêu cầu: Xây dựng chương trình có phụ thuộc vào cả 2 thư viện ở
Bài 1 và Bài 2 vào Buildroot có ràng buộc phụ thuộc.

3.1. Đưa thư viện ở bài 2 vào Buildroot và biên dịch tích hợp thành công

3.2. Viết 01 chương trình C/C++ có sử dụng cả 2 thư viện ở bài 01 và bài 02.

3.3. Biên dịch thành công chương trình và chạy thành công trên BBB

3.4. Viết cấu hình cho chương trình trên Buildroot, chú ý kèm điều kiện phụ thuộc vào 02 thư
viện đã nêu. (Khi bật (enable) chương trình này, CJSON và thư viện tùy chỉnh tự động
được kích hoạt).

3.5. Biên dịch lại Buildroot, cài đặt và khởi chạy chương trình thành công trên BBB ngay sau
khi cài đặt.

# Thực hiện 

Tổng quan dự án

Dự án giải quyết 03 bài toán cốt lõi trong phát triển Linux nhúng:

Sử dụng thư viện có sẵn (cJSON) bằng Toolchain của Buildroot.

Tự thiết kế thư viện động (mymath) và quản lý Sysroot.

Đóng gói ứng dụng thành Package chuẩn, thiết lập ràng buộc phụ thuộc (Dependencies) trong Buildroot.


Bài tập 01: Biên dịch ứng dụng với thư viện đã có

Mục tiêu

Parse gói tin JSON và in lên Terminal BBB.


1\. Bật cJSON trong Buildroot

Truy cập:

```
make menuconfig

```

Target packages\
Libraries\
JSON/XML

Chọn:

```
[*] cjson

```

Thực hiện:

```
make

```

để Buildroot tải và biên dịch thư viện tích hợp vào OS.


2\. Mã nguồn HelloJSON.c

```
#include <stdio.h>
#include <cjson/cJSON.h>

int main() {
    const char *json_string = "{\"name\":\"Jap-BBB\",\"lab\":1,\"status\":\"Success\"}";
    cJSON *json = cJSON_Parse(json_string);
    if (json == NULL) return 1;
    printf("--- KET QUA PARSE JSON ---\n");
    printf("Device: %s\n", cJSON_GetObjectItem(json, "name")->valuestring);
    printf("Status: %s\n", cJSON_GetObjectItem(json, "status")->valuestring);
    cJSON_Delete(json);
    return 0;
}

```


3\. Biên dịch chéo và nạp xuống mạch

Trỏ Toolchain:

```
export CC=~/workspace/buildroot/output/host/bin/arm-buildroot-linux-gnueabihf-gcc

```

Biên dịch liên kết cJSON:

```
$CC HelloJSON.c -o HelloJSON -lcjson

```

Copy xuống BBB:

Sử dụng thẻ nhớ, copy file `HelloJSON` vào thư mục `/root`\
và file `libcjson.so*` vào `/usr/lib` trên phân vùng rootfs.


4\. Khởi chạy trên BBB

```
chmod +x HelloJSON
./HelloJSON

```


Bài tập 02: Tự tạo thư viện cá nhân

Mục tiêu

Tạo thư viện toán học **mymath** và so sánh thư viện tĩnh / động.


1\. Mã nguồn thư viện mymath.h và mymath.c

```
// mymath.h
#ifndef MYMATH_H
#define MYMATH_H

int cong_hai_so(int a, int b);

#endif

```

```
// mymath.c
#include "mymath.h"

int cong_hai_so(int a, int b) {
    return a + b;
}

```


2\. Biên dịch thư viện (.a và .so)

Tạo Object file

```
$CC -c -fPIC mymath.c -o mymath.o

```

Tạo Static Library

```
~/workspace/buildroot/output/host/bin/arm-buildroot-linux-ar rcs libmymath.a mymath.o

```

Tạo Dynamic Library

```
$CC -shared -o libmymath.so mymath.o

```


3\. Đưa vào Sysroot
```
cp mymath.h ~/workspace/buildroot/output/host/arm-buildroot-linux-gnueabihf/sysroot/usr/include/

cp libmymath.a libmymath.so\
~/workspace/buildroot/output/host/arm-buildroot-linux-gnueabihf/sysroot/usr/lib/

```


4\. Ứng dụng test_math.c và so sánh

Biên dịch

Tĩnh

```
$CC test_math.c -o app_static -lmymath -static

```

Động

```
$CC test_math.c -o app_dynamic -lmymath

```

So sánh

Dung lượng

```
ls -lh

```

Kết quả:\
`app_static` nặng hơn (khoảng vài trăm KB) do bao gồm cả mã nguồn thư viện\
`app_dynamic` rất nhẹ (vài KB)

Phụ thuộc

```
readelf -d app_dynamic

```

Thấy yêu cầu thư viện `libmymath.so`.


Bài tập 03: Tích hợp ứng dụng và thư viện vào Buildroot

Mục tiêu
--------

Xây dựng ứng dụng **app_kethop** phụ thuộc vào cả **cJSON** và **mymath**.


1\. Mã nguồn ứng dụng app_kethop.c

```
#include <stdio.h>
#include <stdlib.h>
#include <cjson/cJSON.h>
#include <mymath.h>

int main() {
    int x = 100, y = 50;
    int ket_qua = cong_hai_so(x, y); // Dùng thư viện mymath

    // Dùng cJSON để đóng gói
    cJSON *root = cJSON_CreateObject();
    cJSON_AddStringToObject(root, "thong_diep", "Ket hop thanh cong cJSON va MyMath");
    cJSON_AddNumberToObject(root, "so_a", x);
    cJSON_AddNumberToObject(root, "so_b", y);
    cJSON_AddNumberToObject(root, "tong", ket_qua);

    char *string = cJSON_Print(root);
    printf("%s\n", string);

    free(string);
    cJSON_Delete(root);
    return 0;
}

```

2\. Tạo Package app_kethop trong Buildroot (Ý 4)

Tạo thư mục

```
mkdir -p package/app_kethop

```

### File Config.in (Ràng buộc phụ thuộc)

```
config BR2_PACKAGE_APP_KETHOP
    bool "app_kethop"
    select BR2_PACKAGE_CJSON
    select BR2_PACKAGE_MYMATH
    help
      Chuong trinh ket hop cJSON va MyMath (Bai Tap 03).
      Tu dong kich hoat cac thu vien phu thuoc.

```

### File app_kethop.mk

```
APP_KETHOP_VERSION = 1.0
APP_KETHOP_SITE = $(HOME)/BaiTap2
APP_KETHOP_SITE_METHOD = local
APP_KETHOP_DEPENDENCIES = cjson mymath

define APP_KETHOP_BUILD_CMDS
	$(TARGET_CC) $(TARGET_CFLAGS) $(@D)/app_kethop.c -o $(@D)/app_kethop -lcjson -lmymath
endef

define APP_KETHOP_INSTALL_TARGET_CMDS
	$(INSTALL) -D -m 0755 $(@D)/app_kethop $(TARGET_DIR)/usr/bin/app_kethop
endef

$(eval $(generic-package))

```

3\. Khai báo và Biên dịch (Ý 5)

Thêm dòng sau vào file:

```
package/Config.in

```

```
source "package/app_kethop/Config.in"

```

Sau đó chạy

```
make menuconfig

```

Tích chọn

```
app_kethop

```

Kiểm tra thấy **cJSON và mymath tự động bật**.

Build package

```
make app_kethop

```

Kiểm tra kết quả

```
ls -l output/target/usr/bin/app_kethop

```


4\. Khởi chạy thành công trên BBB

### Cách 1 (Mount thẻ nhớ Ý 3)

```
mount /dev/mmcblk0p1 /mnt/card
./app_kethop

```

### Cách 2 (Sau khi cài đặt Ý 5)

Gõ trực tiếp

```
app_kethop

```

từ bất cứ đâu.

Kết quả

Màn hình in ra chuỗi JSON

```
{"ten_bai": "Bai Tap 03 - Ket hop", "tong": 150}

```







