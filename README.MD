# OpenCore EFI for AMD Ryzen Hackintosh

## Specification
| **Component** | **Model** |
| ------------- | --------- |
| CPU | AMD Ryzen 7 1700 @ 3.8GHz |
| Motherboard | ASUS B350 Plus |
| RAM | 16GB (2 x 8GB) Corsair Vengeance @ 3000MHz |
| Audio Chipset | ALC-887 |
| GPU | MSI RX Vega 64 |
| WiFi & Bluetooth | Fenvi T919 (BCM94360CD) |
| OS Disk (NVMe) | ADATA SX8200 Pro 1TB |

**macOS version**: 11.3.1 (20E241) \
**OpenCore version**: 0.6.9

## Table of content
 - [Compatible macOS versions](#Compatible-macOS-versions)
 - [Issues](#Issues)
 - [How to use](#How-to-use)
 - [BIOS Settings](#BIOS-Settings)
 - [Hardware compatibility](#Hardware-compatibility)
 - [Sleep information](#Sleep-information)
 - [PAT patch information](#PAT-patch-information)
 - [Adobe applications fix](#Adobe-applications-fix)
 - [Virtualization](#Virtualization)
 - [Guides](#Guides)
 - [Credits](#Credits)

## Compatible macOS versions
 - Big Sur (11.x)
 - Catalina (10.15.x)
 - Mojave (10.14.x)
 - High Sierra (10.13.x)

## Issues
 - Not working 3.5mm Jack microphone (only USB/Bluetooth microphones)

## How to use
  1. Make your USB installer with [**this guide**](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/)
  2. Clone the repository and paste "BOOT" and "OC" directories into your's pendrive "EFI" folder
  3. Download [**GenSMBIOS**](https://github.com/corpnewt/GenSMBIOS) to generate unique SMBIOS information. Run it and select **Generate SMBIOS**, as the model select **iMacPro1,1**.
  4. Open config.plist with [**ProperTree**](https://github.com/corpnewt/ProperTree) and go to PlatformInfo > Generic. Set MLB (Board Serial), SystemSerialNumber (Serial) and SystemUUID (SmUUID) to generated values. Change ROM to your network card's MAC address without the `:` character. [**How to get MAC Address?**](https://www.wikihow.com/Find-the-MAC-Address-of-Your-Computer)
  5. Update your BIOS to latest version and set it's settings to [**required values**](#BIOS-Settings)
  6. Read [**information**](#Hardware-compatibility) about required changes for some hardware
  7. Boot and install it!
  8. After installation you can copy your EFI directory to disk's EFI partition - then you can boot macOS without pendrive
  9. If you want to use dual boot you have to enable Bootstrap - it will protect your OpenCore to be overriden by Windows' boot loader. Click [**here**](https://dortania.github.io/OpenCore-Post-Install/multiboot/bootstrap.html) for information about it.
  10. To fix not working apps (e. g. Discord voice chat) download and run [**ryzen_patch.sh**](/Resources/ryzen_patch.sh). When patching is done remember to reboot your Hackintosh.

**You CAN NOT use SMBIOS from this repository, it MUST be unique for every macOS installation**

## BIOS Settings
| **Option** | **Status** |
| ------------- | --------- |
| SATA Mode | AHCI |
| Above 4G Decoding | Enabled <sup>1</sup> |
| EHCI/XHCI Hand-off | Enabled |
| SVM | Enabled |
| CSM | Disabled |
| Resizable BAR Support | Disabled |
| Secure Boot | Disabled |
| Serial Port | Disabled |
| Parallel Port | Disabled |

<sup>1</sup> If you have this option you **MUST** remove `npci=0x2000` from `boot-args` in `config.plist`

**Most of these options may not exist in your firmware, just try to match it as closely as possible. Don't be too concerned if many of these options are not available in your BIOS**

**Before booting macOS remember to update BIOS to latest version**

## Hardware compatibility
Most builds with **17h and 19h (All Ryzen's generations, Athlon 2xxGE)** CPUs with [**macOS compatible peripherals**](https://dortania.github.io/Anti-Hackintosh-Buyers-Guide/CPU.html) should work with EFI from this repository. \
**Support for 15h (FX series), 16h (A series) and Threadripper CPUs is not covered here.**

Integrated GPUs **DO NOT WORK**, for NVIDIA check details [**here**](https://dortania.github.io/GPU-Buyers-Guide/modern-gpus/nvidia-gpu.html) (it's not covered in this repository). \
If you have **NVIDIA GPU** you may have to change PAT Patch to Algrey's version, read [**here**](#PAT-patch-information) about it

Motherboards with **B550 and A520** chipsets require additional **SSDT-CPUR** to boot macOS. \
You have to [**download it**](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-CPUR.aml) and put it under `OC/ACPI` directory. Then you have to add it to your config. Open it with ProperTree or your favourite text editor and add it under `ACPI -> Add`. You should do this in the same way as you did `SSDT-EC-USBX-DESKTOP`, just change the name of the file.

For **B550, A520 and B450, X470, X570 with newer BIOS versions** `SetupVirtualMap` must be disabled. Go to `Booter -> Quirks -> SetupVirtualMap` in your config and change it to `false`

For **AMD Navi GPUs (RX 5500, 5600, 5700)** you have to add `agdpmod=pikera` to `boot-args` to fix black screen issue.

If you experience issues with your audio, then you have to change `alcid` proper for your audio chipset. Find your chipset [**here**](https://github.com/acidanthera/applealc/wiki/supported-codecs) and try setting `alcid` in `boot-args` parameter to every layout-id values from AppleALC wiki until you get correct value (working audio inputs and outputs) for your motherboard.

If you experience issues with network connection, then you probably have another ethernet chipset. Click [**here**](https://dortania.github.io/OpenCore-Install-Guide/ktext.html#ethernet) for information about kexts for ethernet cards. \
For High Sierra use [**this kext**](https://bitbucket.org/RehabMan/os-x-realtek-network/downloads/) for Realtek 8111 ethernet card, because version from this repository needs Mojave or newer version. \
If you are using Big Sur with RTL8111 but network is unstable try [**this version**](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases/tag/v2.2.2) of kext. \
Remember to remove `RealtekRTL8111.kext` before use another kext.

By default enabled is PAT patch made by Shaneee - it improves GPU performance but can cause some issues. It can break HDMI audio and make builds with NVIDIA GPU unbootable. You can switch patch with better compatibility (but worse performance). Click [**here**](#PAT-patch-information) for more information.

## Sleep information
If you have issues with sleep, firstly you have to map your USB ports. You can read about it [**here**](https://dortania.github.io/OpenCore-Post-Install/usb/). If map does not help, you should try patching USB via SSDT.

In `SSDT-SLEEP.aml` there are patches for _STA method. Patch as applied to `_SB.PCI0.GPP2.PTXH` and `_SB.PCI0.GP17.XHC0` USB controllers, if you have other addresses of USB controllers you have to edit SSDT for your build. Patch is applied only for macOS, so USB on other systems should work normally.

Sleep in AMD systems is often broken by USB issues, but not always. If USB patching does not help read [**this guide**](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html) about fixing sleep.

## PAT patch information
| **Shaneee's** | **Algrey's** |
| ------------- | --------- |
| Much better GPU performance | Worse GPU performance |
| May not work with NVIDIA GPUs | Compatible with all GPUs |
| HDMI/DP audio may not work | HDMI/DP audio works |
| Enabled by default | Disabled by default |

To switch to another patch search for `mtrr_update_action` in `config.plist`. Then set `Enabled` to `true` for patch which you want to use. Remember to set `Enabled` to `false` for second PAT patch.

**Don't try to use them both at the same time, it won't work.**

## Adobe applications fix
Adobe applications crash on AMD Hackintoshes due to missing intel_fast_memset instructions.
Run [**adobe_patch.sh**](/Resources/adobe_patch.sh) or if you prefer to do it manually follow [**this guide**](https://gist.github.com/mikigal/8e1f804fcd7dbafbded2f236653be7c8) to get it working. Remember to re-run this script after installation of every Adobe app. Remember to run [**ryzen_patch.sh**](/Resources/ryzen_patch.sh) previously!

If Photoshop crashes while opening image from file you have to downgrade it to version 22.0

## Virtualization
Firstly you have to enable `SVM` in your BIOS settings. \
Parallels Desktop (only up to 13.1, newer version require AppleHV) and VirtualBox (it works much worse than Parallels) are only compatible software for virtual machines. There's also VMWare Fusion 10, but it's totally broken on Big Sur, on Catalina it needs [**this workaround**](https://posts.boy.sh/vmware-fusion-catalina) \
Docker also does not work - you have to use Docker Toolbox instead, but it does not have all Docker's features.

On Big Sur (11.0) Parallel's won't start - it will show `Required components are missing from the OS` error due to changed versions number schema. You have to run installer from with `SYSTEM_VERSION_COMPAT=1` parameter. \
Open your Terminal app and run Parallel's installer like this: `SYSTEM_VERSION_COMPAT=1 open /Volumes/Parallels\ Desktop\ 13.1.0/Install.app/`, just replace volume's name to proper for your Parallel's DMG. \
Same problem will exist while trying to start installed Parallels - you can run it every time with the same way as installer or use my launcher (it simply starts Parallels with required parameter) - you can get it [**here**](/Resources/Parallels%20Desktop%20Launcher.app.zip)

Parallels 13.1 support only Windows 10 Anniversary Update (build 1607) and older versions. Newer versions stuck while installing. Also I recommend using Windows 7 - it works much better. Do not use automatical installation feature - it caused some issue for me.

**DO NOT** add too much resources to virtual machines, it causes performance issues independently of host specification. \
I tested lot of VM configurations - the best performance results gives:
  - Parallels Desktop 13.1
  - 4 CPU cores
  - 4GB RAM
  - 1GB VRAM
  - 3D Acceleration: DirectX 9
  - OS: Windows 7 (SP1, build 7601) with disabled Aero theme

With above configuration system is generally responsive, it runs simple games (e. g. The Binding of Isaac: Repentace) smoothly too. All Parallels host-integration features works great.

If guest OS does not see USB device reconnecting it to another port usually solve this issue. \
If [**Coherence Mode**](https://www.parallels.com/blogs/how-to-use-coherence-mode-in-parallels-desktop/) does not work you have to disable guest's antivirus or add these files to it's exclusions list:
  - `C:\Program Files (x86)\Parallels\Parallels Tools\Services\coherence.exe`
  - `C:\Program Files (x86)\Parallels\Parallels Tools\Services\prl_hook.dll`

## Guides
**If you have any problems with installation or booting your macOS, kernel panics or another system related issue check OC configuration guide**

**If something else does not work properly (for example USB ports, iServices, DRM/Netflix) check Post-Install guide**
 - Creating USB installer: [**\*click\***](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/)
 - OpenCore configuration: [**\*click\***](https://dortania.github.io/OpenCore-Install-Guide/AMD/zen.html)
 - Post-Install: [**\*click\***](https://dortania.github.io/OpenCore-Post-Install/)
 - Troubleshooting: [**\*click\***](https://dortania.github.io/OpenCore-Post-Install/)
 - ACPI patching: [**\*click\***](https://dortania.github.io/Getting-Started-With-ACPI/)
 - USB mapping: [**\*click\***](https://dortania.github.io/OpenCore-Post-Install/usb/)

If you have any other questions or issues, feel free to ask on [**AMD-OSX Discord**](https://discord.gg/EfCYAJW) or [**Forum**](https://forum.amd-osx.com)

## Credits
**Software:**
 - [[Bootloader] OpenCore](https://github.com/acidanthera/OpenCorePkg)
 - [[Resources] Picker GUI](https://github.com/acidanthera/OcBinaryData/tree/master/Resources)
 - [[Patch] AMD_Vanilla](https://github.com/AMD-OSX/AMD_Vanilla)
 - [[SSDT] EC-USBX-DESKTOP](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-EC-USBX-DESKTOP.aml)
 - [[SSDT] SLEEP-PTXH](./OC/ACPI/SSDT-SLEEP-PTXH.aml)
 - [[Driver] OpenRuntime](https://github.com/acidanthera/OpenCorePkg)
 - [[Driver] OpenCanopy](https://github.com/acidanthera/OpenCorePkg)
 - [[Driver] OpenHfsPlus](https://github.com/acidanthera/OpenCorePkg)
 - [[Kext] Lilu](https://github.com/acidanthera/Lilu)
 - [[Kext] VirtualSMC](https://github.com/acidanthera/VirtualSMC)
 - [[Kext] WhateverGreen](https://github.com/acidanthera/WhateverGreen)
 - [[Kext] AppleALC](https://github.com/acidanthera/AppleALC)
 - [[Kext] RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X)
 - [[Kext] AMDRyzenCPUPowerManagement](https://github.com/trulyspinach/SMCAMDProcessor)
 - [[Kext] SMCAMDProcessor](https://github.com/trulyspinach/SMCAMDProcessor)
 - [[Kext] NVMeFix](https://github.com/acidanthera/NVMeFix)
 - [[Kext] RestrictEvents](https://github.com/acidanthera/RestrictEvents)
 - [[Kext] AppleMCEReporterDisabler](https://github.com/AMD-OSX/AMD_Vanilla/blob/opencore/Extra/AppleMCEReporterDisabler.kext.zip)

 **People:**
 - [Apple](https://apple.com) for macOS
 - [AMD-OSX Developers](https://github.com/AMD-OSX) for kernel patches for AMD CPUs
 - [Acidanthera](https://github.com/acidanthera) for OpenCore and most of used kexts
 - [Trulyspinach](https://github.com/trulyspinach) for Ryzen power management and monitoring kexts
 - [Mieze](https://github.com/Mieze) for RealtekRTL8111 kext
 - [Dortania](https://github.com/dortania) for OpenCore configuration guides
 - [XLNC](https://github.com/naveenkrdy) for Adobe patches for AMD CPUs
 - [AMD-OSX Community](https://amd-osx.com) for support while making my Hackintosh
<br>

![Screenshot](/screenshot.png?raw=true)
