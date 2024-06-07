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
# eMule Sunrise
iptables -A OUTPUT -p tcp -d 176.123.5.89 --dport 4725 -j ACCEPT
iptables -A INPUT -p tcp -s 176.123.5.89 --sport 4725 -j ACCEPT
# eMule Security
iptables -A OUTPUT -p tcp -d 45.82.80.155 --dport 5687 -j ACCEPT
iptables -A INPUT -p tcp -s 45.82.80.155 --sport 5687 -j ACCEPT
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

### Disabling USB Current Limitation

To be able to connect two USB hard disks directly to the RPi, the USB current limitation should be removed from the RPi using the RPi configuration tool.

```shell
sudo raspi-config
```

Then the USB power limit can be disabled following these menus inside the raspi-config tool: `Performance Options > USB Current >Would you like the USB current limit to be disabled? > Yes`

This way, given the RPi PSU is powerful enough, two USB hard disks can be connected directly to the RPi without using an external powered USB hub in the middle.

### Move RAID 1 from Another Machine

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

## Amuled

First the `amule` and the `amule-daemon` packages should be installed

```shell
sudo apt-get install amule amule-daemon
```

Once installed, run the `amuled` command the first time to create the configuration files at `~/.aMule/`

```console
pi@raspberrypi5:~ $ amuled
 2024-03-20 16:09:22: Initialising aMuleD 2.3.3 compiled with wxBase(GTK2) v3.2.2 and Boost 1.74
 2024-03-20 16:09:22: Checking if there is an instance already running...
 2024-03-20 16:09:22: No other instances are running.
!2024-03-20 16:09:22: ERROR: Información  --- Es la primera vez que inicia aMule 2.3.3 ---
!2024-03-20 16:09:22: En nuestra web podrá encontrar más información, soporte y nuevas versiones,
!2024-03-20 16:09:22: en www.aMule.org, o en nuestro canal de IRC #aMule en irc.freenode.net.
!2024-03-20 16:09:22: Envíenos cualquier fallo a http://forum.amule.org
 2024-03-20 16:09:22: Socket de escucha: correcto.
 2024-03-20 16:09:22: Cargando archivos temporales desde /home/pi/.aMule/Temp.
 2024-03-20 16:09:22: Todos los archivos de partes han sido cargados.
 2024-03-20 16:09:22: amuled: OnInit - iniciando el temporizador
!2024-03-20 16:09:22: ERROR: el servicio de aMule no se puede usar cuando están desactivadas las conexiones externas. Para activarlas, use o bien un aMule normal, inicie amuled con la opción --ec-config, o bien, establezca "AcceptExternalConnections" a 1, en el archivo ~/.aMule/amule.conf
 2024-03-20 16:09:22: Ahora, saliendo de la aplicación principal...
 2024-03-20 16:09:22: aMule OnExit: terminando el núcleo.
 2024-03-20 16:09:22: Cierre de aMule completado.
16:09:22: Debug: 1 threads were not terminated by the application.
```

First set a bandwidth limitation to avoid saturate the connection

```
MaxUpload=10
MaxDownload=50
```

Now the aMule can be configured. First make a copy of the current `amule.conf` file. Now the aMule ports should be changed to random values avoid blocking from some ISPs:

```
Port=4662
UDPPort=4672
```

Also, the download of a server list from a server can be enabled

```
AddServerListFromServer=1
```

Enable connect only to secure servers

```
SafeServerConnect=1
```

Also, the files folders are changed from the normal place, to an external HDD

```
TempDir=/media/HardDisk/aMule/Temp
IncomingDir=/media/HardDisk/aMule/Incoming
MinFreeDiskSpace=3000
```

The server from were the Kad nodes and the Ed2k servers lists should be downloaded

```
KadNodesUrl=http://upd.emule-security.org/nodes.dat
Ed2kServersUrl=http://upd.emule-security.org/server.met
```

Now, the web interface can be configured

```
AcceptExternalConnections=1
ECPassword=c87ac475dba8e4522dbf573a7355dca6
[WebServer]
Enabled=1
Password=c87ac475dba8e4522dbf573a7355dca6
Port=4711
```

To start the amule daemon at boot, an user should be added on the amule-daemon configuration file at `/etc/default/amule-daemon`. To keep it simple the our user will be used.

```console
pi@raspberrypi5:~ $ sudo cat /etc/default/amule-daemon 
# Configuration for /etc/init.d/amule-daemon

# The init.d script will only run if this variable non-empty.
AMULED_USER="pi"

# You can set this variable to make the daemon use an alternative HOME.
# The daemon will use $AMULED_HOME/.aMule as the directory, so if you
# want to have $AMULED_HOME the real root (with an Incoming and Temp
# directories), you can do `ln -s . $AMULED_HOME/.aMule`.
AMULED_HOME=""
```

Now the amule daemon can be started.

```shell
sudo /etc/init.d/amule-daemon start
```

At last, the amule web interface can be accesed at [http://rpi-address:4711/](http://rpi-address:4711/)

## Samba

To share folders with remote PCs, a Samba server will be installed.

```shell
sudo apt install samba samba-common-bin 
```

Then, at the end of the configuration file `/etc/samba/smb.conf` an entry for the folders we want to share will be added. I.e. share the RPi folder.

```
# Amule downloads folder
[aMule]
    comment = Share Directory
    path = /media/HardDisk/aMule/Incoming/
    browseable = Yes
    writeable = no
    only guest = no
    create mask = 0644
    directory mask = 0755
    public = yes
```

Then the `smbd` daemon should be restarted to load the new configuration.

```shell
sudo service smbd restart
```

After that the `testparm` command can be used to check the Samba server services:

```console
pi@raspberrypi5:~ $ testparm 
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed by GnuTLS (e.g. NTLM as a compatibility fallback)

Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
	log file = /var/log/samba/log.%m
	logging = file
	map to guest = Bad User
	max log size = 1000
	obey pam restrictions = Yes
	pam password change = Yes
	panic action = /usr/share/samba/panic-action %d
	passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
	passwd program = /usr/bin/passwd %u
	server role = standalone server
	unix password sync = Yes
	usershare allow guests = Yes
	idmap config * : backend = tdb


[homes]
	browseable = No
	comment = Home Directories
	create mask = 0700
	directory mask = 0700
	valid users = %S


[printers]
	browseable = No
	comment = All Printers
	create mask = 0700
	path = /var/tmp
	printable = Yes


[print$]
	comment = Printer Drivers
	path = /var/lib/samba/printers


[aMule]
	comment = Share Directory
	create mask = 0644
	guest ok = Yes
	path = /media/HardDisk/aMule/Incoming/
```

Then, to access the samba folder from remote clients, the address should be used is:

`smb://<samba_ip_address>/<samba_folder_name>`

**THIS SAMBA SETUP IS NOT SECURE, MORE STEPS SHOULD BE TAKEN TO SECURE IT**

## Disable WiFi Power Save

In some distributions, the power save feature is enabled for the WiFi interface:

```console
pi@raspberrypi5:~ $ iw wlan0 get power_save
Power save: on
```

To avoid the WiFi getting disconnected every time, the power saving feature should be disabled:

```console
pi@raspberrypi5:~ $ iwconfig wlan0 power off
pi@raspberrypi5:~ $ iw wlan0 get power_save
Power save: off
```

To persist this setting after reboot, the disabling command can be added to `/etc/rc.local` to disable it each time the RPi starts. The disabling command is:

```bash
/sbin/iwconfig wlan0 power off
```

And the `/etc/rc.local` file will look like:

```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

/sbin/iwconfig wlan0 power off

exit 0
```