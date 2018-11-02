Vanilla macOS on T430, almost perfectly with High Sierra 10.13


Credits goes to ThiagoSchetini & Rehabman, thanks bunches!


#### WORKING =)
* Intel Hyperthread (4 cores) - I have the i5-3320M.
* Intel TurboBoost (up to 3.2 Ghz)
* Intel SpeedStep
* CPU HW Monitor (temps, clock and power)
* RAM DDR3 at 1600mhz
* Intel HD4000 1536MB
* Brightness (FULL Range + Fn Brightness Keys)
* Display Port Out: FullHD + Audio
* Battery Status
* Wi-Fi (I use an Atheros AR5BHB92 [rebranded](https://github.com/ThiagoSchetini/AR9285-rebranding))
* Ethernet
* Sleep
* Display turns on automatically on wake/instant wake
* USB 3.0 Full Speed
* Audio ALC269
* Webcam + Mic
* Keyboard + TouchPad <- switch mapping of < and $.
* Bluetooth: bought and installed this [BT daughter card](https://www.aliexpress.com/item/Free-shipping-New-original-bluetooth-module-4-0-for-Lenovo-Thinkpad-X220-X230-T420-T430-T530/1640038938.html) worked out of the box. *Sadly no Handoff - yet*


#### NOT WORKING ...
* On Shutdown, sometimes need to remove USB devices otherwise system will restart.
* SSD Trim (Works, but I have disabled it.)
* VGA output (No support.)
* SD Card Reader <- never tested.



#### Creating the installer
	- You'll need thumbdrive of at least 8 GB, jHFS+ formatted.

    - Use Unibeast to create the installer. Select UEFI and no ATI/nVidia.

	- After creating the USB Installer go to EFI/CLOVER/drivers64UEFI/
		Remove everything and put the 4 files this repository (add the apfs.efi if need be)

	- Put any kexts in EFI/CLOVER/kexts/10.12 etc. into ./other/ and copy the kexts from this
	  repository into ./other/ as well.
		- Thus removing all kexts folders except ./other/


#### Install macOS
    - You'll need a mouse and external keyboard plugged in (for the install process).

	- If you get stuck on boot disable 'InjectIntel' graphics.

##### Install on Fusion Drive (Optional)
If you have installed an SSD and a HDD in your T430, you can gain the advantages of using a fusion drive. Follow these steps to install on a fusion drive rather than a "regular" drive.

	- Setup Fusion drive according to guides on the internet.

	- Install macOS on external HDD.

	- Restore HDD installation to Fusion Drive with Disk Utility on the install disk.

	- *Alternatively* (never got this to work) do `/Volumes/"Image Volume/Install macOS High Sierra.app"
    	/Contents/Resources/startosinstall --volume the_target_volume--converttoapfs NO --agreetolicense`
    	from Terminal instead of regular install.

	- Boot into new installation on Fusion Drive.


#### Show hidden files (optional, but helpful)
	- After first boot, on Terminal:
    	defaults write com.apple.finder AppleShowAllFiles YES
		killAll Finder


#### Install Clover

	- Install “clover.pkg” (found in repo/tools folder)
    	Check 'Install for UEFI Booting Only'
		This will automatically check 'Install Clover in the ESP'
		Check Drivers 64 UEFI (IMPORTANT! Do only check OsxAptioFix2Drv-64)

	- Now you can remove your Installer Pen Drive (you have boot)


#### Configure EFI/Clover
	- config.plist: use the one from this repository
        take from low-resolution-config.plist folder for 1366x768 display
		take from high-resolution-config.plist folder for 1600x900 or + display

	- /drivers64UEFI
		use the folder from this repository (with 4 files only!)

	- /kexts
		Remove everything except other (leave it empty)

	- /ACPI/patched:
		(OPTION 1 Difficult - but recommended) Make your own SSDT and DSDT:
			-> First you'll need the vanilla .aml generated by your UEFI (BIOS)
				Enter in the clover boot menu and press F4 and FN+F4
				That’s all. It’s gonna be on EFI/CLOVER/ACPI/origin

			-> Them disassembly correctly only the DSDT.aml and DSDT’s.aml (0, 1, 2 etc…)
				please, don’t open directly on MacIASL!
				use iasl on terminal (look at tools folder/iasl howTo.txt)

			-> patch your DSDT.dsl with all the code from file “…-dsdt-patch.txt” using MacIASL
				it includes only the necessary code to Thinkpad T430
				* choose low resolution for 1366 x 762
				* choose high resolution for 1600 x 900 or +

			-> manually last patch for screen turn on after wake (Warning! It’s a manual change)
				on your .dsl find for “Return (WAKI)”
				On the last two method calls there is:
					\_SB.PCI0.LPC.EC.LED(Zero, 0x80)
    					\_SB.PCI0.LPC.EC.LED(0x07, 0x00)
				change for (carefull with identation, must be perfect):
					\_SI._SST (One)
        				\_GPE._L1D ()

			-> Save from MacIASL as binary (.aml) and paste on ACPI/patched inside clover UEFI

			-> Generate your own SSDT (Power Management for your processor)
				https://www.tonymacx86.com/threads/quick-guide-to-generate-a-ssdt-for-cpu-power-management.177456/
			-> Put the SSDT.aml on the same folder of DSDT: ACPI/patched inside clover UEFI


		(OPTION 2 Easy One - Not recommended) take my patched files .aml:
	 		->take ONLY the .aml files from “pached ACPI dsl’s” folder
				* choose only one SSDT.aml: “i5 3230M” or “i7 3632QM”
				* if you have another processor make your own SSDT or leave it empty

			-> Put the .aml files on the folder: ACPI/patched inside clover UEFI

#### Kexts

	- Make a beckup of AppleBacklight.kext (Brightness)
		Why? for future updates you need to reinstall the original and then reinstall the pached

		** Warning!, the backlight kext inside kexts folder is patched for T430 brightness full range control

	- Now, install all the kexts from the folder using “Kext Utility.app” (look inside tools folder)

#### Voodoo Extra Files (for Keyboard)

	- Go to the voodoo's' folder with the terminal
		sudo cp org.rehabman.voodoo.driver.Daemon.plist /Library/LaunchDaemons
		sudo cp VoodooPS2Daemon /usr/bin


#### Restart at get audio working

	- After restarting you need to flush the kexts:
		sudo touch /System/Library/Extensions && sudo kextcache -u /
	- Restart again to hear the audio working!

	- Repeat the process if nescessary
		sudo touch /System/Library/Extensions && sudo kextcache -u /
	- Restart Again!

	* If you keep having problems, try disabling the option 'No Cache' in config.plist and restart again.


#### HWMONITOR
	- Put the HWMONITOR from tools folder in Applications and open it, you should read power, clock, and temps of your CPU


#### Disable Hibernation
	-> Disable Hibernation and related options
        sudo pmset -a hibernatemode 0
        sudo rm /var/vm/sleepimage
        sudo mkdir /var/vm/sleepimage
        sudo pmset -a standby 0
        sudo pmset -a autopoweroff 0

**And that's it! You're done. (Hopefully.)** =)


#### ABOUT THIS DSDT CODE

    -> ThiagoSchetini modified some original patches from Rehabman:

			[modified] Lenovo X220
				-> added T430 sleep leds
				-> cleaned extra code

			[modified] Brightness fix (HD3000/HD4000)
				-> included FN brightness keys (T430 only)

			[usb] USB3_PRW 0x0D (instant wake)
				-> to have sleep working (original Rehabman code)

			[audio] Audio Layout 28
				-> to work with the ALC269 kext (original Rehabman code)

			[sys] IRQ Fix
				-> to enable audio (original Rehabman code)

			[igpu] Low Resolution (or High)
				-> to enable HDMI display port audio out

			[manual] Code change on .dsl to turn on screen automatically on wake


#### COMPLETE KEXT FLUSH (recomended after remove a kext)

	-> repair permissions:
        sudo /usr/libexec/repair_packages --repair --standard-pkgs --volume /

    -> rebuild kext cache
        sudo rm -r /System/Library/Caches/com.apple.kext.caches
        sudo touch /System/Library/Extensions
        sudo kextcache -update-volume /

	-> rebuild cache short form
        sudo touch /System/Library/Extensions && sudo kextcache -u /


#### GENERATE VANILLA CONFIG.PLIST (if you need it)

	on /usr/local/bin/clover-genconfig > config.plist
	execute and take the xml

#### HOW TO FLASH T430 BIOS (if you want to install an 1300AC Wi-Fi …)

https://github.com/bibanon/Coreboot-ThinkPads/wiki/xx30-BIOS-Whitelist-Removal

## Updates

### Updating from Sierra to High Sierra (10.12.6) (Windows edition)
	- Downloaded full macOS 10.13.6 from App Store
	- Copy latest apfs.efi to /EFI/Clover/drivers64UEFI
	- Backup all kext and update with Install High Sierra.app
	- Disable default volumes, set timeout 10s (If necessary)
	- If post update boot fails:
		Use a windows machine
		Download Clover Iso [here](https://sourceforge.net/projects/cloverefiboot/files/Bootable_ISO/)
		Extract with 7zip
		Use [rufus](https://rufus.akeo.ie) to burn iso to usb
		Delete all drivers64UEFI in clover and replace drivers64UEFI in this repo
		Delete all kext except Other and paste all kext in this repo
		Replace config.plist with the one in this repo
		Copy apfs.efi to /EFI/Clover/drivers64UEFI
		Boot to usb and select volumes update
		Do not enable File Vault during setup.
	- Flushed kext cache to get audio working again.
	- sudo touch /System/Library/Extensions && sudo kextcache -u /
	- Reboot :D

### Updating from 10.13.0 to 10.13.1
	- Downloaded full macOS 10.13.1 from App Store.
	- Copied kexts and apfs.efi to their respective places in EFI.
	- Installed macOS 10.13.1
	- Flushed kext cache to get audio working again.
		sudo touch /System/Library/Extensions && sudo kextcache -u /
	- Remove kexts from EFI.
	- Reboot and voilà. =)

### Updating from 10.13.1 to 10.13.2
	- Downloaded newest FakeSMC.kext -> installed in bootloader.
	- Ran `softwareupdate -i -a`.
	- Rebooted after installation and chose the Install option in Clover.
	- Flushed kext cache to get audio working again with KextUtility.
		- At the same time installed the new FakeSMC.kext.
	- Reboot and voilà once more. =)

### Updating from 10.13.2 to 10.13.2 Supplemental Update
	- Downloaded newest FakeSMC.kext -> installed in bootloader.
	- Ran `softwareupdate -i -a`.
	- Rebooted after installation and chose the Install option in Clover.
	- And voilà! =)

### Updating from 10.13.2 Supplemental Update to 10.13.3
	- Ran `softwareupdate -i -a`.
	- Rebooted after installation and chose the Install option in Clover.
	- Flushed kext cache to get audio working again with KextUtility.
	- Reboot and voilà! =)
	- Kept FakeSMC.kext in EFI

