# rpi-setup-tutorial
Setup tutorial for my RPi5

## Image Setup

Download the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) tool and use it for setting up a 16GB SD card whith the Raspberry Pi OS (64-bit) Lite.

In the RPi Imager tool, you can setup your WiFi network, your locale and a ssh server for remote connections.

## First Boot

Connect to the SSH connection which was set in the last step.

```console
$ ssh pi@rpi
pi@rpi's password: 
Linux raspberrypi5 6.6.20+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.6.20-1+rpt1 (2024-03-07) aarch64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Mar 19 11:29:41 2024 from 192.168.1.45
```

Then the running OS and packages should be updated.

```shell
sudo apt-get update
sudo apt-get upgrade
```

After that the raspberri pi configuration tool should be executed

```shell
sudo raspi-config 
```

The `raspi-config` tool will allow to:

* Resize the image to the full SD card size
* Setup our locale
* Lot of things...

## IPTABLES

The package `iptables` will setup a firewall to add to protect the PRi from unwanted access.

First, the `iptables` package should be installed

```shell
sudo apt-get install iptables
```

Then a script for applying the reules will be created (p.e. `firewall.sh`)

```shell
#! /bin/sh
# /etc/init.d/firewall.sh

### Required-Stop:     $remote_fs $syslog
# Should-Start:      $portmap
# Should-Stop:       $portmap
# X-Start-Before:    nis
# X-Stop-After:      nis
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# X-Interactive:     true
# Short-Description: firewall
# Description:       This file contains a IPtables rules for a firewall.
### END INIT INFO

echo Start applying firewall rules...

## Rules FLUSH
iptables -F
iptables -X
iptables -Z
iptables -t nat -F

## DEFAULT politics
iptables -P INPUT DROP
iptables -P OUTPUT DROP 
iptables -P FORWARD DROP

## LOCALHOST
iptables -A INPUT -s localhost -j ACCEPT
iptables -A OUTPUT -d localhost -j ACCEPT

## WEB Browser
## HTTP
iptables -A INPUT -p tcp --sport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT

## SSH SERVER
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

## HTTPS
iptables -A INPUT -p tcp --sport 443 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT

## FTP SERVER
#iptables -A PREROUTING -t raw -p tcp --dport 21 -j CT --helper ftp
#iptables -A INPUT -p tcp --dport 21 -m conntrack --ctstate ESTABLISHED,NEW -j ACCEPT
#iptables -A OUTPUT -p tcp --sport 21 -m conntrack --ctstate ESTABLISHED,NEW -j ACCEPT
#iptables -A INPUT -p tcp --dport 20 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
#iptables -A OUTPUT -p tcp --sport 20 -m conntrack --ctstate ESTABLISHED -j ACCEPT
#iptables -A INPUT -p tcp --dport 10000:34607 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
#iptables -A OUTPUT -p tcp --sport 10000:34607 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
#iptables -A INPUT -p tcp --dport 34609: -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
#iptables -A OUTPUT -p tcp --sport 34609: -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

## DNS Server
iptables -A INPUT -p udp --sport 53 -j ACCEPT
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT

## DDNS
## no-ip (traquinedes.no-ip.biz)
iptables -A INPUT -p tcp --sport 8245 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 8245 -j ACCEPT

## NTP Server
iptables -A INPUT -p udp --sport 123 -j ACCEPT
iptables -A OUTPUT -p udp --dport 123 -j ACCEPT

## AMULED
## AMULED WEB
iptables -A INPUT -p tcp --dport 4711 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 4711 -j ACCEPT

## AMULED
## DATA
iptables -A INPUT -p tcp --sport 3456 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 3456 -j ACCEPT
iptables -A INPUT -p tcp --dport 3456 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 3456 -j ACCEPT
iptables -A INPUT -p udp --sport 3459 -j ACCEPT
iptables -A OUTPUT -p udp --dport 3459 -j ACCEPT
iptables -A INPUT -p udp --dport 3459 -j ACCEPT
iptables -A OUTPUT -p udp --sport 3459 -j ACCEPT
iptables -A INPUT -p udp --dport 34608 -j ACCEPT
iptables -A OUTPUT -p udp --sport 34608 -j ACCEPT

## AMULED
## SERVERS
# GrupoTS Server
iptables -A OUTPUT -p tcp -d 46.105.126.71 --dport 4661 -j ACCEPT
iptables -A INPUT -p tcp -s 46.105.126.71 --sport 4661 -j ACCEPT
# Sharing-Devils No.1
iptables -A OUTPUT -p tcp -d 91.208.184.143 --dport 4232 -j ACCEPT
iptables -A INPUT -p tcp -s 91.208.184.143 --sport 4232 -j ACCEPT
# Da Na Hui
iptables -A OUTPUT -p tcp -d 103.73.64.146 --dport 51013 -j ACCEPT
iptables -A INPUT -p tcp -s 103.73.64.146 --sport 51013 -j ACCEPT
#
iptables -A OUTPUT -p tcp -d 5.45.85.226 --dport 6584 -j ACCEPT
iptables -A OUTPUT -p tcp -s 5.45.85.226 --sport 6584 -j ACCEPT
# eMule Security
iptables -A OUTPUT -p tcp -d 80.208.228.241 --dport 8369 -j ACCEPT
iptables -A INPUT -p tcp -s 80.208.228.241 --sport 8369 -j ACCEPT
# Astra-1
iptables -A OUTPUT -p tcp -d 103.73.66.116 --dport 38407 -j ACCEPT
iptables -A INPUT -p tcp -s 103.73.66.116 --sport 38407 -j ACCEPT
# Astra-3
iptables -A OUTPUT -p tcp -d 213.252.245.239 --dport 43333 -j ACCEPT
iptables -A INPUT -p tcp -s 213.252.245.239 --sport 43333 -j ACCEPT
# PEERATES.NET
iptables -A OUTPUT -p tcp -d 62.210.28.77 --dport 7111 -j ACCEPT
iptables -A INPUT -p tcp -s 62.210.28.77 --sport 7111 -j ACCEPT
# Astra-2
iptables -A OUTPUT -p tcp -d 95.217.134.86 --dport 22888 -j ACCEPT
iptables -A INPUT -p tcp -s 95.217.134.86 --sport 22888 -j ACCEPT

## SAMBA SERVER
iptables -A INPUT -p tcp --dport 137:139 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 137:139 -j ACCEPT


## Rules CHECKING
iptables -L -n
```

