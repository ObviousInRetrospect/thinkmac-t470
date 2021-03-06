# Lenovo ThinkPad T470 (Type 20JM) Hackintosh

_**Supports:** macOS Catalina 10.15.x including iCloud, iMessage, FaceTime, etc._

![macOS Catalina on the ThinkPad T470](/macos-t470.png)

### Introduction
This is my attempt at outlining the necessary configuration and dependencies to get macOS running on a ThinkPad T470. There's quite a lot of useful Reddit & forum posts, guides and other repositories here on GitHub that I was able to pull small bits of information from in order to make this guide. One problem that I discovered is that the T470 has shipped with many different hardware configurations making a one-size-fits-all approach impossible. Another problem I found is that many of the sources I used didn't provide enough detail making the learning curve a little more challenging. In turn, I decided to create this for anyone else who may have purchased one of these T470 models (specifically, Type 20JM, although it may work on similar sub-models with minor modifications).

### Definitions
This section is used to specify some abbreviations or acronyms you'll see either in this guide or elsewhere in the hackintosh community. It can be a little confusing following instructions if you don't know what certain things stand for.

* **\/S\/L\/E** - /System/Library/Extensions (location of official Apple kexts)

* **\/L\/E** - /Library/Extensions (location of modified/user installed kexts)

* **\/C\/K\/O** - /CLOVER/kexts/Other (location of modified/user installed kexts used for installation/setup)

### Note on EFI and kexts location
For the purpose of this install, I keep my kexts stored on the flash drive's EFI partition under /C/K/O. Once you have a system that boots, copy the EFI folder from the flash drive to the local EFI partition (you can use Clover Configurator to mount it). Some recommend installing kexts locally to /L/E but I haven't seen any issues with storing them on the EFI partition. I would _**highly**_ recommend keeping the flash drive as a backup in the event that you're unable to boot from the local drive due to an update or your own experimenting.

***

### ThinkPad T470 (Type 20JM) Specs
* **Part Number:** 20JMS0Q400
* **BIOS:** 1.60 (N1QET85W)
    * **Set default config**
    * **Secure Boot:** disabled
    * **Boot Mode:** UEFI Only
    * **CSM Support:** enabled

