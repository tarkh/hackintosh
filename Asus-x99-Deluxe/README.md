# Asus x99 Deluxe
**([русская версия](https://github.com/tarkh/hackintosh/blob/main/Asus-x99-Deluxe/README-rus.md))**

## <img src="https://via.placeholder.com/12/f03c15/f03c15.png"> `Version for macOS 14 (Sonoma)`
<p align="center">
  <img src="./benchmarks/aboutxl.webp">
</p>

##
* [Introductory Information](#intro)
* [BIOS Setup](#biosSetup)
* [macOS Startup](#runMac)
* [Nvidia Kepler](#nvidiaKepler)
* [AMD Radeon](#amdRadeon)
* [WiFi and Bluetooth](#wifiAndBluetooth)
* [Removing the USB drive from the system](#removeUsb)
* [Benchmarks in macOS](#benchmarks)
* [Epilogue](#end)

<a name="intro"></a>
## Introductory information

This repository provides information on installing **macOS** on a system with the following technical characteristics:

* Motherboard: `Asus x99 Deluxe`
* Processor: `i7-5960X`

**OpenCore** will be used as an EFI loader, therefore, before starting current process, it is highly recommended to familiarize yourself with this loader and its functionality [here (OpenCore Guide)](https://dortania.github.io/OpenCore-Install-Guide/).

<a name="biosSetup"></a>
## BIOS Setup

Bios version: `3802`

Main settings:
```
Enhanced Intel SpeedStep Technology [Enabled]
Turbo Mode [Enabled]
Hyper-Threading [ALL] [Enabled]
Limit CPUID Maximum [Disabled]
Execute Disable Bit [Enabled]
Intel Virtualization Technology [Enabled]
Hardware Prefetcher [Enabled]
Boot Performance Mode [Max Performance]
CPU C-States [Auto]
SATA Controller 2 Mode Selection [AHCI]
Intel VT for Directed I/O (VT-d) [Disabled]
Intel xHCI Mode [Enabled]
EHCI Legacy Support [Auto]
xHCI Hand-off [Enabled]
EHCI Hand-off [Enabled]
HD Audio Controller [Enabled]
Asmedia USB 3.0 Controller [Enabled]
Bluetooth Controller [Enabled]
Wi-Fi Controller [Enabled]
Intel LAN1 Controller [Enabled]
Intel LAN2 Controller [Enabled]
Fast Boot [Enabled]
SATA Support [All Devices]
USB Support [Full Initialization]
Boot up NumLock State [Enabled]
Above 4G Decoding [Enabled]
Launch CSM [Disabled]
OS Type [Other OS]
```

<a name="runMac"></a>
## Launching macOS

To run macOS, we need the `./EFI` directory, which must be copied to the root of the USB drive, previously formatted it in **fat32**. Pay attention, that `./EFI` directory contains `production` version of bootloader.

> If you don't have macOS installed earlier on the internal disk, then you need to create an installation flash drive according to [official OpenCore instructions](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/). When you have written the installation flash drive, copy the `./EFI` directory from this repository to the root of the mounted EFI partition.

The EFI directory contains an OpenCore loader pre-configured for this particular system: patched and compiled all the necessary ACPI tables, added the necessary kexts, added support for 24 cores and 48 threads, configured `EFI/OC/config.plist`.

However, before launching, it is necessary to make some mandatory changes to the EFI directory and `EFI/OC/config.plist`. To avoid unexpected errors, open files like `*.plist` in a specialized plist editor! So, what needs to be done:

* Generate and add your own unique serial numbers for the model **MacPro7,1**. You will need the [GenSMBIOS utility](https://github.com/corpnewt/GenSMBIOS). The received UUIDs and serial numbers need to be replaced in the file `EFI/OC/config.plist` in the following keys:
* `PlatformInfo > Generic > MLB` we set the generated value `Board Serial`.
* `PlatformInfo > Generic > SystemSerialNumber` we set the generated value `Serial`.
* `PlatformInfo > Generic > SystemUUID` we set the generated value `SmUUID`.
* `PlatformInfo > Generic > ROM` we set the generated value `Rom`.
* USB ports are configured for a specific motherboard model, the settings are in `EFI/OC/Kexts/USBPorts.kext/Contents/info.plist`. Detailed instructions for configuring USB ports [can be found here](https://dortania.github.io/OpenCore-Post-Install/usb/system-preparation.html).

Save the configuration file, restart the computer, go to the BIOS and select boot from the UEFI partition of your flash drive. If everything was done correctly, you will see the OpenCore boot menu, where you can choose an internal drive with macOS already installed or the macOS installer in case of a fresh installation.

<a name="nvidiaKepler"></a>
## Nvidia Kepler
Starting from macOS Monterey Apple has removed drivers for Nvidia Kepler from the system. Never the less, there is ways to revert support for this video cards with [OpenCore Legacy Patcher](https://dortania.github.io/OpenCore-Legacy-Patcher/). Familiarize with it's manual and some limitation, that will occure while using this modification.

Change the boot keys in NVRAM:

* `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82` comment (rename) the `boot-args` key to `#boot-args.`
* `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82` uncomment (rename) the `#NVIDIA_KEPLER#boot-args` key to `boot-args`.

<a name="amdRadeon"></a>
## AMD Radeon
For proper operation of AMD sensors, enable 2 kext files `RadeonSensor.kext` and `SMCRadeonGPU.kext` in `EFI/OC/config.plist` under the following keys:

* `Kernel > Add > 20 > Enabled` set to `True`.
* `Kernel > Add > 21 > Enabled` set to `True`.

In some cases, the `Navi` generation may require the boot parameter `agdpmod=pikera`. You can read more about NVRAM parameters in the [official Opencore documentation](https://dortania.github.io/OpenCore-Install-Guide/AMD/zen.html#nvram). Support for other generations of AMD video cards you can try to patch using [OpenCore Legacy Patcher](https://dortania.github.io/OpenCore-Legacy-Patcher/). The 500 series of AMD video cards (rx560, rx580...) should work without additional manipulation.

<a name="wifiAndBluetooth"></a>
## WiFi and Bluetooth
The motherboard has a module with a Broadcom chip, support for which has ceased in the current version of macOS. For it to function, after installing the macOS system, it needs to be patched using the [OpenCore Legacy Patcher](https://dortania.github.io/OpenCore-Legacy-Patcher/). After the patch, without reboot, you need to enable in `EFI/OC/config.plist` kext files `IOSkywalkFamily.kext`, `IO80211FamilyLegacy.kext` and `IO80211FamilyLegacy.kext/Contents/PlugIns/AirPortBrcmNIC.kext`:

* `Kernel > Add > 13 > Enabled` set to `True`.
* `Kernel > Add > 14 > Enabled` set to `True`.
* `Kernel > Add > 15 > Enabled` set to `True`.

Also, it is necessary to block the system driver `com.apple.iokit.IOSkywalkFamily`:

* `Kernel > Block > 0 > Enabled` set to `True`.

After this, you need to reboot the system and the `Fenvi T919` should work along with Apple's wireless functions.

<a name="removeUsb"></a>
## Removing the USB drive from the system

After the perfect efi driver is found and you are **happy**, we are ready to get rid of a bootable USB flash drive in the USB connector of the computer. To do this, we need to mount the EFI partition of the internal disk and move the EFI directory from the root of our USB drive to its root. If we use macOS, it is advised to mount the EFI partition on the same media where the OS is located. This is also true for any other operating systems. In the case of multi-OS boot, it is enough to place OpenCore on one of the disks with OS and set it first in the queue for loading in the bios.

You can find out how to mount EFI partitions in different operating systems in `Google`.

<a name="benchmarks"></a>
## Benchmarks in macOS

<table width="100%">
  <tr>
    <td align="center" valign="top" colspan="2">
      Cinebench R23<br>
      <img src="./benchmarks/cinebenchR23.webp">
    </td>
  </tr>
  <tr>
    <td align="center" valign="top">
      Geekbench 5<br>
      <img src="./benchmarks/geekbench5.webp">
    </td>
  </tr>
</table>

<a name="end"></a>
## Epilogue

All main functions, such as CPU Power Management, USB ports, WiFi+Bluetooth, and Sleep, work properly. Wireless functions like Handoff and AirDrop are also operational. The operating system itself runs quickly and smoothly.

I will be glad to receive feedback in order to improve this configuration. You can find me on telegram:
* [@tarkhx](https://t.me/tarkhx)

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p align="center">
  <img src="./about.webp">
</p>