Then add execution permission and run the script

```shell
sudo chmod +x firewall.sh 
./firewall.sh
```

Then, the package `iptables-persistent` will be installed to make the rules persistent between reboots

```shell
sudo apt install iptables-persistent
```

The rules should be saved from the root account

```shell
sudo -i
```

As root use the `iptables-save` command to save the rules in the file `/etc/iptables/rules.v4`

```shell
sudo iptables-save > /etc/iptables/rules.v4
```

## Dynamic DNS

First the `ddclient` package and dependencies should be installed

```shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install libio-socket-ssl-perl
sudo apt-get install ddclient
```

During the installation, follow the setup instruction. After that, all the ddns settings will be stored at `/etc/ddclient.conf`

```console
pi@raspberrypi5:~ $ sudo cat /etc/ddclient.conf 
# Configuration file for ddclient generated by debconf
#
# /etc/ddclient.conf

protocol=noip \
use=web, web=noip-ipv4 \
login=USER \
password='PASSWORD' \
USER_DNS_DOMAIN
```

At last, the ddclient service can be checked

```console
pi@raspberrypi5:~ $ sudo service --status-all
 [ - ]  alsa-utils
 [ - ]  apparmor
 [ + ]  bluetooth
 [ - ]  console-setup.sh
 [ + ]  cron
 [ + ]  dbus
 [ + ]  ddclient
 [ + ]  dphys-swapfile
 [ + ]  exim4
 [ + ]  fake-hwclock
 [ - ]  hwclock.sh
 [ - ]  keyboard-setup.sh
 [ + ]  kmod
 [ - ]  mdadm
 [ - ]  mdadm-waitidle
 [ + ]  netfilter-persistent
 [ + ]  networking
 [ - ]  nfs-common
 [ + ]  procps
 [ + ]  raspi-config
 [ - ]  rpcbind
 [ - ]  rsync
 [ + ]  ssh
 [ - ]  sudo
 [ + ]  triggerhappy
 [ + ]  udev
```

## RAID

The package `mdadm` will be used to create a RAID 1. First, `mdadm` should be installed

```shell
sudo apt-get update
sudo apt-get install mdadm
```

### Move RAID 1 from another machine

The `mdadm` configuration file should be edited. First a backup of the file is created just in case...

```console
pi@raspberrypi5:~ $ sudo cp /etc/mdadm/mdadm.conf /etc/mdadm/mdadm.conf.back
pi@raspberrypi5:~ $ ls -l /etc/mdadm/
total 8
-rw-r--r-- 1 root root 688 mar 19 12:47 mdadm.conf
-rw-r--r-- 1 root root 688 mar 19 12:58 mdadm.conf.back
pi@raspberrypi5:~ $ sudo vi /etc/mdadm/mdadm.conf
```

Debian standard permissions will be enabled.

```
# auto-create devices with Debian standard permissions
CREATE owner=root group=disk mode=0660 auto=yes
```

The data of the RAID should be added to the file. Remember that it should be done from the root account.

```shell
sudo -i
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
```

After that, a new line should appear at `/etc/mdadm/mdadm.conf`

```
# This configuration was auto-generated on Tue, 19 Mar 2024 12:47:05 +0100 by mkconf
ARRAY /dev/md/raspberrypi:0 metadata=1.2 name=raspberrypi:0 UUID=198a49ea:75b71465:6c4aac64:27207dfe
```

The new raid details can be checked

```console
pi@raspberrypi5:~ $ sudo mdadm -D /dev/md/raspberrypi\:0
/dev/md/raspberrypi:0:
           Version : 1.2
     Creation Time : Sat May 19 08:35:23 2018
        Raid Level : raid1
        Array Size : 1953383488 (1862.89 GiB 2000.26 GB)
     Used Dev Size : 1953383488 (1862.89 GiB 2000.26 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Tue Mar 19 15:02:53 2024
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : bitmap

              Name : raspberrypi:0
              UUID : 198a49ea:75b71465:6c4aac64:27207dfe
            Events : 539471

    Number   Major   Minor   RaidDevice State
       3       8        0        0      active sync   /dev/sda
       1       8       16        1      active sync   /dev/sdb
```

Now the name of the RAID ARRAY should be edited at `/etc/mdadm/mdadm.conf` changing it from `/dev/md/raspberrypi:0`  to `/dev/md0`

```
# This configuration was auto-generated on Tue, 19 Mar 2024 12:47:05 +0100 by mkconf
ARRAY /dev/md0 metadata=1.2 name=raspberrypi:0 UUID=198a49ea:75b71465:6c4aac64:27207dfe
```

Now, the new RAID should be added to `/etc/fstab` so its mounted automatically on boot:

```
/dev/md0		/media/HardDisk ext4    defaults        0       0
```

To mount the `fstab` devices

```shell
sudo mount -a
```