|                      | Hardware                                      | Working | Supported in            |
|:-------------------- | :-------------------------------------------- |:-------:|:-----------------------:|
| **CPU**              | Intel Core i7-6600U @ 2.6GHz (3.4GHz Turbo)   | Yes     | config.plist            |
| **Graphics**         | Intel HD Graphics 520                         | Yes     | config.plist            |
| **Memory**           | 16GB DDR4 2666Mhz (SK Hynix)                  | Yes     | - |
| **Storage**          | Intel SSD Pro 7600P 512GB NVMe                | Yes     | - |
| [**Battery**](#battery) | 3 + 3-cell (Internal + Removable)             | Yes     | ACPIBatteryManager.kext |
| [**USB**](#usb-camera-usb-ports-etc) | XHC 100-series chipset (8086:9d2f)            | Yes     | USBInjectAll.kext |
| [**SD Card Reader**](#sd-card-reader) | Realtek USB 3.0 Card Reader (0BDA:0316)       | WIP     | - |
| [**Audio**](#audio) | Realtek ALC298                                | Yes     | AppleALC.kext, layout-id 3 |
| [**Camera**](#usb-camera-usb-ports-etc)           | IMC Networks Integrated Camera                | Yes     | USBInjectAll.kext |
| [**Ethernet**](#ethernet) | Intel I219-LM                                 | Yes     | IntelMausiEthernet.kext |
| **WiFi/BT** _(M.2 2230 "A")_ | Intel Dual-Band Wireless-AC 8260 (vPro)       | No¹     | - |
| **WWAN** _(M.2 2242 "B")_ | Not installed                                 | -       | - |
| [**Function/media keys**](#function-and-media-keys) |                                            | Yes     | - |
| **Fingerprint Reader**| Validity Sensors (138a:0097)                 | No      | - |
| [**Touchpad**](#touchpad) | Synaptics UltraNav                            | Yes     | VoodooPS2Controller.kext, SSDT |
| **Trackpoint**       |                                               | Yes²     | VoodooPS2Controller.kext |
| **Keyboard backlight** |                                             | Yes     | - |
| [**Backlight**](#backlight) |                                               | Yes     | AppleBacklightFixup.kext, SSDT |
| **Touchscreen**      | AU Optronics Touchscreen (appears as Raydium Touch System)                      | No      | - |
| [**Sleep/Wake**](#power-management-sleep-wake-battery-life-etc) |                                               | WIP     | - |
| **Power Button**     |                                               | Yes     | - |
| [**Power Management**](#power-management-sleep-wake-battery-life-etc) |                                               | WIP     | ACPIPowerManagement.kext |
| **Headphone Jack**   |                                               | -       | - |
| **Thunderbolt**      |                                               | -       | - |
| **Other**            | ThinkPad Ultra Dock (90w)                     | Yes³    | - |

¹ Bluetooth appears to be detected and allows you to scan but never detects devices.\
² Trackpoint is a little too fast; I haven't looked into this so there could be improvement.\
³ Only have tested USB3, ethernet and charging; video output untested.

## Known Issues
  - Audio device still disappearing after sleep but not everytime now
  - HEVC videos do not play even though HEVC is supported (per VideoProc)
  - Battery life while sleeping is still high (10% per hour)
  - Overall battery life is only 4hr (maybe the best on dual 3-cell batteries)
 
## Hardware Setup and Configuration

> _**Info:** Everything you see below is already contained in the EFI folder. Since not all models of the ThinkPad T470 are completely the same, I'm including the methods I used to get the individual modules working which can be cherrypicked to finalize your setup._

> _**Usage:** For usage information or support on a specific kext, follow the associated link to its repository page._

### Prerequisites

   > _**Info:** These are needed to get this off the ground. Place the kexts on your EFI partition under **/CLOVER/kexts/Other** and config.plist under **/CLOVER**_
   
  - [Lilu.kext](https://github.com/acidanthera/Lilu)
  - [WhateverGreen.kext](https://github.com/acidanthera/WhateverGreen)
  - VirtualSMC.kext
    - SMCBatteryManager.kext
    - SMCLightSensor.kext
    - SMCProcessor.kext
    - SMCSuperIO.kext
    
  - [config.plist for HD 520](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/blob/master/config_HD515_520_530_540.plist)
  

### Battery

> _**Info:** The ThinkPad T470 has two batteries, an internal and external (a.k.a removable). This uses an SSDT patch to make macOS see the two batteries as one large battery._

  - SSDT-BATC-T470.aml
    - **Location:** /EFI/CLOVER/ACPI/patched
    
  - config.plist (DSDT Patches)
    - **Add via Clover Configurator:** ACPI > DSDT > Patches
     
       | comment                                    | find | replace |
       |--------------------------------------------|------|---------|
       | change Notify(\_SB.BAT0 to Notify(_SB.BATC | _SB.BAT0 | _SB.BATC |
       | change Notify(\_SB.BAT1 to Notify(_SB.BATC | _SB.BAT1 | _SB.BATC |
       | change Notify(BAT0 to Notify(BATC          | BAT0     | BATC     |
       | change Notify(BAT0 to Notify(BATC          | BAT1     | BATC     |

### Audio

> _**Info:** This was fairly straightforward. You'll need to know which codec your system has which you can find by booting from a Linux live USB. Try running `lspci | grep audio` or `aplay -l`. My ALC298 works with a layout-id of 3 and 47._

  - [AppleALC.kext](https://github.com/acidanthera/AppleALC)
  - [Lilu.kext](https://github.com/acidanthera/Lilu)
    - **Both kexts located at:** /EFI/CLOVER/kexts/Other
    
  - config.plist
     - **Option 1** - Set Device Property
       - **Add via Clover Configurator:** Devices > Properties
    
           | Properties Key | Properties Value | Value Type |
           |----------------|------------------|:----------:|
           | layout-id      | 3                | DATA       |
    
         > _**Note:** You'll need to know the PCI location. I only had two entries appear: one for the iGPU (should have key named "ig-platform-id") and another for onboard audio (PciRoot(0)/Pci(0x1f,3)._
        
    - **Option 2** - Use Boot Argument
      - **Add via Clover Configurator:** Boot > Arguments
        `alcid=47`

### Ethernet
  - [IntelMausiEthernet.kext](https://github.com/RehabMan/OS-X-Intel-Network)
    - **Location:** /EFI/CLOVER/kexts/Other
    
  > _**Note:** For some reason, I couldn't get this to work with the installer so I used a USB to Ethernet adapter. It does work after booting the first time.


### Backlight
  - SSDT-PNLF.aml
    - **Location:** /EFI/CLOVER/ACPI/patched
    
  - [AppleBacklightFixup.kext](https://github.com/RehabMan/AppleBacklightFixup)
    - **Location:** /EFI/CLOVER/kexts/Other
    
  -  Brightness Controls
     > _**Note:** My model's brightness keys (Fn+F5 & Fn+F6) uses ACPI not PS2. To determine the key name (\_Q14, \_Q15 below) please see RehabMan's [ACPIDebug.kext](https://github.com/RehabMan/OS-X-ACPI-Debug)._
     > _**Compatibility Note:** If using ACPIDebug.kext to determine keys, syslog/Console.app will not show these anymore. From Terminal run `log show --last 5 | grep ACPIDebug` instead._

     - *Requires DSDT patch; my model uses LPC**B**.KBD **not** LPC.PS2M or LPC.PS2K*

            Method (_Q14, 0, NotSerialized)  // _Qxx: EC Query
            {

                // Brightness Up
                Notify(\_SB.PCI0.LPCB.KBD, 0x0206)
                Notify(\_SB.PCI0.LPCB.KBD, 0x0286)

            }

            Method (_Q15, 0, NotSerialized)  // _Qxx: EC Query
            {

                // Brightness Down
                Notify(\_SB.PCI0.LPCB.KBD, 0x0205)
                Notify(\_SB.PCI0.LPCB.KBD, 0x0285)

            }
            
### Function and Media Keys

> _**Note:** This section is still a work in progress. I haven't had time to play around with the remaining "special keys" such as Settings (F9), Bluetooth (F10), On-screen Keyboard (F11) and Favorites (F12). I imagine these will require a similar DSDT patch as the brightness keys by determining the keys ID via ACPIDebug.kext._
- Working:
  - Mute, VolUp, VolDown, BrightDown, BrightUp
- Not Working (yet):
  - F1-F12, Mic, Display, Airplane, Settings, Bluetooth, OSK, Favorites

### SD Card Reader

> _**Coming soon:** This section is still a work in progress. I don't have a regular need for this so it's definitley the last thing on my list, however I did stumble across a kext patch that may work. I'll test this out and update._

### Touchpad

> _**Info:** For my Synaptics touchpad, I was able to get it working using RehabMan's VoodooPS2Controller.kext but tluck's SSDT-Thinkpad_Clickpad patch was required to enable the PrefPane. Multi-touch gestures are fairly limited. You can utilize two finger scrolling but swiping to go back a page in Safari, pinch-to-zoom and rotate don't work. I would also recommend disabling tap-to-click as this caused the cursor to intermittently jump to a second position on the screen if you touched the touchpad simultaneoulsy with a second finger._

> _**Note:** To enable three finger gesture for Exposé you must define the gesture as a keyboard shortcut under SysPrefs > Keyboard > Shortcuts. Under Mission Control, select Application Windows and make the gesture on the touchpad to set it._

- [VoodooPS2Controller.kext](https://github.com/RehabMan/OS-X-Voodoo-PS2-Controller)
  - **Location:** /EFI/CLOVER/kexts/Other
- SSDT-Thinkpad_Clickpad.aml
  - **Location:** /EFI/CLOVER/ACPI/patched

### USB (Camera, USB ports, etc.)
Using USBInjectAll, I was able to build a custom SSDT (SSDT-UIAC) to only enable the ports that are physically present. I disabled the internal USB port designated for biometrics since it doesn't work anyway.

### Power Management (Sleep, wake, battery life, etc.)
CPUFriend (KEXT) + CPUFriendFriend (SSDT patch) has allowed me to run the system down to 800Mhz and has prevented the CPU fan from running as often. Temps however around 45-50C when idle which is a little higher than I like but it doesn't seem to hurt anything.

***

### Post-installation work (Serial, UUID)
Make sure you generate a new serial number and UUID in Clover Configurator to avoid any conflicts with iCloud, iMessage, etc. I've removed some of my unique identifiers so they

### Special Thanks
- [okay](https://github.com/okay/t470) - some good info here which kickstarted my efforts
- [RehabMan](https://github.com/RehabMan) - base config.plist for Intel 520, various kexts, DSDT patches and SSDTs
- [tluck](https://github.com/tluck/Lenovo-T460-Clover/tree/master/DSDT.T470) - modified version of RehabMan's SSDT-BATC; SSDT-Thinkpad_Clickpad
- [ImmersiveX](https://github.com/ImmersiveX/clover-theme-minimal-dark) - dark theme for Clover
