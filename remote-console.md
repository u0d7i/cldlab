# Remote Serial Console

Enable serial console on RPi:
```text
$ sudo raspi-config
```
"3 Interface Options" -> "P6 Serial Port" -> "Yes"

Connect RPI GPIO pins to USB TTL Serial Cable, and plug it to the OpenWrt router first USB port:

![USB to TTL Cable](img/usbttl-cable.png?raw=true "USB to TTL Cable")
![Serial GPIO](img/gpiottl.png?raw=true "Serial GPIO")

Connect Cisco serial console cable to USB RS232 serial adapter, and plug it to the OpenWrt router second USB port:

![Cisco Serial Cable](img/cisco-cable.png?raw=true "Cisco Serial Cable")
![USB Serial](img/usbserial.png?raw=true "USB Serial")


## OpenWrt USB Over IP

via https://openwrt.org/docs/guide-user/services/usb.iptunnel

```text
root@gw:~# lsusb
-ash: lsusb: not found

root@gw:~# opkg update
root@gw:~# opkg install usbutils


root@gw:~# lsusb | sort
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

root@gw:~# lsusb | sort
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 002: ID 067b:2303 Prolific Technology, Inc. PL2303 Serial Port

root@gw:~# opkg install usbip-server usbip-client

root@gw:~# usbip list -l
\<no output\>
```

It's a bug,
see https://forum.openwrt.org/t/usb-over-ip-problems/7589/12
fixed in https://github.com/openwrt/packages/issues/12844

```text
root@gw:~# wget -c http://downloads.openwrt.org/releases/17.01.7/packages/mips_24kc/packages/libudev_3.2-1_mips_24kc.ipk

root@gw:~# opkg remove --force-depends libudev-fbsd
root@gw:~# opkg install libudev_3.2-1_mips_24kc.ipk

root@gw:~# usbip list -l
 - busid 1-1 (0403:6001)
   Future Technology Devices International, Ltd : FT232 Serial (UART) IC (0403:6001)

 - busid 2-1 (067b:2303)
   Prolific Technology, Inc. : PL2303 Serial Port (067b:2303)

root@gw:~# usbipd -D
root@gw:~# usbip bind -b 1-1
root@gw:~# usbip bind -b 2-1
```

To make it permanent, add these lines to /etc/rc.local
```text
/usr/sbin/usbipd -D &
sleep 1
/usr/sbin/usbip bind -b 1-1
/usr/sbin/usbip bind -b 2-1
```

On remote machine (gw is a hostname of the OpenWrt above, you may need to use IP address instead):

```text
$ sudo apt install usbip


$ sudo usbip list -r gw
Exportable USB devices
======================
 - gw
        2-1: Prolific Technology, Inc. : PL2303 Serial Port (067b:2303)
           : /sys/devices/platform/ehci-platform.1/usb2/2-1
           : (Defined at Interface level) (00/00/00)

        1-1: Future Technology Devices International, Ltd : FT232 Serial (UART) IC (0403:6001)
           : /sys/devices/platform/ehci-platform.0/usb1/1-1
           : (Defined at Interface level) (00/00/00)


$ sudo modprobe vhci_hcd
$ sudo usbip attach -r gw -b 1-1
$ sudo usbip attach -r gw -b 2-1
```

Dmesg shows (relevant lines):
```text
 vhci_hcd vhci_hcd.0: pdev(0) rhport(0) sockfd(3)
 vhci_hcd vhci_hcd.0: devid(65538) speed(2) speed_str(full-speed)
 usb 3-1: New USB device found, idVendor=0403, idProduct=6001, bcdDevice= 4.00
 usb 3-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
 usb 3-1: Product: usb serial converter
 usb 3-1: Manufacturer: fidi
 ftdi_sio 3-1:1.0: FTDI USB Serial Device converter detected
 usb 3-1: Detected FT232BM
 usb 3-1: FTDI USB Serial Device converter now attached to ttyUSB0

 vhci_hcd vhci_hcd.0: pdev(0) rhport(1) sockfd(3)
 vhci_hcd vhci_hcd.0: devid(131074) speed(2) speed_str(full-speed)
 usb 3-2: New USB device found, idVendor=067b, idProduct=2303, bcdDevice= 3.00
 usb 3-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
 usb 3-2: Product: USB-Serial Controller
 usb 3-2: Manufacturer: Prolific Technology Inc.
 pl2303 3-2:1.0: pl2303 converter detected
 3-2: pl2303 converter now attached to ttyUSB1

$ sudo usbip port
Imported USB devices
====================
Port 00: <Port in Use> at Full Speed(12Mbps)
       Future Technology Devices International, Ltd : FT232 Serial (UART) IC (0403:6001)
       3-1 -> usbip://gw:3240/1-1
           -> remote bus/dev 001/002
Port 01: <Port in Use> at Full Speed(12Mbps)
       Prolific Technology, Inc. : PL2303 Serial Port (067b:2303)
       3-2 -> usbip://gw:3240/2-1
           -> remote bus/dev 002/002
```

Detaching device on remote machine:
```text
$ sudo usbip detach -p 00
usbip: info: Port 0 is now detached!
```

Unbinding devices on gw:
```text
root@gw:~# usbip unbind -b 1-1
usbip: info: unbind device on busid 1-1: complete
```

![Final setup](img/remote-console.png?raw=true "Final setup"
