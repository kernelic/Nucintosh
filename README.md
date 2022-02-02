# XCY Intel 8xxxU Hackintosh build
This is fork of [Nucintosh repo](https://github.com/zearp/Nucintosh) with some changes for AliExpress XCY 8th gen computers (like XCY X41 [2020](https://aliexpress.com/item/4001015107317.html)/[2021](https://aliexpress.com/item/4000703339500.html) ONLY WITH i3/5/7 8xxxU and two LAN ports). Compatible with macOS Mojave, Catalina, Big Sur and Monterey.

![macOS Monterey](https://github.com/zearp/Nucintosh/blob/master/Stuff/Monterey.png?raw=true)

## Details
* Works with macOS *Catalina*, *Big Sur* and *Monterey*
* OpenCore bootloader with the following kexts:
  - Lilu
  - VirtualSMC
  - WhateverGreen
  - AppleALC
  - IntelMausi
  - RealtekRTL8111
  - NVMeFix
  - CPUFriend
  - BlueToolFixup -- fixes bluetooth in Monterey
  - AirportItlwm

## Index
* [Installation](#installation)
* [Post install](#post-install)
* [Updating](#updating)
* [Apple and 3rd party wifi/bt](#apple3rd-party-bluetooth-and-wifi)
* [Intel wifi/bt](#intel-bluetooth-and-wifi)
* [Native bt dongle](#natively-supported-bluetooth-dongle)
* [What doesn't work?](#not-workinguntested)
* [Todo](#todo)
* [Credits](#credits)

## Installation
+ BIOS Changes
```
Devices -> USB -> USB Legacy -> Disabled
Power -> Wake on LAN from S4/S5: Stay Off
Boot -> Boot Configuration -> Network Boot: Disable
Boot -> Secure Boot -> Disable
```
+ Download macOS from the App Store and create a USB installer with *[createinstallmedia](https://support.apple.com/en-us/HT201372)* on macOS (real mac/hack or vm) or use [gibMacOS](https://github.com/corpnewt/gibMacOS)\*
+ Download the EFI folder [here](https://github.com/zearp/Nucintosh/releases) or download/clone the complete repo for latest builds
+ Edit config.plist with [ProperTree](https://github.com/corpnewt/ProperTree) and change the following fields;
```
PlatformInfo -> Generic -> MLB
PlatformInfo -> Generic -> ROM
PlatformInfo -> Generic -> SystemSerialNumber
PlatformInfo -> Generic -> SystemUUID
```
Generate new serials with [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS). The ROM value is your ethernet (en0) mac address ([more info](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html#fixing-en0)).
+ Copy the EFI folder to the EFI partition on the USB installer
+ Clear NVRAM from the OpenCore picker
+ Install macOS

\* Installers made with GibMacOS on Windows and Linux require a working internet connection as it uses the recovery image only, it then downloads the full installer from Apple. The *createinstallmedia* script makes an offline installer.

> Note: OpenCore doesn't always select the correct partition in the menu when installing. You will only boot into the installer once, do your formatting and have the installer copy all it needs to the internal disk. From that point onwards always select the internal disk from the menu. The name might change during the installation, but it shouyld be easy to spot as it won't have an "external" label.

## Post install
- Check if TRIM is enabled, If not run ```sudo trimforce enable``` to enable it
- Don't forget to copy the EFI folder from the installer's EFI partition to the internal disk's EFI partition. This is needed to boot from the internal disk. You can use [EFI Agent](https://github.com/headkaze/EFI-Agent) to easily mount EFI partition.

Finally make sure sleep works properly. You can skip some of these but it will make your machine wake up from time to time. Same as real Macs.
```
sudo pmset standby 0
sudo pmset autopoweroff 0
sudo pmset proximitywake 0
sudo pmset powernap 0
sudo pmset tcpkeepalive 0
sudo pmset womp 0
sudo pmset hibernatemode 0
```
The first two and last need to be ```0``` the rest can be left on if you want.

- Proximity wake can wake your machine when an iDevice is near
- Power Nap will wake up the system from time to time to check mail, make Time Machine backups, etc, etc
- Disabling TCP keep alive has resolved periodic wake events after setting up iCloud, just disabling Find My wasn't enough.
- Womp is wake on lan, which is disabled in the BIOS as it (going by other people's experience) might cause issues. I never use WOL, if you do use WOL please try enabling it in the BIOS and leave this setting on, the issues might have been due to bugs that haven been solved by now. Let me know if it works or not.
- Hibernate is sometimes set to 3 in my testing. It could be possible to get hibernation to work by using [HibernationFixup](https://github.com/acidanthera/HibernationFixup).

With hibernation disabled you can delete the sleepimage file and create an empty folder in its place so macOS can't create it again, this saves some space and is optional.
```
sudo rm /var/vm/sleepimage
sudo mkdir /var/vm/sleepimage
```


At this point you can also enable FileVault if you want to encrypt your disk. The config is setup to support this and it works flawlessly, to get a nicer boot experience you can remove the verbose boot flag ```-v```in the config and also set ```ShowPicker``` to false. To get the OpenCore picker/menu again hold down the *alt* key when booting.

That's all!

## Updating
Updating is easy, first copy the MLB/ROM/SystemSerialNumber/SystemUUID values from your current config to a text file then delete the whole EFI folder and replace it with the latest release/clone from this repo. Copy your PlatformInfo fields from the text file into the new config. Unless you made other changes this is all thats needed.

## Apple/3rd party bluetooth and wifi
For both 1st and 3rd party you will need a [supported](https://dortania.github.io/Wireless-Buyers-Guide/) wifi/bluetooth combo card and an adapter (see below) to convert it to M key. As far as I know compatible M key combo cards don't exist.

3rd party cards will need these kexts: [AirportBrcmFixup](https://github.com/acidanthera/AirportBrcmFixup) + [BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM), read the instructions on the repo's and you'll be up and running in no time. For some cards you may need to create an entry under devices in the config that disables ASPM, this only needed if you have issues with sleep.

1st party is my preferred option. Grab an Apple 6+12 pin to m.2 M-key [converter card](https://dortania.github.io/Wireless-Buyers-Guide/Airport.html) and go native with something like the BCM94360CS2. Please note the number of antenna connectors. Some have more than 2, so you'll have to add some antenna's and maybe even mod your case. Though there is some room under the plastic lid, it can fit internal antennas like [this](https://ae01.alicdn.com/kf/HTB1AAiAKVXXXXc4aXXXq6xXFXXX4/2pcs-Internal-Antennas-40cm-15-7-inches-IPEX-MHF4-2-4-5G-wifi-antennas-for-BCM94352Z.jpg). The lid can be removed with some strategic force and there's a hole to route the wires trough too. I would use those and leave the standard antenna's connected to the Intel module. They're very cheap and the antenna connectors on the Intel module are very fragile.

One big plus of going native is that you gain HID-proxy. This means that when there is no OS running the Airport card will proxy any paired HID bluetooth devices to the machine as usb devices. This means you can enter the BIOS or boot menu using the bluetooth keyboard and mouse. This is not a feature you will find on many other cards, including the the one Intel put in here. Even expensive bluetooth cards often can not do this. But Apple has added it even in the cheap BCM943224PCIEBT2 Airport card.


Speaking of the $10 BCM943224PCIEBT2, I personally tested that card on XCY X41 (2020) and it still works fine in Catalina by setting ```Kernel -> Patch -> 0``` to true. Big Sur and Monterey will need the patch disabled and [AirportBrcmFixup](https://github.com/acidanthera/AirportBrcmFixup) added with boot flags ```brcmfx-driver=2 brcmfx-country=#a``` instead. You can also add your card as a device in the configs DeviceProperties section and set the options there, for example;
```
<key>PciRoot(0x0)/Pci(0x1C,0x4)/Pci(0x0,0x0)</key>
 	<dict>
 		<key>AAPL,slot-name</key>
 		<string>Internal@0,28,4/0,0</string>
		<key>device_type</key>
 		<string>Network controller</string>
 		<key>model</key>
 		<string>Apple Airport BCM43224 802.11a/b/g/n</string>
 		<key>brcmfx-country</key>
 		<string>#a</string>
 		<key>brcmfx-driver</key>
 		<string>2</string>
 	</dict>
```

Make sure you check if the PciRoot/slot-name paths are correct, you can find them in IOreg or Hackintool. Also make sure the AirPortBrcm4360_Injector.kext plug-in that will be added if you use the ProperTree snapshot command is disabled. It is part of AirportBrcmFixup but can cause Monterey boot-up to stall and wifi not working properly (shows as disabled).

Some sellers on AliExpress have converter cards that already have [the small 1.25mm pitch jst](https://github.com/zearp/Nucintosh/blob/master/Stuff/NUC8-m2adapter.jpg?raw=true) connector on it. It connects to one of the two internal usb ports. They usually list them as NUC8 compatible and cost a bit more than other converter cards.

Those other cards (and 3rd party ones) do not come with this connector so you'd have to make your own. Cheaper eBay card came with a cable with standard internal usb header and a cable without any plugs so you can attach your own. Check the listing carefully before ordering. Also make sure it converts to M key and once you have it that the spacing pillar is in the correct position. Don't short the poor Airport out.

- All internal usb ports are already mapped in the USBMap.kext, if you made your own map you'll need to make a new map if you use the internal usb headers
- When using a 1st or 3rd party combo card you need to disable both bluetooth and wifi in the BIOS and also remove any Intel related bluetooth and wifi kexts
- You will also need to remove the config block for HS07 used by the onboard Intel wifi/bt card (this was HS10 in previous usb portmaps) from Info.plist inside USBMap.kext, without this step bluetooth won't work (properly) after sleep. On 1st party cards it gets "stuck" in HID-proxy mode; bluetooth mouse and keyboard may still work but not optimally and laggy.

You'll also want to set your region to ```#a``` as it allows for full 80mhz channel width on ac cards. It might not be 100% legal depending on where you live. I've used this method on a few DW1820A cards and the speed increase was pretty amazing. This method may also apply when using real Apple cards, you will need add [AirportBrcmFixup](https://github.com/acidanthera/AirportBrcmFixup) on 1st party cards. To change the region simply add the following boot flag ```brcmfx-country=#a```. Make sure your router also supports 80mhz channel width and before doing anything hold down alt while clicking the wifi icon to check the current channel width.

One last thing to remember is that waking the machine from sleep using bluetooth devices will not work. This is due to power being cut to the module. The module does start itself up very fast. By the time screen wakes up bluetooth devices are already reconnected. There is way to bypass this but it includes either modding your adapter card or [making your own](https://github.com/BbIKTOP/M.2-key-M-to-wifi). [@zearp](https://github.com/zearp) asked some sellers on AliExpress to produce this card but didn't have any luck. If you can make it or know a seller who's willing to make it please let [@zearp](https://github.com/zearp) know.

## Intel bluetooth and wifi
+ Wifi works and can be managed using native tools, speeds are still slow but connections are stable
+ Bluetooth don't works (TODO).

## Natively supported bluetooth dongle
I often use these cheap dongles from [eBay](https://www.ebay.co.uk/itm/1PCS-Mini-USB-Bluetooth-V4-0-3Mbps-20M-Dongle-Dual-Mode-Wireless-Adapter-Device/324106977844) that work in macOS out of the box. When going this route don't forget to disable the Intel bluetooth kexts in the config and also disable bluetooth in the BIOS when using a dongle. You will also need to map the port it connects to as internal else sleep will be dodgy. You can do this easily by setting the port type to 255 in the USBMap.kext info.plist file. You can find the port identifier (example HS03) with Hackintool. Because power isn't cut when entering sleep you can wake the machine up with bluetooth devices.

## Not working/untested
+ ~~Card reader~~ Fixed. Added new USBMap kext.
+ ~~Thunderbolt (Type-C)~~ Fixed. Added new USBMap kext.
+ Bluetooth
+ AirDrop (because only Wifi works, not Bluetooth)
+ You tell me!

## Todo
+ Fix Bluetooth and AirDrop (OpenIntelWireless Bluetooth kexts don't work, i would like to learn this issue)

## Credits
* [@zearp](https://github.com/zearp) for that incredible guide and main repo: https://github.com/zearp/Nucintosh
+ https://github.com/acidanthera
+ https://github.com/OpenIntelWireless
+ https://github.com/Mieze
+ https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html
+ https://github.com/osy
+ Many [random](https://github.com/Rashed97/Intel-NUC-DSDT-Patch/commit/47476815b52f8e4c97e8f85df158c9ab1b6ecedd) repos [with](https://github.com/honglov3/NUC8I7BEH) information [and](https://github.com/sarkrui/NUC8i7BEH-Hackintosh-Build) research [that](https://github.com/mbarbierato/Intel-NUC8i3BEH) [@zearp](https://github.com/zearp) and me have [forgotten](https://github.com/honglov3/NUC8I7BEH) about.
