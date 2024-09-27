# Huananzhi F8D
**([русская версия](https://github.com/tarkh/hackintosh/blob/main/Huananzhi-F8D/README-rus.md))**

## <img src="https://via.placeholder.com/12/f03c15/f03c15.png"> `Version for macOS 15 (Sequoia)`
<p align="center">
  <img src="./aboutxl.webp?v=15">
</p>

*This version should work with macOS operating systems starting from **10.14 (Mojave)** and ending with **macOS 15 (Sequoia)**, however, if you have problems running **macOS < version 15**, you can try an earlier build from the corresponding branch `macOS 10.14 (Mojave)`, `macOS 11 (Big Sur)`, `macOS 12 (Monterey)` or `macOS 14 (Sonoma)` in this repository.*

##
* [Introductory Information](#intro)
* [BIOS Setup](#biosSetup)
* [macOS Startup](#runMac)
* [Nvidia Kepler](#nvidiaKepler)
* [AMD Radeon](#amdRadeon)
* [Fenvi T919 WiFi+BT](#fenviT919)
* [BIOS Firmware](#wbios)
* [Undervolting](#undervolting)
* [For Windows users](#windows)
* [For Linux users](#linux)
* [Removing the USB drive from the system](#removeUsb)
* [I'm pro, I don't want to read many letters](#imPro)
* [Benchmarks in macOS](#benchmarks)
* [Epilogue](#end)

<a name="intro"></a>
## Introductory information

This repository provides information on installing **macOS** on a system with the following technical characteristics:

* Motherboard: Huananzhi F8D
* Processor: x2 E5-2698V3

**OpenCore (1.0.1)** will be used as an EFI loader, therefore, before starting current process, it is highly recommended to familiarize yourself with this loader and its functionality [here (OpenCore Guide)](https://dortania.github.io/OpenCore-Install-Guide/). In addition to the basic OpenCore setup, a method for unlocking turbo mode on all processor cores and its undervolting will be described.

**CPU setup hint**: Xeon E5-2698V3 was chosen because it has 16 cores and 32 threads, so total 2x CPUs will have 32 cores and 64 threads in the system. Keep in mind, that **macOS kernel do not support CPU setups with more than 64 threads in total**, so if you have larger amount of cores, disable some of them in bios to fit 64 threads limit!

**System memory setup hint**: this motherboard supports **8 channel** DDR4 RAM (x2 CPUs with 4 channel each), so if you utilize all 8 memory slots, this will give you performance boost **up to 15%** comparing to only 4 utilized slots (2 for each CPU).

<a name="biosSetup"></a>
## BIOS Setup

If you are not going to flash the BIOS from this repository, then you need to make certain changes to the settings of the stock bios. The modified bios from this repository already contains all necessary default settings for the correct launch of macOS.

To get started, go to the bios, reset all settings to default and save with a reboot, then change the following parameters:

* `Advanced > Trusted Computing > Security Device Support` **Disabled**
* `Advanced > ACPI Settings > Enable ACPI Auto Configuration` **Enabled**
* `Advanced > NCT5532D SSIO Configuration > Serial Port 1 Configuration > Serial Port` **Disabled**
* `Advanced > Smart Fan function setting` should be configured depending on your cooling system and processors. In my case, with A500 towers on top of both processors, options are set as follows (quiet and temperature at full load does not exceed 65 degrees):
* `Smart Fan Temperature 1` **30**
* `Smart Fan Temperature 2` **42**
* `Smart Fan Temperature 3` **55**
* `Smart Fan Temperature 4` **67**
* `Smart Fan Critical Temperature` **67**
* `Smart Fan PWM 1` **25**
* `Smart Fan PWM 2` **51**
* `Smart Fan PWM 3` **127**
* `Smart Fan PWM 4` **255**
* `Advanced > Serial Port Console Redirection > Console Redirection` **Disabled**
* `Advanced > PCI Subsystem Settings > Above 4G Decoding` **Enabled**
* `Advanced > Network Stack Configuration > LAN Wake up Control` **Disabled**
* `Advanced > CSM Configuration > CSM Support` **Disabled**. To disable this option, first you need to switch the Video from `Legacy` mode to `UEFI` mode, save the BIOS settings and reboot. Only then it will be possible to set `CSM Support` to the **Disabled** position. At the same time, depending on the video card, when setting the Video parameter from the `Legacy` mode to the `UEFI` mode, it is necessary to put the `Advanced > PCI Subsystem Settings > Above 4G Decoding` parameter in the **Enabled** position, otherwise the video may disappear after reboot and the bios will have to be reset by removing the battery.
* `Advanced > USB Configuration > EHCI Hand-Off` **Enabled**
* `IntelRCSetup > Processor Configuration > MSR Lock Control` **Disable**
* `IntelRCSetup > Advanced Power Management Configuration > CPU C State Control > Package C State limit` **C2 state**
* `IntelRCSetup > Advanced Power Management Configuration > CPU C State Control > CPU C3 report` **Enable**
* `IntelRCSetup > Advanced Power Management Configuration > CPU C State Control > CPU C6 report` **Disable**
* `Security > Secure Boot menu > Secure Boot` **Disabled**

<a name="runMac"></a>
## Launching macOS

To run macOS, we need the `./EFI` directory, which must be copied to the root of the USB drive, previously formatted it in **fat32**. Pay attention, that `./EFI` directory contains `production` version of bootloader.

> If you don't have macOS installed earlier on the internal disk, then you need to create an installation flash drive according to [official OpenCore instructions](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/). When you have written the installation flash drive, copy the `./EFI` directory from this repository to the root of the mounted EFI partition.

The EFI directory contains an OpenCore loader pre-configured for this particular system: patched and compiled all the necessary ACPI tables, added the necessary kexts, configured `EFI/OC/config.plist`.

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

<a name="amdRadeon"></a>
## AMD Radeon
For proper operation of AMD sensors, **enable (Enabled: True)** kext files in `EFI/OC/config.plist` under `Kernel > Add` path:

* `RadeonSensor.kext`
* `SMCRadeonGPU.kext`

Change the boot keys in NVRAM:

* `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82` comment (rename) the `boot-args` key to `#boot-args.`
* `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82` uncomment (rename) the `#RADEON#boot-args` key to `boot-args`.

You can read more about NVRAM parameters in the [official Opencore documentation](https://dortania.github.io/OpenCore-Install-Guide/AMD/zen.html#nvram).

<a name="fenviT919"></a>
## Fenvi T919 WiFi+BT
I have a `Fenvi T919` PCI card installed with a Broadcom chip, support for which has ceased in the current version of macOS. For it to function, after installing the macOS system, it needs to be patched using the [OpenCore Legacy Patcher](https://dortania.github.io/OpenCore-Legacy-Patcher/). After the patch, without reboot, you need to **enable (Enabled: True)** kext files in `EFI/OC/config.plist` under `Kernel > Add` path: 

* `IOSkywalkFamily.kext`
* `IO80211FamilyLegacy.kext`
* `IO80211FamilyLegacy.kext/Contents/PlugIns/AirPortBrcmNIC.kext`
* `AirportBrcmFixup.kext`
* `AirportBrcmFixup.kext/Contents/PlugIns/AirPortBrcmNIC_Injector.kext`

**Enable (Enabled: True)** the system driver blocking in `EFI/OC/config.plist` under `Kernel > Block` path:

* `com.apple.iokit.IOSkywalkFamily`

After this, you need to reboot, reset NVRAM and the `Fenvi T919` should work along with Apple's wireless functions including Bluetooth features.

<a name="wbios"></a>
## BIOS flashing

This method uses BIOS [HNX99F8D_220105_kot](https://github.com/Koshak1013/HuananzhiX99_BIOS_mods/tree/master/Huananzhi%20X99-F8D/2022-01-05) from the wonderful repository of [Koshak1013](https://github.com/Koshak1013). This bios contains a number of fixes, microcode updates, BCLK 100.00MHz, access to memory timings settings and unlocked ME-regions so that you can later flash bioses without a programmer.

In the directory `./Bios` you'll find bios file `F8D_220105_KSM.bin`:

* **F8D_220105_KSM.bin**, where:
* **K** is the original bios from [Koshak1013](https://github.com/Koshak1013 ) (`HNX99F8D_200624_kot`).
* **S** - all bios settings alredy builted in for correct operation in macOS (from [BIOS Setup](#biossetup)).
* **M** - the microcode `6F 06F2` has been removed, turbo boost on all cores unlocked, undervolting available from bios menu.

If there is a stock bios in your motherboard, then the ME-regions are locked in it. Nevertheless, there are 2 ways to flash the modified bios...

> **As usual, you perform all operations with bios firmware at your own risk, with full understanding that it is simply necessary to have a programmer to be able to restore the "bricked" board.**

#### Method 1 (correct)

Use the programmer `CH341A` with a test clip. In this case, there is no need to solder out the bios, it is enough to put on the test clip correctly, remove the battery from the motherboard, and leave the PSU connected to the power with the latch turned on. You can `Google` a lot of materials on doing this process on Windows machine.

I would like to describe an alternative way of flashing the bios through a programmer from under macOS or Linux. To do this, we need the command-line utility [flashrom](https://flashrom.org/Flashrom). In my opinion, working with it is much easier than using a GUI application in Windows. On macOS, this utility can be [installed via Homebrew](https://brewinstall.org/install-flashrom-on-mac-with-brew/). After installation, the `flashrom` command becomes available in the Terminal ([read man](https://linux.die.net/man/8/flashrom)). Afret connecting the programmer correctly we can make a dump of the current bios:

`sudo flashrom --programmer ch341a_spi -r backup.bin`

Then we flash our modified bios:

`sudo flashrom --programmer ch341a_spi -w F8D_220105_KSM.bin`

Flashrom will automatically dump bios into memory, clear the bios, flash a new one, read it and compare it with the dump. In general, everything is quite simple.

#### Method 2

Despite the fact that the stock bios has a lock on the ME-regions, it is still possible to flash over the modified one using the `Afudos` program from a bootable USB drive. The bios will be flashed, but not completely, leaving the locked ME-regions. This method of flashing was also tested by me personally, the turbo boost unlock and the memory timings menu work. Nevertheless, **I highly recommend getting a programmer and flashing the modified bios correctly 1 time, all subsequent flashing can be done without a programmer, since everything will be unlocked**.

#### Post-flashing

After flashing the bios from this repository, there is no need to reset the bios settings in the motherboard to defaults - everything is already configured for macOS to work correctly. When bios flashing is completed, turn off the computer, wait 10 seconds and turn it back on. You can boot into macOS.

<a name="undervolting"></a>
## Undervolting

All CPU overclocking options are availabe from Overclocking bios menu under `IntelRCSetup` section. You can adjust core voltages offset, FIVR etc. 

<a name="windows"></a>
## For Windows users

This repository is primarily intended for Hackintosh adherents, but the OpenCore EFI loader we use is configured and fully ready to work with Windows OS, both in dual-boot mode with macOS and without it.

>**The main requirement for Windows is that the operating system must be installed only in UEFI mode!**

<a name="linux"></a>
## For Linux users

For Linux users everything is the same as [For Windows users](#windows).

<a name="removeUsb"></a>
## Removing the USB drive from the system

After all setup is done and you are **happy**, we are ready to get rid of a bootable USB flash drive in the USB connector of the computer. To do this, we need to mount the EFI partition of the internal system drive and move our EFI directory from our USB drive there. If we use macOS, it is advised to mount the EFI partition on the same media where the OS is located. This is also true for any other operating systems. In the case of multi-OS boot, it is enough to place OpenCore on one of the disks with OS and set it first in the queue for loading in the bios.

You can find out how to mount EFI partitions in different operating systems in `Google`.

<a name="imPro"></a>
## I'm a Pro, I Don’t Want to Read a Lot

OK, then it's simple:

* Flash bios **F8D_220105_KSM.bin**. It contains all bios settings for macOS + microcode `6F 06F2` removed + turbo unlocked + overclocking available from bios menu.
* Format the flash drive in **fat32**, drop the `./EFI` directory into the root of USB stick.
* Alter `EFI/OC/config.plist` with your generated serial numbers for **MacPro7,1**.
* Save, turn off, wait, turn on, boot from the flash drive — win-win.

<a name="benchmarks"></a>
## Benchmarks in macOS

<table width="100%">
  <tr>
    <td align="center" valign="top" colspan="2">
      Cinebench R23<br>
      <img src="./benchmarks/cinebenchR23.webp?v=15">
    </td>
  </tr>
  <tr>
    <td align="center" valign="top">
      Geekbench 5<br>
      <img src="./benchmarks/geekbench5.webp?v=15">
    </td>
  </tr>
</table>

<a name="end"></a>
## Epilogue

Regarding macOS, I achieved a fully stable and functional system. Power management works well, and even without unlocking turbo boost, the system delivers good results. On average, it’s about 15-20% less than with turbo unlocked.

>**Sleep mode doesn’t work, but this is a limitation of the macOS kernel itself, as there are no modern Apple computers with dual-socket setups. Perhaps someone will solve this problem someday. If you discover a solution, please let me know.**

I would appreciate any feedback to improve this configuration. You can find me on Telegram:
* [@tarkhx](https://t.me/tarkhx)
* [Telegram group on LGA2011 and XEON](https://t.me/Chinese_lga2011_3_x99)

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p align="center">
  <img src="./about.webp?v=15">
</p>