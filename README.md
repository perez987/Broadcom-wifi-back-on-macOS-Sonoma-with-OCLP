## Get back Fenvi T919 and other Broadcom Wi-Fi on macOS 14 Sonoma thanks to OLCP

<p align="center">
<img src="img/Wi-Fi.png">
</p>

---

#### July 2024: macOS Sequoia 15 breaks OCLP root patch

OCLP root patch stopped working on macOS Sequoia beta. OCLP team has a fix, still in development, that allows you to recover Fenvi Wi-Fi on macOS Sequoia.
You can read about this in the repo [macOS 15 Sequoia beta on Z390 using OpenCore](https://github.com/perez987/macOS-15-Sequoia-on-z390-with-OpenCore) >> [Fenvi and Broadcom Wi-Fi](https://github.com/perez987/macOS-15-Sequoia-on-z390-with-OpenCore?tab=readme-ov-file#fenvi-and-broadcom-wi-fi).

#### March 2024: macOS Sonoma 14.4 breaks OCLP root patch

In macOS 14.4, Apple has modified parts of the Wi-Fi stack and OCLP root patch has stopped working so the Fenvi and Broadcom Wi-Fi are no longer operational.

To recover these Wi-Fi, 2 changes are required:

- OLCP 1.4.2 >> [Link](https://github.com/dortania/OpenCore-Legacy-Patcher/releases)
- replace `IOSkywalkFamily.kext`, current version is 1.0.0 and you have to change to version 1.1.0, also available on the OLCP GitHub >> payloads >> Kexts >> Wifi >> [Link](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi)

How to make the change?

1. replace `IOSkywalkFamily.kext`, revert OCLP root patch and reboot
2. apply root patch of OCLP 1.4.2 and reboot.
   
Other settings do not change.

---

### Broadcom Wi-Fi stops working in Sonoma

Apple has dropped support for Broadcom Wi-Fi chipset used in pre-2017 Macs:

Dependant of AirPortBrcmNIC.kext (IO80211FamilyLegacy.kext plugin)

* device-id pci12e4,43ba >> BCM43602
* device-id pci12e4,43e3 >> BCM4350
* device-id pci12e4,43a0 >> BCM4360

Dependant of AirPortBrcm4360.kext (IO80211Family.kext plugin)

* device-id pci12e4,4331 >> BCM94331
* device-id pci12e4,4353 >> BCM943224.

Many users including myself have used Fenvi T919 or Fenvi HB1200 PCI-express cards (Wi-Fi + Bluetooth combo) which have worked since at least High Sierra OOTB, without the need for additional drivers, automatically installed by macOS and recognized as Airdrop and Bluetooth by macOS.

Both Fenvi cards have the BCM4360 Wi-Fi chipset so they have stopped working in Sonoma. Bluetooth works well, as in Ventura and earlier. This is a serious inconvenience because most of the features associated with the Apple ecosystem are lost: Airdrop, Continuity, iPhone camera...

As extra information, Macs that can officially update to Sonoma have these Broadcom Wi-Fi:

* 2017 iMacPro1,1 / 2018/2019 MacBookAir8,x -> BCM4355 (pci14e4,43dc)
* 2018 MacMini8,1 / 2018/2019 MacBookPro15,x / 2019 iMac19,x / 2019 MacPro7,1 / 2019/2020 MacBookPro16,x / 2020 iMac20,x -> BCM4364 (pci14e4,4464)
* 2020 MacBookAir9.1 -> BCM4377b (pci14e4.4488).

They are chipsets soldered on the board that are not sold on the market and we cannot got them to install a Wi-Fi compatible with Sonoma in our Hack.

### Get back Fenvi Wi-Fi in Sonoma

OCLP developers have been working on this issue and have released a fix in OCLP 0.6.9 that makes Wi-Fi work again like it did in Ventura. I know it is not the ideal situation, many of us want to have the system as close as possible to a real Mac and OCLP has to apply root patches that force it to work by relaxing some macOS security rules. But what the OCLP team has achieved is a very big advance.

You have the instructions in this post (look for the **Hackintosh notes** section):

[Early preview of macOS Sonoma support now available!](https://github.com/dortania/OpenCore-Legacy-Patcher/pull/1077#issuecomment-1646934494)

Note: OCLP developers prefer that we download OCLP from the link they post, for this reason I do not put a direct link here. It is a way to take users to the original post so that it can be read, which is highly recommended.

In summary, this is what to do:

* System Integrity Protection disabled: `csr-active-config=03080000`
* AMFI disabled: `boot-args = amfi=0x80`
* `Secure Boot Model = Disabled`
* Block com.apple.iokit.IOSkywalkFamily, setting MinKernel to 23.0.0 to ensure the patch is applied only in Sonoma
* Inject 3 extensions (Kexts folder and config.plist): IOSkywalk.kext, IO80211FamilyLegacy.kext and AirPortBrcmNIC.kext (IO80211FamilyLegacy.kext plugin) in this order, setting MinKernel to 23.0.0 to ensure they are injected only in Sonoma
* Reboot and apply OCLP root patch (Modern Wireless Network).

My Wi-Fi is Fenvi T919 so I have tried this pre-release version of OCLP 0.6.9. I have followed the instructions TO THE LETTER and they have worked well. I have Wi-Fi and Airdrop in Sonoma. Please note that _khronokernel_ instructions must be followed EXACTLY. In short, this version of OCLP 0.6.9 beta works, at least for me.
<br>
<p align="center">
<img width="640" src="img/Wifi active again.png">
</p>
<br>

Don't forget to enable (`Enabled=True`) the 3 added extensions and the blocked extension.
Important: com.apple.iokit.IOSkywalkFamily block must have `Enabled=True` and `Strategy=Exclude`. Otherwise, you may have kernel panic at boot.

Incremental updates are lost with this configuration, updates can be notified from Software Update but the full installation package is downloaded and not the delta package that only contains changes from the previous version. To obtain incremental updates you have to revert the OCLP root patch and restart but you lose Wi-Fi, keep this in mind if you depend on it to have Internet access, in this case do not revert root patch before proceeding with the update.

Note: After updating, **you must ALWAYS reapply root patch** since macOS overwrites the files modified by the patch, installing original unmodified versions.

Note: OCLP has added basic support for 3rd party Broadcom chipsets, not officially supported as never shipped in any official Mac but supported by AirportBrcmFixup, these chipsets are used in Hackintoshes:

* device-id pci12e4,4357 >> BCM43225
* device-id pci12e4,43B1 >> BCM4352
* device-id pci12e4,43B2 >> BCM4352 (2.4 GHz).

---

### AMFI and AMFIpass.kext

AMFI (Apple Mobile File Integrity) was originally seen on iOS but migrated to macOS in 10.12 Sierra, possibly in 2012 when GateKeeper and digitally signed code were introduced. In short, it is a technology that blocks the execution of non signed code. It consists of 2 components:

* `/usr/libexec/amfid` service run as root from `/System/Library/LaunchDaemons/com.apple.MobileFileIntegrity.plist`
* `/System/Library/Extensions/AppleMobileFileIntegrity.kext`.

AMFI must be enabled to grant third-party applications access to privacy-relevant services and/or peripherals, such as external cameras and microphones. But, with SIP and/or AMFI disabled (a necessary condition to apply OCLP root patches) the dialog box to grant access to those applications is not shown to the user so those peripherals simply cannot be used in applications like Zoom or MS Teams, for example.

AMFI is usually enabled but it has already been seen that OCLP root patches require disabling AMFI and SIP in order to be applied. To avoid the problem of peripherals not working with third-party applications, the OCLP team has developed the **AMFIPass.kext** extension (1.3.0 release) that allows AMFI to be enabled when the system must operate with AMFI and SIP disabled, such as when using OCLP or applying root patches. This fixes the permissions issue and OCLP can apply the patches as if AMFI were disabled.

If macOS has previously given permissions to these third-party applications and then AMFI and/or SIP is disabled, these permissions are transferred and the new system maintains them. But in a clean installation they do not exist. This is the main problem that AMFIPass.kext tries to solve. Being able to root patch OCLP with AMFI enabled is just a positive side effect.

In summary, when applying OCLP root patches you can act in 2 different ways:

* with boot argument `amfi=0x80` without AMFIPass.kext. `amfi=0x80` is a bitmask that disables AMFI completely. The value 0x80 is equivalent to `AMFI_ALLOW_EVERYTHING`
* with AMFIPass.kext 1.3.0 removing `amfi=0x80` and adding `-amfipassbeta` in boot args.

The `-amfipassbeta` boot argument is provided by AMFIPass.kext to override kernel version checking, so that the extension is loaded regardless of the macOS version. This way AMFIPass can work on macOS beta for which the extension does not yet have support.

Note: current AMFIPass.kext 1.4.0 doesn't need anymore `amfi=0x80` boot arg on Sonoma.

I use AMFIPass.kext, removing `amfi=0x80`. If OCLP root patching fails due to this setting, you can temporarily disable AMFI with the boot argument `amfi=0x80`, apply the patches, reboot, remove `amfi=0x80`, and reboot again.

---

**Note**: there’s a semi-automated patch available, thanks to AppleOSX, aplicable only when installing from USB, tried with success -> [link](https://github.com/AppleOSX/PatchSonomaWiFiOnTheFly).

---

(Clic [here](https://perez987.github.io/Broadcom-wifi-back-on-macOS-Sonoma-with-OCLP/) to see this repo as GitHub pages)

