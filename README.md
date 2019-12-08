# macOS Mojave on Dell G7 7588

## Edit: There is a Catalina repo already, all the files in it can be used with Mojave. I have not had any time to explain things as on this README; but drivers and configuration files are updated, better use the ones in the Catalina repo.

As always, first things first, I will start mentioning that Nvidia cards drivers were dropped since macOS High Sierra, which means that If you want them working you will have to go back, however; Coffe Lake graphics are natively supported since macOS Mojave, that's good news so no need for much workaround with it. I have seen people on youtube as on forums claming to have macOS on the Dell G7 and they actually do but here's a thing, I checked their files, a couple videos and I can say those are probably not the best installations. No native macOS trackpad gestures, high CPU use all the time, no USB ports mapped, VoodooHDA leading to crappy sound and horrible input. Some people show their Geekbench 4 score and... well not too much to say it isn't either the best one. Hackintosh may not be that hard but you may need to read a lot to understand quite a few things and even come up with some solutions yourself for a perfectly working machine, lucky you, I have already done all that so follow the guide carefully and you will be able to experience the smoothness of macOS on this laptop.

## What you need:

* Dell G7 7588 (if newer versions of the G7 keep the same hardware you may reuse most files and look to replace if needed the CPU and maybe iGPU files)

* +8gb USB stick

* A real Mac (or working Hackintosh) to download the Mojave installer from the App Store

## What’s working?

* Brightness control

* Intel Quartz Extreme

* iMessage/FaceTime

* Keyboard with the special keys like volume control or brightness

* Ethernet

* Trackpad with native macOS gestures

* Sound (Internal speakers, internal mic, external speakers, external mic)

* Battery display and manager

* Thunderbolt 3? (not sure I got no TB3 devices to test)

* Mapped USB ports. I created two SSDTs and a single kext (use only one) found in the Optional Changes folder which map all HS (High Speed) USB ports and the SS (Super Speed) USB ports, TB3 have not even been tested, therefore; it isn't mapped either.

* Sleep

* Native CPU power management. I have created 4 energy profiles for the CPU (max power saving, balance power, performance and max performance. By following the guide you will install the balance power one which is by far my recommended one) and they all allow a drop to 800MHz (minimum frequency for the CPU according to the BIOS just like on Windows to save power) when iddle rather than the 1.3GHz that macOS natively allows so anyone can pick their favorite (If you wish to change it, it's a CPUFriendDataProvider.kext which can be found in the Post-Installation folder and Optional Changes).

![Imgur](https://i.imgur.com/yCXbsNa.jpg)

## What’s not working?

* Wi-Fi and Bluetooth (to get both working you need to buy and replace the wifi card with one which chipset is used on real macs to have a proper driver, google hackintosh compatible wifi cards if you want to buy one, Dell has some you can buy on eBay or Amazon)
Even though Wi-Fi can work with a USB dongle but you will need to spend some money on it to get a good one and have a reliable connection.

* SD card slot, no one really cares about this on Hackintosh but I think some people have got it working in different machines.

* Hibernation, it can work with HibernationFixup.kext but I am not interested as I do not use these feature.

## What you should know before starting:

* If you are not familiar installing operating systems some things may be confusing for you. Try installing Ubuntu or any easy-to-install Linux distribution and you can later come back.

* macOS and it’s EFI partition must be on the same disk and of at least 200mb, if you have Windows in another disk you will have to create an EFI partition on  the other disk which will be covered later. If your current EFI partition is not big enough you will need to either install macOS on a different disk or remove Windows, install macOS first so that the installer creates the EFI partition and then Windows.

* This guide is intended for Mojave, I may update/create one for future macOS releases, but if I do not; you should know most files will probably be usable.

* To make it easier to later move/use another bootloader (in case a better one comes up), audio and graphics are enabled by injecting device properties.

* I currently run latest BIOS version for this laptop which is the 1.10.0 and I created a SSDT file for it which contains the needed ACPI patches needed for a smooth experience. I currently rely on a single hotpatch file because a DSDT can mess when BIOS settings are changed or when it is updated. A clean-decompiled DSDT with fixed errors is included if you still think it is a good idea patching it.
If you want to modify any ACPI file make sure you are using the latest MaciASL. You can grab it from this repo: <https://github.com/acidanthera/MaciASL>

## Installation

**1) Create the efi partition on a separate drive than Windows.**

**This should only be done by those who want macOS on a separate drive than Windows**

We need to first create the efi partition on the drive macOS will be installed for that open CMD with elevated rights and enter the following code:
	
	Diskpart
	List Disk
	Select disk # (# is the number of the drive you want to install macOS)
	Create partition efi size=200
	Format quick fs=fat32 label=“System”

Now on Windows go to Create and Format Hard Drive Partitions and create a partition with NTFS format with the desired space for macOS.

**2) Create a bootable USB**
I use vanilla macOS and creating the USB is another story I won't cover, a great guide for this can be found [here](https://hackintosh.gitbook.io/-r-hackintosh-vanilla-desktop-guide/).
When installing Clover make sure to pick your USB, only check "Install Clover for UEFI booting only", "Install Clover in the ESP" will be checked and in the UEFI Drivers section check only FSInject, ApfsDriverLoader and AptioMemoryFix or OsxAptioFixDrv3 in case AptioMemoryFix is not available on newer Clover versions.
Carefully follow the guide and navigate to the downloaded USB Files folder, replace/copy the following files from the clover install folder on your USB:

