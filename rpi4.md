# Raspberry Pi 4 setup

As a main lab server we use [Raspberry Pi 4 Model B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)
8G RAM version with [aluminium heatsink case](https://shop.pimoroni.com/products/aluminium-heatsink-case-for-raspberry-pi-4?variant=29430673178707)
for passive cooling.

![RPi 4 and USB flash](img/rpi-fit.png?raw=true "RPi 4 and USB flash")

One (bootable) Samsung FIT Plus USB 3.1 flash drive is used for system,
and another one for data storage.

At the time of writting Raspberry Pi OS 64 bit version was still in BETA.

```
$ wget -c https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2020-08-24/2020-08-20-raspios-buster-arm64-lite.zip
$ unzip 2020-08-20-raspios-buster-arm64-lite.zip
$ sudo dd if=2020-08-20-raspios-buster-arm64-lite.img of=/dev/sdb status=progress
```
