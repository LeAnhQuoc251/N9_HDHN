TUẦN 8 - Sử dụng các công cụ gỡ lỗi và đánh giá hiệu
năng cơ bản - NHÓM 9
--------------------------------

* * * * *
1. MỤC TIÊU
--------------

-   Sử dụng các công cụ:
    -   `gdb` (debug)
    -   `valgrind` (phân tích bộ nhớ)
    -   `perf` (phân tích hiệu năng)
    -   `strace`, `ltrace` (tracing)
-   Phát hiện lỗi và phân tích chương trình trên Linux

* * * * *
2. CHUẨN BỊ HỆ THỐNG (BUILDROOT)
-----------------------------------

### Bước 1: Mở cấu hình

```
make menuconfig
```

### Bước 2: Bật debug symbols

```
Build options  ---> build packages with debugging symbols
```

### Bước 3: Chọn công cụ

-   gdb
-   valgrind
-   strace
-   ltrace
-   perf

### Bước 4: Build hệ thống

```
make
```

* * * * *

3. PHÂN TÍCH BỘ NHỚ (VALGRIND)
---------------------------------

### Bước 1: Tạo chương trình lỗi

```
nano leak.c
```

```
#include <stdio.h>#include <stdlib.h>#include <string.h>int main(){    char *buffer = malloc(100);    strcpy(buffer, "Hello memory leak test");    printf("Data: %s\n", buffer);    return 0;   // không free → gây leak}
```

### Bước 2: Biên dịch

```
gcc -g leak.c -o leak
```

### Bước 3: Chạy chương trình

```
./leak
```

**Kết quả:**

```
Data: Hello memory leak test
```

### Bước 4: Kiểm tra bằng valgrind

```
valgrind --leak-check=full ./leak
```

**Kết quả:**

```
definitely lost: 100 bytes
```

### Bước 5: Sửa lỗi

```
nano leak.c
```

Sửa:

```
free(buffer);
```

### Bước 6: Biên dịch lại

```
gcc -g leak.c -o leak
```

### Bước 7: Kiểm tra lại

```
valgrind --leak-check=full ./leak
```

**Kết quả cuối:**

```
All heap blocks were freed -- no leaks are possible
```

* * * * *

4. PHÂN TÍCH CORE DUMP
-------------------------

### Bước 1: Tạo chương trình crash

```
nano crash.c
```

```
#include <stdio.h>void cause_crash(){    int *ptr = NULL;    *ptr = 42;}int main(){    printf("Start program\n");    cause_crash();    return 0;}
```

### Bước 2: Biên dịch

```
gcc -g crash.c -o crash
```

### Bước 3: Bật core dump

```
ulimit -c unlimited
```

### Bước 4: Chạy chương trình

```
./crash
```

**Kết quả:**

```
Segmentation fault (core dumped)
```

### Bước 5: Kiểm tra file core

```
ls -l
```

**Xuất hiện:**

```
core.crash.<pid>
```

### Bước 6: Mở bằng GDB

```
gdb ./crash core.*
```

### Bước 7: Xem stack trace

```
bt
```

**Kết quả:**

```
#0 cause_crash()#1 main()
```

* * * * *

5. PHÂN TÍCH HIỆU NĂNG (PERF)
-------------------------------

### Bước 1: Cài công cụ

```
sudo apt install linux-tools-common linux-tools-generic -y
```

### Bước 2: Tạo chương trình

```
nano perf_test.c
```

```
#include <stdio.h>int main(){    long long sum = 0;    for (long long i = 0; i < 100000000; i++)    {        sum += i;    }    printf("sum = %lld\n", sum);    return 0;}
```

### Bước 3: Biên dịch

```
gcc -O0 -g perf_test.c -o perf_test
```

### Bước 4: Chạy thử

```
./perf_test
```

### Bước 5: Đo hiệu năng

```
perf stat -e task-clock,context-switches,cpu-migrations,page-faults ./perf_test
```

### Bước 6: Kết quả hiển thị

-   task-clock
-   context-switches
-   cpu-migrations
-   page-faults
-   seconds time elapsed

### Bước 7: Lưu báo cáo

```
perf stat ./perf_test 2> perf_report.txtcat perf_report.txt
```

* * * * *

6. PHÂN TÍCH TRACING
-----------------------

### Bước 1: Tạo chương trình

```
nano trace_test.c
```

```
#include <stdio.h>#include <unistd.h>int main(){    FILE *fp;    char buffer[100];    printf("Start tracing program\n");    fp = fopen("data.txt", "w");    fprintf(fp, "Hello tracing test\n");    fclose(fp);    fp = fopen("data.txt", "r");    fgets(buffer, sizeof(buffer), fp);    fclose(fp);    printf("Read data: %s", buffer);    sleep(1);    return 0;}
```

### Bước 2: Biên dịch

```
gcc -g trace_test.c -o trace_test
```

### Bước 3: Chạy chương trình

```
./trace_test
```

**Kết quả:**

```
Start tracing programRead data: Hello tracing test
```

### Bước 4: Sử dụng strace

```
strace ./trace_test
```

**Quan sát:**

-   openat
-   write
-   read
-   close
-   clock_nanosleep

### Bước 5: Lưu log

```
strace ./trace_test 2> strace_report.txtcat strace_report.txt
```

### Bước 6: Sử dụng ltrace

```
ltrace ./trace_test
```

**Kết quả:**

-   fopen
-   fprintf
-   fgets
-   fclose
-   printf
-   sleep
