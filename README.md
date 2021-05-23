# BeagleBone Black
* <https://beagleboard.org/black>
* Debian image version: `AM3358 Debian 10.3 2020-04-06 4GB SD IoT`
* Pinout: <https://beagleboard.org/Support/bone101#headers-black>

Set time manually and then get time via NTP, otherwise `sudo apt-get update`, `sudo apt-get install ...` etc. will not work:
```sh
sudo apt-get update
# Get:1 http://deb.debian.org/debian buster InRelease [121 kB]
# Get:2 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
# Get:3 http://deb.debian.org/debian-security buster/updates InRelease [65.4 kB]
# Get:4 http://repos.rcn-ee.com/debian buster InRelease [3,078 B]
# Reading package lists... Done
# E: Release file for http://deb.debian.org/debian/dists/buster/InRelease is not valid yet (invalid for another 354d 17h 8min 40s). Updates for this repository will not be applied.

# Set time manually
sudo date -s "18 MAY 2021 20:00:00"

# Update packages
sudo apt-get update && sudo apt-get dist-upgrade

# Install ntpd
sudo apt-get install ntp

# Get time from NTP server
sudo service ntp stop && sudo ntpd -gq && sudo service ntp start
```

## Wifi
Add WLAN access point (does not work without disabling Wifi first):
```sh
sudo su -
connmanctl
>tether wifi off
>disable wifi
>enable wifi
>scan wifi # Scan for available access points
>services # List available access points
>agent on # Save connection infos
>connect wifi_....._managed_psk
>Passphrase? # Enter passphrase (will be saved)
>quit
```

Wifi is not automatically reconnected after rebooting.
Fix for re-enabling Wifi:

`/etc/rc.local`:
```sh
#!/bin/sh

/root/connect-wifi.sh &

exit 0
```

`root/connect-wifi.sh`:
```sh
#!/bin/sh

sleep 60
connmanctl disable wifi
connmanctl enable wifi
```

## CAN

CAN transceiver SN65HVD230: <https://www.ti.com/lit/ds/symlink/sn65hvd230.pdf>

CAN0 interface (shared with I2C interface):
* DCAN0_RX: header P9, pin 19
* DCAN0_TX: header P9, pin 20

CAN1 interface:
* DCAN1_RX: header P9, pin 24
* DCAN1_TX: header P9, pin 26

Test CAN1 interface (from <https://www.beyondlogic.org/adding-can-to-the-beaglebone-black/>, run as root):
```sh
# Configure pins 24 and 26 as CAN1 (on header P9)
config-pin p9.24 can
config-pin p9.26 can

# Current mode for P9_24 is:     can
# Current mode for P9_26 is:     can

# Bring up can1 interface with 500 kbit/s
ip link set can1 up type can bitrate 500000

ip addr show dev can1

#3: can1: <NOARP,UP,LOWER_UP,ECHO> mtu 16 qdisc pfifo_fast state UP group default qlen 10
#    link/can
```

CAN-utils:
```sh
# Show load on CAN bus
canbusload can1@500000

# can1@500000     0       0      0   0%
#
# can1@500000     0       0      0   0%

# Generate random CAN messages
cangen can1

# Display CAN messages
candump can1

# Send CAN message with ID 0x123 and 8 bytes of data
cansend can1 123#11.22.33.44.55.66.77.88
```

Bring up CAN interface automatically at startup (from <https://www.beyondlogic.org/adding-can-to-the-beaglebone-black/>, <https://www.thomas-wedemeyer.de/beaglebone-canbus-python.html>, run as root):
```sh
# Load device tree overlay for DCAN1
echo -e '\n# Configure pins 24, 26 as CAN1\nuboot_overlay_addr4=/lib/firmware/BB-CAN1-00A0.dtbo' >>/boot/uEnv.txt

# Bring up can0 (DCAN1 interface) at startup
echo -e '\n# Bring up can1 (DCAN1 interface) with 500 kbit/s\nallow-hotplug can1\niface can1 can static\n\tbitrate 500000' >>/etc/network/interfaces

# Check if can1 device is up
ip addr show dev can1

#3: can1: <NOARP,UP,LOWER_UP,ECHO> mtu 16 qdisc pfifo_fast state UP group default qlen 10
#    link/can
```

# SocketCAN
* Linux Kernel documentation for SocketCAN: <https://www.kernel.org/doc/Documentation/networking/can.txt>

# DUINOMITE-MEGA
* <https://www.olimex.com/Products/Duino/Duinomite/DUINOMITE-MEGA/open-source-hardware>
* CAN bus demos (in BASIC): <https://github.com/OLIMEX/DuinoMite/tree/master/SOFTWARE/DMBasic/src/CANdocs/doc>
