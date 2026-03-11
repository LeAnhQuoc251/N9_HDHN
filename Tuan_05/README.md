# Tuần 5 - Bài tập HDH Nhúng - Biên dịch chéo thư viện và ứng dụng - Nhóm 9
### Bài tập 01: Biên dịch ứng dụng với thư viện đã có

## Tổng quan dự án

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

sự phụ thuộc :
```
/home/jap/workspace/buildroot/output/host/bin/arm-linux-readelf -d test_dynamic
/home/jap/workspace/buildroot/output/host/bin/arm-linux-readelf -d test_static
```

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