### Updating from 10.13.3 to 10.13.3 Supplemental Update
	- Ran `softwareupdate -i -a`.
	- Rebooted after installation and (automatically) chose the Install option in Clover.
	- Everyting (including sound) worked without a kext cache flush.
	- And voilà! =)

### Updating from 10.13.3 Supplemental Update to 10.13.4
	- Ran `softwareupdate -i -a`.
	- Installed new apfs.efi in /EFI/CLOVER/drivers64UEFI/
	- Rebooted 2x after installation and automatically chose the Install option in Clover.
	- Flushed kext cache to get audio working again with KextUtility.
	- Reboot and voilà! =)
	- Kept FakeSMC.kext in EFI

### Updating from 10.13.4 to 10.13.4 Security Update
	- Updated apfs.efi
	- Ran `softwareupdate -i -a`.
	- Rebooted after installation and (automatically) chose the Install option in Clover.
	- Everyting (including sound) worked without a kext cache flush.

### Updating from 10.13.4 Security Update to 10.13.5
	- Installed new apfs.efi in /EFI/CLOVER/drivers64UEFI/
	- Ran `softwareupdate -i -a --restart`.
	- Rebooted 2x after installation and automatically chose the Install option in Clover.
	- Removed BrcmBluetoothInjector.kext from /S/L/E
	- Added and updated the following bluetooth related kexts in /S/L/E
	    - BrcmFirmwareRepo.kext (updated)
	    - BrcmNonPatchRAM2.kext (updated)
	    - BrcmPatchRAM2.kext (added)
	- Flushed kext cache to get audio working again with KextUtility.
	- Reboot and voilà! =)

