# Hi3518E-IP-Camera

# Cheap chinese IP camera with H264 encoding based on Hisilicon 8M (Hi3518E) chip

Video stream url for VLC/DVR:
  * rtsp://192.168.1.123:554/user=admin&password=&channel=&stream=.sdp?real_stream--rtp-caching=100

Telnet access
  * telnet 192.168.1.123 23
  * Localhost login: root
  * Password: xmhdipc

Configuration placed at this path:
  * /mnt/mtd/Config

To change static IP Address:
  * armbenv
  * netinit eth0 192.168.1.123

For enabling DHCP
  * echo 1 > mnt/mtd/Config/dhcp.cfg

Show single frame
  * http://192.168.1.123/webcapture.jpg?command=snap&chanel=1

Interesting links:
  * http://marcusjenkins.com/linux/hacking-cheap-ebay-ip-camera/
  * http://www.hkvstar.com/technology-news/china-ip-camera-configuration-firmware.html


```
# cat /proc/cpuinfo
Processor       : ARM926EJ-S rev 5 (v5l)
BogoMIPS        : 218.72
Features        : swp half thumb fastmult edsp java
CPU implementer : 0x41
CPU architecture: 5TEJ
CPU variant     : 0x0
CPU part        : 0x926
CPU revision    : 5

Hardware        : hi3518
Revision        : 0000
Serial          : 0000000000000000
```


```
# ps
  PID USER       VSZ STAT COMMAND
    1 root      1240 S    init
    2 root         0 SW   [kthreadd]
    3 root         0 SW   [ksoftirqd/0]
    4 root         0 SW   [kworker/0:0]
    5 root         0 SW   [kworker/u:0]
    6 root         0 SW   [rcu_kthread]
    7 root         0 SW<  [khelper]
    8 root         0 SW   [kworker/u:1]
  119 root         0 SW   [sync_supers]
  121 root         0 SW   [bdi-default]
  122 root         0 SW<  [kintegrityd]
  124 root         0 SW<  [kblockd]
  137 root         0 SW   [khubd]
  148 root         0 SW<  [cfg80211]
  149 root         0 SW   [kworker/0:1]
  231 root         0 SW<  [rpciod]
  234 root         0 SW   [kswapd0]
  288 root         0 SW   [fsnotify_mark]
  291 root         0 SW<  [nfsiod]
  302 root         0 SW<  [crypto]
  372 root         0 SW   [mtdblock0]
  377 root         0 SW   [mtdblock1]
  382 root         0 SW   [mtdblock2]
  387 root         0 SW   [mtdblock3]
  392 root         0 SW   [mtdblock4]
  397 root         0 SW   [mtdblock5]
  431 root         0 SW<  [wusbd]
  440 root         0 SW<  [kpsmoused]
  465 root       872 S <  udevd --daemon
  471 root         0 SWN  [jffs2_gcd_mtd5]
  703 root      1552 S    /utils/upgraded
  711 root      2440 S    searchIp
  713 root      9536 S    dvrHelper /lib/modules /usr/bin/Sofia 127.0.0.1 9578 1
  714 root      1244 S    telnetd
  728 root      482m S    /usr/bin/Sofia
  836 root      1264 S    -sh
  853 root      1240 R    ps
```


```
# ls /bin
BurnHWID   cat        env        hush       ln         netinit    searchIp   true
[          chmod      false      ip         login      netstat    sed        tty
[[         cp         fgrep      ipaddr     ls         ping       sh         udevd
armbenv    date       free       iplink     mkdir      pppd       sleep      udevinfo
arping     dd         grep       iproute    mkfifo     pppoe      sync       udevstart
ash        dvrHelper  himc       iprule     mknod      ps         sysinit    udpsvd
awk        dvrbox     himd       iptunnel   mount      pwd        test       umount
btools     echo       himd.l     kill       msh        rm         top        upgraded
busybox    egrep      himm       killall    mv         rmdir      touch      xargs
```

```
# armbenv -r
LibCrypto : g_cryptotype = 2
**********************************************************************
|                      SYSTEM INFO
|                 ID:           8043420004048425
|       product type:           50H10L
|            product:           HI3518E_50H10L_S39
|      video channel:           1
|      audio channel:           1
|           alarm in:           1
|          alarm out:           1
| forward video chip:           OV9712
|           DSP chip:           HI3518E
|  analog audio mode:           voice codec
|           talkback:           voice codec
|    back video chip:           no chip
|    store interface:           SDIO
|    matrix surpport:           No
| wireless interface:           USB
|    hardware encode:           encode chip
|   hardware version:           1
|    video_interface:           BNC
|      net_interface:           Ethernet
|  hardware info len:           8
**********************************************************************
LIBDVR: Complied at Jun 12 2015 19:34:48 SVN:1028
bootdelay = 1
baudrate = 115200
serverip = 192.168.1.1
ipaddr = 192.168.1.123
netmask = 255.255.255.0
ethaddr = 00:12:13:xx:xx:xx
HWID = 8043420004xxxxxx
ob_start = 0
ob_data = 82
```


```
# cat /etc/init.d/rcS
#! /bin/sh

/etc/init.d/dnode

udevd --daemon
udevstart

mount -t squashfs /dev/mtdblock2 /usr
mount -t squashfs /dev/mtdblock3 /mnt/web
mount -t squashfs /dev/mtdblock4 /mnt/custom
mount -t jffs2 /dev/mtdblock5 /mnt/mtd

mount -t ramfs  /dev/mem        /var/
mkdir -p /var/tmp
mount -t ramfs  /dev/mem2       /utils
mount -t usbfs usbfs /proc/bus/usb/

mkdir -p /mnt/mtd/Config /mnt/mtd/Log /mnt/mtd/Config/ppp /mnt/mtd/Config/Json
if [ -f /mnt/mtd/Config/ppp/3gdigal ]; then
        chmod 777 /mnt/mtd/Config/ppp/3gdigal
fi

/usr/etc/loadmod
netinit
cp /bin/upgraded /utils/ -f
/utils/upgraded &
ifconfig eth2 down
telnetd &
sysinit &
searchIp &
#wlandaemon &
#route_switch &

/bin/pppd pty /etc/ppp/pppoe-start file /etc/ppp/pppoe-options &
if [ -f /mnt/custom/extapp.sh ];then
        /mnt/custom/extapp.sh &
fi
dvrHelper /lib/modules /usr/bin/Sofia 127.0.0.1 9578 1 &
```
