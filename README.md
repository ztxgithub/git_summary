# 项目过程的注意事项
 
## 命令行

- shell脚本在编写完后，一定要注意加**可执行权限**

``` shell

chmod +x *.sh

```

### sd卡

- mount

``` shell

none on /proc type proc (rw,relatime)
none on /sys type sysfs (rw,relatime)
/tmp on /tmp type tmpfs (rw,relatime)
none on /home type ramfs (rw,relatime)
none on /proc/bus/usb type usbfs (rw,relatime)
none on /dev/pts type devpts (rw,relatime,mode=600)
none on /dev/shm type tmpfs (rw,relatime)
******************************************************************************************************************************
**/dev/mmcblk0p1 on /media/sd type ext4 (rw,noatime,nodiratime,errors=remount-ro,commit=2,barrier=1,nodelalloc,data=journal)**
******************************************************************************************************************************
ubi1_0 on /opt type ubifs (rw,sync,relatime)
ubi0_0 on /etc type ubifs (rw,sync,relatime)
ubi2_0 on /yy/log type ubifs (rw,sync,relatime)

    1.mount sd卡将其格式化为 fat 文件系统
        > mkdir -p /media/sd
        > mount -t vfat /dev/mmcblk0p1 /media/sd
        
    2.mount 其他存储设备
        > mount -t vfat /dev/sda1 /yy/

```

- ls /dev/

``` shell

    mmcblk0p1 

```

- dmesg

## 基础知识

```shell
    1.MSVC: Microsoft Visual C++ 集成编译工具
```



