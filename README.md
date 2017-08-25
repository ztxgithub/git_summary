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

```

- ls /dev/

``` shell

    mmcblk0p1 

```

- dmesg



