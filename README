# Integrating LAN969x 24-port EVB to Raspberry Pi OS

> Note: This document is similar to [Integrating LAN966x to Raspberry Pi OS](https://support.microchip.com/s/article/Integrating-LAN966x-to-Raspberry-Pi-OS)

This document describes the process of integrating the **LAN969x PCIe network switch driver** into the standard Raspberry Pi OS, running on a Raspberry Pi Compute Module 4 (CM4). This build bridges the gap by compiling Microchip's Linux kernel fork with the necessary drivers and configurations, allowing users to run a full-featured Raspberry Pi OS alongside the LAN969x switch hardware.

While Microchip provides a basic Board Support Package (BSP), it lacks the full richness of the standard Raspberry Pi OS ecosystem, things like package management, and familiar tooling.

This project solves that by using the standard 64-bit Raspberry Pi OS as the base, then builds and installs a custom kernel derived from Microchip's Linux fork (v6.12).
## General Idea

The core idea is to **replace the stock Raspberry Pi kernel with a custom-built one that includes LAN969x PCIe support**, while keeping everything else about the Raspberry Pi OS intact. The process flows like this:

The build starts by cloning Microchip's Linux kernel repository and checking out the v6.12 branch. Rather than building from scratch, it borrows the `bcm2711_defconfig` (the standard RPi 4 kernel config) from the official Raspberry Pi kernel fork as a baseline. On top of that, a set of additional kernel config options are enabled, primarily the LAN969x switch driver, DSA (Distributed Switch Architecture) framework, SerDes PHY, and various I2C and pinctrl components that the switch depends on.

Device tree sources and overlays are similarly pulled from the Raspberry Pi fork to ensure proper hardware description at boot, and a custom device tree (`bcm2711-rpi-cm4-lan969x.dtb`) is used to describe the combined CM4 + LAN969x hardware. A minor patch is also applied to fix the `ranges` property of the pcie node.

Once built, the kernel image, modules, and device tree blobs are installed onto the RPi boot media, and `config.txt` is updated to point to the new kernel and device tree. After booting, the LAN969x switch enumerates over PCIe and its ports appear as standard Linux network interfaces (swp1`–`swp29), fully manageable with familiar tools like `ip` and `ethtool`.

---
## Hardware
* [LAN969x 24-port EVB](https://www.microchip.com/en-us/development-tool/ev23x71a)
* Raspberry Pi Compute Module 4 (CM4)
* Raspberry Pi IO Board
* Oculink Cable
* Oculink PCIe X1 to SFF-8612

| ![RPi CM4 and IO Board](raspi-cm4-1.png)                 | ![](234870644e6d625d91fdaa40bb68.l.png)         |
| -------------------------------------------------------- | ----------------------------------------------- |
| ![Oculink PCIe X1 to SFF-8612](oculink-pcie-sff.png)<br> | ![Oculink Cable](oculink-cable.png) |

Overall set up:
![overall-set-up](lan969x-pcie-rpicm4.jpg)

---
## **Configure LAN969x-24port EVB as PCIe Endpoint:**
The LAN969x-24port EVB must be setup to run as a PCIe endpoint. On this page,[https://github.com/microchip-ung/arm-trusted-firmware/releases/tag/v2.8.17-mchp0](https://github.com/microchip-ung/arm-trusted-firmware/releases/tag/v2.8.17-mchp0) download:
* Download a U-Boot version that setup the PCIe endpoint [lan969x_pcie-release.fip.gz](https://github.com/microchip-ung/arm-trusted-firmware/releases/download/v2.8.17-mchp0/lan969x_pcie-release.fip.gz))
* Download the tool to program the above U-Boot image to the EVB-LAN969x [fwu-lan969x_a0-release.html](https://github.com/microchip-ung/arm-trusted-firmware/releases/download/v2.8.17-mchp0/fwu-lan969x_a0-release.html))

1. On the LAN969x board set the **VCORE[3..0]=1010** and power cycle the board. 
2. Connect a USB-C to the Console of LAN969x board.
3. Open **`fwu-lan969x_a0-release.html`** with [Google Chrome](https://www.google.com/intl/en_ph/chrome/) (with javescript enabled (`CTRL+SHIFT+i`))
4. Click "Connect" MCP2200 USB Serial Port Emulator (COMx) with 115200 baud. Transaction log should show:
```
Connected
Identify platform...
Version: BL1:v2.6(release):lan969x-a0-bl1-rc2
Identified device: LAN969X A0
Please select a BL1 command - or upload BL2U for firmware update functions
```
5. Click "Download BL2U" 
6. Upload the **`lan969x_pcie-release.fip.gz`** in the "Upload" Tab. Log should show:
```
Reading file: lan969x_pcie-release.fip.gz
DDR initialized and cache enabled
Downloading 89034 bytes binary
Download took: 9 seconds, 671 milliseconds
Download was completed
Calculating download data hash
89034 bytes in write buffer
SHA-256 hash: bd33d3a383b61e83ebfba1eae0068fff8d9adfc5270624a2cc859f7454029785
Data integrity check pass, SHA-256 hash match
Decompressing data
Decompressing took: 14 milliseconds
Calculating download data hash
179712 bytes in write buffer
SHA-256 hash: fd1904c2fa58990b4ae19432bcf6c283d55f49e306665b991a24834a6334bae6
```
7. Write **`lan969x_pcie-release.fip.gz`** in the tab “**Write**” by changing to NOR in the line with the text "Write Flash Image". Then press "Write Flash Image" button.
	- ![](Pasted-image-20260330092249.png)
	- Log should show:
```
Starting Write Image to NOR Flash
This may take up to 5 minutes or even longer depending on data size and media.
Write Image to NOR Flash took: 1 second, 724 milliseconds
Image written and verified
```
8. When done writing the image, change the VCORE[3..0]=1000, so that the device boot from NOR when booting.
9. Power cycle the board to bring up the PCIe. Check by closing the chrome and starting teraterm and connecting to the LAN969x board. When Power cycling it will write a short list of lines to the terminal ending with "`NOTICE: pcie: Config EP`".
---
## Install Raspberry Pi OS to the Boot Media (SD Card):
Install the Raspberry Pi OS 64-bit to the Boot Media (SD Card) by following [Install using Imager](https://www.raspberrypi.com/documentation/computers/getting-started.html#raspberry-pi-imager). Use the following options

* _Raspberry Pi Device:_ **Raspberry Pi 4**
* _Operating System:_ **Raspberry Pi OS (64-bit)**
* [`2025-12-04-raspios-trixie-arm64.img`](https://downloads.raspberrypi.com/raspios_arm64/images/raspios_arm64-2025-12-04/2025-12-04-raspios-trixie-arm64.img.xz)

**Boot-up the Raspberry Pi OS as per usual**: 
* Set up credentials
* Set up configurations
* Set up time and date
* Update using `sudo apt update && sudo apt upgrade -y`

Once done, remove the Boot Media.
___
## **Build the Microchip Linux Kernel:**
### Build Environment:
+ Ubuntu Linux 24.04 LTS
+ **Target Kernel Version v6.12.48-v8+**

We will need a Linux cross-compilation host. We will use **Ubuntu Linux 24.04 LTS** with the following packages installed.
```shell
sudo apt install bc bison flex libssl-dev make libc6-dev libncurses5-dev crossbuild-essential-arm64
```

The Linux Kernel we will use is from the Microchip's Fork. The kernel version we'll be using is `v6.12`--the latest version as of this writing. We will also use the `bcm2711_defconfig` from Raspberry Pi's Fork as the base config.
### 1. Prepare the Linux-UNG Repository
```shell
git clone https://github.com/microchip-ung/linux linux-ung
```
Checkout the Kernel Version 6.12
```shell
cd linux-ung
git checkout bsp-6.12-2025.12
```
### 2. Prepare the `defconfig` from the Raspberry Pi Repository
Add the Raspberry Pi remote and fetch the `rpi-6.12.y`
Overwrite `bcm2711_defconfig` file from Raspberry Pi Repository to `arch/arm64/configs/`
```shell
# Add the raspberrypi remote
git remote add rpi https://github.com/raspberrypi/linux.git

# Fetch only the specific branch (no need to fetch everything)
git fetch rpi rpi-6.12.y

# Copy/overwrite the configs from that branch into the working tree
git checkout rpi/rpi-6.12.y -- arch/arm64/configs/bcm2711_defconfig
```
> **Note:** The version of the defconfig should match with our target version.
### 3. Build Configuration
```shell
# for 64-bit
KERNEL=kernel8
make -j12 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
```
> **Note:** The `bcm2711_defconfig` loads the default Linux configuration for the Raspberry Pi  

```shell
make -j12 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

Add the LAN969x drivers and related components in the Linux config:
> Tip: Inside menuconfig, press **`'/'`** key. Type the config, i.e., **`LAN969X_SWITCH`**. It'll display all the config option having this string. Press the number against the desired option. It'll take to that specific hardware configuration. Change configuration with a **`'space'`**, **`'Y'`**, or **`'N'`** key.
```c
# Tick [Y] to Add
<*> The IPv6 protocol                         (IPV6 [=y])
[*] Distributed Switch Architecture           (NET_DSA [=y])
<*> 802.1d Ethernet Bridging                  (BRIDGE [=y])
[*] Data Center Bridging support              (DCB [=y])
[*] ARMv8 based Microchip LAN969X SoC family  (ARCH_LAN969X [=y])
<*> Lan969x switch driver                     (LAN969X_SWITCH [=y])
-*- Microchip Sparx5 SerDes PHY driver        (PHY_SPARX5_SERDES [=y])
<*> Microchip LAN969X Support                 (MFD_LAN969X_PCI [=y])
<*> Userspace I/O drivers  --->
<*>   Generic driver for Sparx5 SoC           (UIO_SPARX5 [=y])
<*>   Generic driver for Sparx5 SoC           (UIO_SPARX5_IRQMUX [=y])
<*> SFP cage support                          (SFP [=y])
[*] Microchip Sparx5 reset driver             (RESET_MCHP_SPARX5 [=y])
<*> Microsemi MIIM interface support          (MDIO_MSCC_MIIM [=y])
{*} I2C bus multiplexing support              (I2C_MUX [=y])
<*> pinctrl-based I2C multiplexer             (I2C_MUX_PINCTRL [=y])
<*> pinctrl-based I2C demultiplexer           (I2C_DEMUX_PINCTRL [=y])
<*> GPIO-based bitbanging I2C                 (I2C_GPIO [=y])
<*> GPIO-based I2C multiplexer                (I2C_MUX_GPIO [=y])
<*> I2C device interface                      (I2C_CHARDEV [=y])
<*> Atmel AT91 I2C Two-Wire interface         (I2C_AT91 [=y])
<*> Microchip AT91 I2C experimental slave mode (I2C_AT91_SLAVE_EXPERIMENTAL [=y])
<*> Broadcom BCM2835 I2C controller           (I2C_BCM2835 [=y])
<*> BRCM Settop/DSL I2C controller            (I2C_BRCMSTB [=y])
[*] Atmel Flexcom                             (MFD_ATMEL_FLEXCOM [=y])
<*> Pinctrl driver for Microchip Serial GPIO  (PINCTRL_MICROCHIP_SGPIO [=y])
<*> Pinctrl driver for the Ocelot and Jaguar2 SoCs (PINCTRL_OCELOT)
<*> High-availability Seamless Redundancy     (HSR [=y])
[*] IP-VLAN support                           (IPVLAN [=y])
# MFD_LAN966X_PCI is needed to build without error
<*> Microchip LAN966X Support                 (MFD_LAN966X_PCI [=y])

# Tick [M] to Module
<M> Credit Based Shaper (CBS)                 (NET_SCH_CBS [=m])
<M> Earliest TxTime First (ETF)               (NET_SCH_ETF [=m])
<M> Time Aware Priority (taprio) Scheduler    (NET_SCH_TAPRIO [=m])
```
Remove the following component in the Linux config:
```c
#Tick [N] to Remove
[ ] Memory-mapped io interface driver for DW SPI core  (SPI_DW_MMIO [=n])
```

Save and Exit
```
<Save>
<Exit>
```
### 4. Prepare the Device-Tree
Copy the device tree sources `dts` and overlays `dtso` from Raspberry Pi Repository:
```shell
# Copy the arm64 dts from Raspberry Pi Repository into the working tree
git checkout rpi/rpi-6.12.y -- arch/arm64/boot/dts/broadcom

# Copy the arm dts from Raspberry Pi Repository into the working tree
git checkout rpi/rpi-6.12.y -- arch/arm/boot/dts/broadcom

# Copy the dtso from Raspberry Pi Repository into the working tree
git checkout rpi/rpi-6.12.y -- arch/arm64/boot/dts/overlays
git checkout rpi/rpi-6.12.y -- arch/arm/boot/dts/overlays

# Copy the Makefile from Raspberry Pi Repository into the working tree
git checkout rpi/rpi-6.12.y -- arch/arm64/boot/dts/Makefile

git checkout rpi/rpi-6.12.y -- scripts/Makefile.lib
git checkout rpi/rpi-6.12.y -- scripts/Makefile.build

```
Copy some dependencies from the `include` to properly build the `dts` and `dtso`
```shell
# Copy the dependencies from Raspberry Pi Repository into the working tree
git checkout rpi/rpi-6.12.y -- include/dt-bindings/clock/rp1.h
git checkout rpi/rpi-6.12.y -- include/dt-bindings/gpio/gpio-fsm.h
git checkout rpi/rpi-6.12.y -- include/dt-bindings/mfd/rp1.h
```

We need to correct the `ranges` property in the PCIe node of the device tree `bcm2711.dtsi`. 
```shell
nano arch/arm/boot/dts/broadcom/bcm2711.dtsi
```
The relevant DT node (/scb/pcie@7d500000) should have its inbound mapping changed from:
```
ranges = <0x02000000 0x0 0xf8000000 0x6 0x00000000
				  0x0 0x04000000>;
```
To:
```
/* Correct - identity map for LAN969x FDMA */
ranges = <0x02000000 0x0 0xc0000000 0x6 0x00000000
				  0x0 0x40000000>;
```

The device tree to use is `bcm2711-rpi-cm4-lan969x`. So edit the `Makefile` to include `bcm2711-rpi-cm4-lan969x` during build:
```shell
nano arch/arm64/boot/dts/broadcom/Makefile
```
Insert `bcm2711-rpi-cm4-lan969x.dtb` on this location like so:
```c
dtb-$(CONFIG_ARCH_BCM2835) += bcm2711-rpi-cm4-lan969x.dtb
```

### 5. Build
Run the following command to build a 64-bit kernel, modules and the device-tree blob:
```shell
make -j12 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```
> The build may take a while so grab a coffee ☕

---

## **Install the Microchip Linux Kernel**
Having built the kernel, copy it onto the Raspberry Pi OS 64-bit Image and install the modules.
### 1. Mount Boot Media
First, run `lsblk`. Then, connect the boot media. Run `lsblk` again; the new device represents the boot media. See output similar to the following:
```shell
$ lsblk
sdc
   sdc1
   sdc2
```

Mount these partitions as `mnt/boot` and `mnt/root`, adjusting the partition letter to match the location of the boot media:
```shell
mkdir mnt
mkdir mnt/boot
mkdir mnt/root
sudo mount /dev/sdc1 mnt/boot
sudo mount /dev/sdc2 mnt/root
```
### 2. Next, install the kernel modules onto the boot media:
```shell
sudo env PATH=$PATH make -j12 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=mnt/root modules_install
```
### 3. Install Kernel onto the boot media:
Run the following commands to create a backup image of the current kernel, install the fresh kernel image, overlays.
```shell
sudo cp mnt/boot/kernel8.img mnt/boot/kernel8-backup.img
sudo cp arch/arm64/boot/Image mnt/boot/kernel8.img
sudo cp -r arch/arm64/boot/dts/broadcom/*.dtb* mnt/boot/
sudo cp -r arch/arm64/boot/dts/overlays/*.dtb* mnt/boot/overlays/
```
Modify `config.txt` to define the new kernel and device-tree at boot
```shell
sudo nano mnt/boot/config.txt
```
Add the following texts at the bottom of `config.txt`, save then exit.
```shell
[all]
#use 64-bit
arm_64bit=1

#boots the newly built kernel
kernel=kernel8.img

#this overlay prevents firmware from extending the PCIe inbound window beyond 32-bit
dtoverlay=pcie-32bit-dma

#target DTB to be loaded
device_tree=bcm2711-rpi-cm4-lan969x.dtb

#Disable OTG mode of the USB
otg_mode=0
```
Save and Exit
```
<Save>
<Exit>
```
Unmount the partitions:
```shell
sudo umount mnt/boot
sudo umount mnt/root
```

Finally, connect the boot media to the Raspberry Pi and connect it to power to run the freshly-compiled kernel.

---
## **Verification**
Boot the Raspberry Pi with the newly modified Boot Media
### Show Linux Version
```shell
uname –a

#It should return the following
Linux raspberrypi 6.12.48-v8+ #42 SMP PREEMPT Fri Mar 27 16:31:28 PST 2026 aarch64 GNU/Linux
```
### Show that the LAN969x driver is loaded and enumerated.
```shell
$ lspci -vv
```
It should return **`Kernel driver in use: microchip_lan969x_pci`**:
```shell
lspci -vv
00:00.0 PCI bridge: Broadcom Inc. and subsidiaries BCM2711 PCIe Bridge (rev 20) (prog-if 00 [Normal decode])
	Device tree node: /sys/firmware/devicetree/base/scb/pcie@7d500000/pci@0,0
	Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Latency: 0
	Interrupt: pin A routed to IRQ 19
	Bus: primary=00, secondary=01, subordinate=01, sec-latency=0
	Memory behind bridge: 00000000-037fffff [size=56M] [32-bit]
	Prefetchable memory behind bridge: [disabled] [64-bit]
	Secondary status: 66MHz- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- <SERR- <PERR-
	BridgeCtl: Parity- SERR- NoISA- VGA- VGA16- MAbort- >Reset- FastB2B-
		PriDiscTmr- SecDiscTmr- DiscTmrStat- DiscTmrSERREn-
	Capabilities: <access denied>
	Kernel driver in use: pcieport

01:00.0 Ethernet controller: Microchip Technology / SMSC Device 9690
	Subsystem: Microchip Technology / SMSC Device 9690
	Device tree node: /sys/firmware/devicetree/base/scb/pcie@7d500000/pci@0,0/ep_lan969x@0,0
	Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx-
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
	Latency: 0
	Interrupt: pin A routed to IRQ 19
	Region 0: Memory at 600000000 (32-bit, non-prefetchable) [size=32M]
	Region 1: Memory at 602000000 (32-bit, non-prefetchable) [size=16M]
	Region 4: Memory at 603000000 (32-bit, non-prefetchable) [size=128K]
	Region 5: Memory at 603020000 (32-bit, non-prefetchable) [size=8K]
	Capabilities: <access denied>
	Kernel driver in use: microchip_lan969x_pci

```
IP Command
```shell
$ ip a
```
Returns:
```shell
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether d8:3a:dd:db:e2:99 brd ff:ff:ff:ff:ff:ff
    inet 10.161.140.107/24 brd 10.161.140.255 scope global dynamic noprefixroute eth0
       valid_lft 4733sec preferred_lft 4733sec
    inet6 fe80::9b7e:b3b8:2ce4:f0f7/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: swp0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:01 brd ff:ff:ff:ff:ff:ff
4: swp1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether f2:82:a1:e7:be:02 brd ff:ff:ff:ff:ff:ff
    inet 10.0.1.10/24 brd 10.0.1.255 scope global noprefixroute swp1
       valid_lft forever preferred_lft forever
    inet6 fe80::8e2a:5ffb:113b:fe97/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
5: swp2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:03 brd ff:ff:ff:ff:ff:ff
6: swp3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:04 brd ff:ff:ff:ff:ff:ff
7: swp4: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:05 brd ff:ff:ff:ff:ff:ff
8: swp5: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:06 brd ff:ff:ff:ff:ff:ff
9: swp6: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:07 brd ff:ff:ff:ff:ff:ff
10: swp7: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:08 brd ff:ff:ff:ff:ff:ff
11: swp8: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:09 brd ff:ff:ff:ff:ff:ff
12: swp9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:0a brd ff:ff:ff:ff:ff:ff
13: swp10: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:0b brd ff:ff:ff:ff:ff:ff
14: swp11: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:0c brd ff:ff:ff:ff:ff:ff
15: swp12: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:0d brd ff:ff:ff:ff:ff:ff
16: swp13: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:0e brd ff:ff:ff:ff:ff:ff
17: swp14: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:0f brd ff:ff:ff:ff:ff:ff
18: swp15: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:10 brd ff:ff:ff:ff:ff:ff
19: swp16: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:11 brd ff:ff:ff:ff:ff:ff
20: swp17: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:12 brd ff:ff:ff:ff:ff:ff
21: swp18: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:13 brd ff:ff:ff:ff:ff:ff
22: swp19: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:14 brd ff:ff:ff:ff:ff:ff
23: swp20: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:15 brd ff:ff:ff:ff:ff:ff
24: swp21: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:16 brd ff:ff:ff:ff:ff:ff
25: swp22: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:17 brd ff:ff:ff:ff:ff:ff
26: swp23: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:18 brd ff:ff:ff:ff:ff:ff
27: swp24: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:19 brd ff:ff:ff:ff:ff:ff
28: swp25: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:1a brd ff:ff:ff:ff:ff:ff
29: swp26: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:1b brd ff:ff:ff:ff:ff:ff
30: swp27: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:1c brd ff:ff:ff:ff:ff:ff
31: swp29: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether f2:82:a1:e7:be:1e brd ff:ff:ff:ff:ff:ff
32: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether d8:3a:dd:db:e2:9a brd ff:ff:ff:ff:ff:ff
```
Ethtool Command
```shell
$ ethtool -i swp0
```
Returns:
```shell
driver: sparx5-switch
version: 6.12.48-v8+
firmware-version: 
expansion-rom-version: 
bus-info: 6020c0000.switch
supports-statistics: yes
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: no
```
### 4. Test the Ethernet interface.
Connect the Ethernet to a link partner and then ping:
```shell
# Assign address to eth1
$ sudo ip addr add 10.0.1.10/24 dev swp1

# Ping the link partner
$ ping 10.0.1.2
PING 10.0.1.2 (10.0.1.2) 56(84) bytes of data.
64 bytes from 10.0.1.2: icmp_seq=1 ttl=64 time=0.577 ms
64 bytes from 10.0.1.2: icmp_seq=2 ttl=64 time=0.399 ms
64 bytes from 10.0.1.2: icmp_seq=3 ttl=64 time=0.434 ms
64 bytes from 10.0.1.2: icmp_seq=4 ttl=64 time=0.545 ms
^C
--- 10.0.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3052ms
rtt min/avg/max/mdev = 0.399/0.488/0.577/0.074 ms
```

```shell
# IPERF TEST
Connecting to host 10.0.1.2, port 5201
[  5] local 10.0.1.10 port 54074 connected to 10.0.1.2 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   112 MBytes   934 Mbits/sec    2    472 KBytes       
[  5]   1.00-2.00   sec   110 MBytes   927 Mbits/sec    0    628 KBytes       
[  5]   2.00-3.00   sec   111 MBytes   934 Mbits/sec    0    754 KBytes       
[  5]   3.00-4.00   sec   112 MBytes   935 Mbits/sec    0    860 KBytes       
[  5]   4.00-5.00   sec   111 MBytes   934 Mbits/sec    0    954 KBytes       
[  5]   5.00-6.00   sec   110 MBytes   922 Mbits/sec    0   1.02 MBytes       
[  5]   6.00-7.00   sec   112 MBytes   935 Mbits/sec    0   1.09 MBytes       
[  5]   7.00-8.00   sec   110 MBytes   925 Mbits/sec    1    875 KBytes       
[  5]   8.00-9.00   sec   111 MBytes   928 Mbits/sec    0    966 KBytes       
[  5]   9.00-10.01  sec   112 MBytes   936 Mbits/sec    0   1.02 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.01  sec  1.08 GBytes   931 Mbits/sec    3            sender
[  5]   0.00-10.02  sec  1.08 GBytes   927 Mbits/sec                  receiver
```

## Screenshots of the Actual Result
![](Pasted-image-20260330083400.png)
![](Pasted-image-20260330083200.png)
![](Pasted-image-20260330083212.png)
![](Pasted-image-20260330083237.png)

---