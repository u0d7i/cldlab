# Raspberry Pi 4 setup

As a main lab server we use [Raspberry Pi 4 Model B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
8G RAM version with [aluminium heatsink case](https://shop.pimoroni.com/products/aluminium-heatsink-case-for-raspberry-pi-4?variant=29430673178707)
for passive cooling.

![RPi 4 and USB flash](img/rpi-fit.png?raw=true "RPi 4 and USB flash")

One (bootable) Samsung FIT Plus USB 3.1 flash drive is used for system,
and another one for data storage.

## USB Boot

To be able to boot from USB Mass Storage Devices, Raspberry Pi 4B bootloader EEPROM version
should be Sep 3 2020 or later, and Raspberry Pi OS 2020-08-20 or later.
[Check documentation](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/msd.md) for details.


```
pi@raspberrypi:~ $ vcgencmd bootloader_version
May 10 2019 19:40:36
version d2402c53cdeb0f072ff05d52987b1b6b6d474691 (release)
timestamp 0
update-time 0
capabilities 0x00000000

pi@raspberrypi:~ $ sudo apt update
pi@raspberrypi:~ $ sudo apt full-upgrade
pi@raspberrypi:~ $ sudo reboot

pi@raspberrypi:~ $ ls -al /lib/firmware/raspberrypi/bootloader/critical

pi@raspberrypi:~ $ sudo rpi-eeprom-update
BCM2711 detected
Dedicated VL805 EEPROM detected
*** UPDATE AVAILABLE ***
BOOTLOADER: update available
CURRENT: Thu  1 Jan 00:00:00 UTC 1970 (0)
 LATEST: Thu  3 Sep 12:11:43 UTC 2020 (1599135103)
 FW DIR: /lib/firmware/raspberrypi/bootloader/default
VL805: update available
CURRENT: 00013701
 LATEST: 000138a1

pi@raspberrypi:~ $ sudo rpi-eeprom-update -a
pi@raspberrypi:~ $ sudo reboot

pi@raspberrypi:~ $ uname -a
Linux raspberrypi 5.4.83-v7l+ #1379 SMP Mon Dec 14 13:11:54 GMT 2020 armv7l GNU/Linux

pi@raspberrypi:~ $ vcgencmd bootloader_version
Sep  3 2020 13:11:43
version c305221a6d7e532693cc7ff57fddfc8649def167 (release)
timestamp 1599135103

```

## Raspberry Pi OS 64 bit

Default Raspbian / Raspberry Pi OS images are 32bit armhf (armv7l).

At the time of writting, Raspberry Pi OS 64bit (arm64) version [was still in BETA](https://www.raspberrypi.org/forums/viewtopic.php?t=275370).

```
$ wget -c https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2020-08-24/2020-08-20-raspios-buster-arm64-lite.zip
$ unzip 2020-08-20-raspios-buster-arm64-lite.zip
$ sudo dd if=2020-08-20-raspios-buster-arm64-lite.img of=/dev/sdb status=progress
```

```
$ sudo mount /dev/sdb1 /mnt/
$ sudo touch /mnt/ssh
$ sudo umount /mnt
```

Boot from USB, after `apt full-upgrade`:
```
pi@raspberrypi:~ $ uname -a
Linux raspberrypi 5.10.5-v8+ #1392 SMP PREEMPT Sat Jan 9 18:56:30 GMT 2021 aarch64 GNU/Linux
```

Note "aarch64" architecture instead of "armv7l".

## Initial setup

```
$ sudo passwd pi
$ sudo adduser user
$ sudo sed -i 's/:pi/:pi,user/' /etc/group
$ sudo sed -i 's/^pi/user/' /etc/sudoers.d/010_user-nopasswd
$ echo "cldlab" | sudo tee /etc/hostname
$ sudo sed -i 's/raspberrypi/cldlab/' /etc/hosts
$ sudo hostname cldlab
```

## Data storage

Insert second (data) USB flash drive.

```
$ sudo dd if=/dev/zero of=/dev/sdb bs=512 count=1
$ sudo cfdisk /dev/sdb
$ sudo mkfs.ext4 /dev/sdb1
$ sudo e2label /dev/sdb1 data

$ blkid /dev/sdb1
/dev/sdb1: LABEL="data" UUID="7675.....ce22" TYPE="ext4" PARTUUID="c7fff6e7-01"

$ sudo mkdir /media/data
$ cat /etc/fstab
$ echo "PARTUUID=c7fff6e7-01  /media/data  ext4  noatime,nofail,x-systemd.device-timeout=1ms  0  2" | sudo tee -a /etc/fstab
$ mount -a
$ lsblk
$ lsblk -f
```

## Temperature

We use passive cooling with heatsink case. To see how efficient it is:

- check GPU temperature:
```
$ vcgencmd measure_temp
```

- check CPU temperature:
```
$ cat /sys/class/thermal/thermal_zone0/temp
```