### Updating from 10.13.5 to 10.13.6
	- Installed new apfs.efi in /EFI/CLOVER/drivers64UEFI/
	- Ran `softwareupdate -i -a --restart`.
	- Flushed kext cache to get audio working again (with KextUtility).
	- Reboot and everything works againg. =)

### Updating from 10.13.6 to 10.14.0 Mojave
	- Installed new apfs.efi in /EFI/CLOVER/drivers64UEFI/
	- Updated to newest kexts
	- Moved all custom kexts from /S/L/E to /EFI/CLOVER/kexts/Other for easier future upgrades.
	- Copied old official IO80211Family.kext and IO80211FamilyV2.kext from HS S/L/E to Mojave S/L/E - to get Wi-Fi working (specific to my Wi-Fi adapter (Atheros AR5BHB92)).
	- Downloaded and ran macOS 10.14.0 installer from App Store
	- Rebooted after installation and (automatically) chose the Install option in Clover.
	- If sound does not work (which was my case), you need Lilu and AppleALC kexts, they can be found in the kexts folder.
	- Flush kext cache and see if audio is working again (with KextUtility).
	- If not (I did not need to do this):
		- Try adding -alcbeta -lilubeta to custom flags in Clover and set Devices -> Audio -> 28
		- Flush kext cache to get audio working again (with KextUtility).
	- Reboot and everything should work again. =)
	
### Updating from 10.14.0 to 10.14.1
	- Update AppleALC.kext to verstion 1.3.3 in /EFI/CLOVER/kexts/Other
		- get kext here: https://github.com/acidanthera/AppleALC/releases
	- Install old official IO80211Family.kext and IO80211FamilyV2.kext from HS S/L/E to Mojave S/L/E with KextUtility.
		- specific to non working Atheros adapters (e.g.: AR5BHB92)
	- Install update via Software Update.
	- Done!

