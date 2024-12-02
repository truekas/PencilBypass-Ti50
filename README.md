
<img src="https://github.com/truekas/PencilSharpener/blob/main/src/Logo.png?raw=true" alt="Pencil Sharpener"/>

> [!WARNING]  
> DURING THE EXPLOIT, MAKE SURE THE CHROMEBOOK STAYS PLUGGED IN TO A CHARGER AT ALL TIMES (EXCEPT WHILE BRIDGING THE WP PINS). THE SYSTEM WILL BRICK IF THIS IS NOT FOLLOWED.

> [!IMPORTANT]
> This writeup is for educational purposes only. Do not use this exploit on your organization's systems without permission. Remember that your orginization's Chromebook is not your personal device. Nobody is responsible for any touble that happens because of this exploit.

## Hey There!
If you're looking at this writeup, this exploit has been patched. Recently, Google rolled all shim keys on all Ti50 boards during the update to Kv5, making it impossible to boot into Sh1mmer or use this exploit. We decided to release this writeup after the patch, to ensure that no damage is caused to the devices of schools and companies. 

If you are an administrator, we recommend that you set the `DeviceMinimumVersion` in Google Admin to ensure that all new Chromebooks have been patched. We also recommend that you monitor the policy sync dates for all users on Cr50 Chromebooks to monitor exploitation.

## Introduction 
This writeup demonstrates how Google's tsunami enrollment patch, released on v114, can be bypassed on newer mainboards with the Ti50 chip on Kv4. The exploit uses a modified version of the original pencil exploit to bypass FWMP and VPD and prevent the system from bricking.

Google can patch this exploit if they decide to change RMA shim keys on Kv5, which is supposed to be released in a few months. 

## The Exploit

What you need:
- A pencil or conductive material (chip clip recommended) 
- A screwdriver and an ESD bracelet to prevent damage to the device
- A sh1mmer image for your system flashed to an SD card or USB stick
- Working device charger
- A ChromeOS recovery USB for v124

First, fully power off and unplug your device, flip it over, and open the back to gain access to the mainboard.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/6.png?raw=true" alt="6.png"/>

Then, disconnect the battery from the mainboard and locate your device's Flash Chip, usually near the mainboard's battery socket and covered in black tape (different for every device). Now, bridge pins 3 and 8 with your conductive material or chip clip.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/2.png?raw=true" alt="2.png"/>

Afterward, re-insert your charger AND KEEP IT PLUGGED IN while pushing `esc + refresh + power` to enter the device recovery menu. Then press `ctrl + d` and as soon as the screen goes black, press the keys to re-open the recovery menu. 

Insert your sh1mmer USB and then choose to boot from it. If it is the first time you have attempted this, it should give you a `no valid image` error because you have keyrolled. If you got this error, follow the steps at the bottom of the document to unbrick.

Now re-open the recovery menu and try to boot back into sh1mmer. Choose `utilities > unenroll` it should give an error. Open the bash console WHILE MAKING SURE THE PINS ARE STILL BRIDGED and run:
```
flashrom --wp-disable
/usr/share/vboot/bin/set_gbb_flags.sh 0x80b3
flashrom --wp-enable
```
Hit `esc + refresh + power` to go back into the recovery menu and boot onto your v124 recovery USB. This is possible because on v130 and below, kernver is set to 4 which allows downgrading up to v120.

After the recovery process is complete, choose to `boot from internal disk` on the menu. Then switch to the Vt2 console by pressing  `ctrl + alt + f2` and run:
```
tpm_manager_client take_ownership
cryptohome --action=remove_firmware_management_parameters
crossystem dev_boot_usb=1
```

Now make sure that the battery is re-inserted on the mainboard and run `gsctool -a -o`. Follow the prompt to push the power button and the system should automatically reboot. When the device turns back on, re-open the recovery menu and re-enable devmode.

Now all you have to do is go back to the Vt2 console, run `gsctool -a -I AllowUnverifiedRo:always`, and the device should be unenrolled.


## Fixing Rolled Keys
After downgrading to v124, some systems will keyroll while in recovery mode and prevent users from booting into the OS or Sh1mmer. This is because the recovery kernel data key will fail to validate the kernel during boot. As the recovery key is in a read-only portion of the system, it would not get overwritten when the kernel signature was changed during the downgrade.

This issue is fixable on Nissa boards by flashing the correct keys to the system to continue the boot process. Here's how to do it:

Go into VT2 (`CTRL+ALT+F2`). If you can't get to VT2, you will have to use a flash programmer (ch341a) or find some other way to get a root shell. You will need `vboot_utils` and `curl` (preinstalled on ChromeOS).

Bridge pins 3 and 8 on the flash chip, and run these commands in your shell:
```bash
flashrom --wp-disable # if applicable
curl -LO https://raw.githubusercontent.com/truekas/PencilSharpener-Kv4/refs/heads/main/src/unrolled_nissa.bin
futility gbb -s --recoverykey unrolled_nissa.bin # add -p if using a programmer
flashrom --wp-enable
```
It is possible to generate the correct recovery file by using a ch341 programmer and [connecting to the device](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html#prepping-to-flash) or using vt2 to run `futility gbb --recoverykey file.bin` and obtain the correct file.

## Citations 
[Breaking chromeOS's enrollment security model: A postmortem](https://blog.coolelectronics.me/breaking-cros-6/)
<br>
[Disabling Firmware Write Protection | MrChromebox.tech](https://docs.mrchromebox.tech/docs/firmware/wp/disabling.html)
<br>
[Unbricking/Flashing with a ch341a USB programmer | Chrultrabook Docs](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html)
<br>
[Verified Boot](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot/)
<br>
[Firmware Boot and Recovery](https://www.chromium.org/chromium-os/chromiumos-design-docs/firmware-boot-and-recovery/)
<br>
[Verified Boot Data Structures](https://www.chromium.org/chromium-os/chromiumos-design-docs/verified-boot-data-structures/)
<br>
[CrOS EC (Embedded Controller) - Google Security Chip (GSC) Case Closed Debugging (CCD)](https://chromium.googlesource.com/chromiumos/platform/ec/+/cr50_stab/docs/case_closed_debugging_gsc.md)
<br>
[hdctools: Chrome OS Hardware Debug & Control Tools - Closed Case Debug (CCD)](https://chromium.googlesource.com/chromiumos/third_party/hdctools/+/HEAD/docs/ccd.md)
<br>
[CrOS EC (Embedded Controller) - Google Security Chip (GSC) Case Closed Debugging (CCD)](https://chromium.googlesource.com/chromiumos/platform/ec/+/fe6ca90e/docs/case_closed_debugging_cr50.md)
<br>
[Read-only firmware unlock on 2023+ devices](https://www.chromium.org/chromium-os/developer-library/guides/device/ro-firmware-unlock/)
<br>
[Firmware Write Protection on ChromeOS Devices | MrChromebox.tech](https://docs.mrchromebox.tech/docs/firmware/wp/)
<br>
[Firmware Management Parameters](https://www.chromium.org/chromium-os/fwmp/)
<br>
[GBB flag-inator](https://binbashbanana.github.io/gbbflaginator/)

