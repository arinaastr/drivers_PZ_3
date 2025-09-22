# drivers_PZ_3
Компиляция модуля ядра:
```
nano hello.c
nano Makefile
make
```
Вывод:
```
make -C /lib/modules/6.8.0-79-generic/build M=/home/drivers/linux-6.8/driver modules
make[1]: Entering directory '/usr/src/linux-headers-6.8.0-79-generic'
warning: the compiler differs from the one used to build the kernel
  The kernel was built by: x86_64-linux-gnu-gcc-13 (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
  You are using:           gcc-13 (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0
  CC [M]  /home/drivers/linux-6.8/driver/hello.o
  MODPOST /home/drivers/linux-6.8/driver/Module.symvers
  CC [M]  /home/drivers/linux-6.8/driver/hello.mod.o
  LD [M]  /home/drivers/linux-6.8/driver/hello.ko
  BTF [M] /home/drivers/linux-6.8/driver/hello.ko
Skipping BTF generation for /home/drivers/linux-6.8/driver/hello.ko due to unavailability of vmlinux
make[1]: Leaving directory '/usr/src/linux-headers-6.8.0-79-generic'
```
Загрузка модуля в ядро:
```
sudo insmod hello.ko
```
Проверка загруженного модуля:
```
lsmod | grep hello
```
Вывод:
```
hello                  12288  0
```
Просмотр логов ядра:
```
sudo dmesg | tail
```
Вывод:
```
[27652.485415]  x86_64_start_reservations+0x18/0x30
[27652.485418]  x86_64_start_kernel+0xbf/0x110
[27652.485421]  secondary_startup_64_no_verify+0x184/0x18b
[27652.485425]  </TASK>
[27652.509160] kauditd_printk_skb: 89 callbacks suppressed
[27652.509166] audit: type=1400 audit(1758527211.458:620): apparmor="DENIED" operation="open" class="file" profile="rsyslogd" name="/run/systemd/sessions/" pid=858 comm=72733A6D61696E20513A526567 requested_mask="r" denied_mask="r" fsuid=103 ouid=0
[28140.186535] workqueue: ata_sff_pio_task hogged CPU for >10000us 4 times, consider switching to WQ_UNBOUND
[28156.642098] workqueue: e1000_watchdog [e1000] hogged CPU for >10000us 16 times, consider switching to WQ_UNBOUND
[32075.654150] Trying to initialize the driver
[32075.654163] Successfully initialized the driver
```
Проверка, зарегистрирован ли символьный драйвер в системе и какой major number ему присвоен:
```
cat /proc/devices | grep foo
```
Вывод:
```
240 foo
```
```
sudo mknod -m 660 /dev/foo c 240 0
```
Проверка:
```
ls -l /dev/foo
```
Вывод:
```
crw-rw---- 1 root root 240, 0 Sep 22 09:10 /dev/foo
```
Написание и запуск пользовательского приложения:
```C
// test_app.c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    const char* device = "/dev/foo";  // ← Должно совпадать с именем, которое ты создала через mknod
    int fd = open(device, O_RDWR);

    if (fd < 0) {
        perror("Failed to open device");
        return 1;
    }

    printf("Successfully opened %s with fd = %d\n", device, fd);

    // Можно добавить read/write, если драйвер их поддерживает
    // Но даже просто open/close — уже выполняет задание

    close(fd);
    printf("Device closed.\n");

    return 0;
}
```
gcc -o app app.c
sudo ./app
```
Вывод:
```
Successfully opened /dev/foo with fd = 3
Device closed.
```