* config.plist
* Copy HFSPlus.efi to the UEFI drivers folder do not copy the others.
* All the kexts found on kexts/Other to the same folder on the USB
* And finally the tonymacx86 clover theme, why this one? Well it is the one used in the config.plist as it is my preffered one.

**3) BIOS**

Enter the BIOS and change the following settings from their default values:

UEFI Boot Path Security -> Never

SATA Operation -> AHCI

Make sure Enable USB Boot Support and Enable External 

USB Ports are checked

No security for the TB3 devices or they won't probably work on macOS.

PTT Security -> Disabled

Secure Boot Enable -> Disabled

Intel SGX -> Disabled

VT for Direct I/O -> Disabled

Auto OS Recovery Threshold -> Disabled

SupportAssist OS Recovery -> Disabled

BIOSConnect -> Disabled

If you have windows currently installed you may need to recreate the pin for logging with it after this changes.

**4) Booting the installer**

Use the default UEFI USB boot entry or create your own and make it to be the main one. Restart and select Install macOS from the Clover screen and wait for the installer to load. (May even take more than 10m depending on the USB speed, be patient)

**5) macOS Installer**

**If you have ethernet connected it will ask to Set up your Apple ID, click on set up later. If not plugged just click on My Mac does not connect to the internet**
Go to disk utility and go to erase to format the created NTFS partition of the desired size for macOS with APFS named Macintosh HD (diskutil may exit), reopen and wait for it to finish erasing and creating the partition.
Now complete the installation process with the desired language and stuff it wil take some time and restart a couple times do NOT touch anything nor shutdown the computer so make sure it is plugged in. After the system has been installed launch macOS from the USB with the Clover bootloader, then proceed to the final step of installing the correct drivers, clover bootloader to a drive so the USB is not needed to boot and setting the proper configurations to get the system running at best, because most things won't work right now, only the essentials to boot.

**6) Post Installation**

Login and set up your user.
**You can plug the ethernet but only login to you Apple ID after step 7**
We first need to install Clover to the Macintosh HD drive, launch the clover install package and make sure you click on Change "Install Location" and set it for the internal drive, check same things as when creating the USB but also check "Install RC Scripts on Target Volume". Navigate to the POST INSTALL/Clover folder,copy/replace the config.plist, the SSDT-G7.aml from ACPI/Patched the kexts from the Other folder and finally the tonymacx86 theme.
In the POST INSTALL folder there will be a another one named Kexts to /L/E/ which means that you should install the kernel extensions in it to /Library/Extensions/, to do that use Kext Wizard, KextBeast (my preffered) or any kext installer. Better leave /System/Library/Extensions/ clean and only install kexts to /L/E/.

Set up the ComboJack fix from the [ComboJack](https://github.com/Andrw0830/ComboJack) repo or there will be a static noise with/without headphones plugged and will lead to high CPU frequency all time. The VerbStub.kext is already in the POST INSTALL/Clover/Others folder, just copy all kexts to the Other folder as stated in step **6**.
Reboot after installing it.

**7) iMessage and FaceTime**

To get both working we need to fake the Serial Number, Board Nuber, BIOS Version... for that follow this easy [guide](https://www.tonymacx86.com/threads/an-idiots-guide-to-imessage.196827/).
Try using Clover Configurator which will generate some serials and stuff to simplify things. Choose MacBookPro15,2 on the mac list for energy purposes.

**8) Sleep**

Sleepimage is saved by default to RAM and disk, the system will first sleep and later hibernate (hibernatemode 3). 
To avoid hibernation and improve sleep by saving sleepimage only to RAM enter the following command in terminal:

	sudo pmset -a hibernatemode 0

If it is not sleeping disconnect the AC adapter from the laptop and verify hibernatemode is set to 0 and no apps are mentioned to be preventing sleep by entering into terminal:

	sudo pmset -g

Lastly use Intel Power Gadget to check for CPU power management, set Clover to be the main boot entry and enjoy you new ~~Mac~~ Hackintosh!

**Appreciate the work and want to donate?** [PayPal](<https://www.paypal.me/juanvasquezcastro>)

## Credits

* Apple for macOS
* The Clover team
* VoodooI2C team
* Rehabman
* vit9696
* PMheart
