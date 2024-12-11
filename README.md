
<img src="https://github.com/truekas/PencilSharpener/blob/main/src/Logo.png?raw=true" alt="Pencil Sharpener"/>

> [!WARNING]  
> DURING THE EXPLOIT, MAKE SURE THE CHROMEBOOK STAYS PLUGGED IN TO A CHARGER AT ALL TIMES (EXCEPT WHILE BRIDGING THE WP PINS). THE SYSTEM WILL BRICK IF THIS IS NOT FOLLOWED.

> [!IMPORTANT]
> This writeup is for educational purposes only. Do not use this exploit on your organization's systems without permission. Remember that your orginization's Chromebook is not your personal device. Nobody is responsible for any touble that happens because of this exploit.

## Hey There!
If you're looking at this writeup, this exploit has been patched. Recently, Google rolled all shim keys on all Ti50 boards during the update to Kv5, making it impossible to boot into Sh1mmer or use this exploit. We decided to release this writeup after the patch, to ensure that no damage is caused to the devices of schools and companies. 

If you are an administrator, we recommend that you set the `DeviceMinimumVersion` in Google Admin to ensure that all new Chromebooks have been patched. We also recommend that you check the policy sync dates for all users on Cr50 Chromebooks to monitor exploitation.

## Introduction 
This writeup demonstrates how Google's tsunami enrollment patch, released on v114, can be bypassed on newer mainboards with the Ti50 chip on Kv4. The exploit uses a modified version of the original pencil exploit to bypass FWMP and VPD and prevent the system from bricking. If you are curious about Pencil Sharpener, you can check out our [explanation](https://github.com/truekas/PencilSharpener/blob/main/Explanation.md) of why it works.

## The Exploit

What you need:
- A pencil or conductive material (chip clip recommended) 
- A screwdriver and an ESD bracelet to prevent damage to the device
- A sh1mmer image for your system flashed to an SD card or USB stick
- Working device charger
- A ChromeOS recovery USB for v125
- A few braincells 

First, fully power off and unplug your device, flip it over, and open the back to gain access to the mainboard.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/6.png?raw=true" alt="6.png"/>

Then, disconnect the battery from the mainboard and locate your device's Flash Chip, usually near the mainboard's battery socket and covered in black tape (different for every device). Bridge pins 3 and 8 with your conductive material or chip clip.

<img src="https://github.com/truekas/PencilSharpener/blob/main/src/2.png?raw=true" alt="2.png"/>

Afterward, re-insert your charger AND KEEP IT PLUGGED IN while pushing `esc + refresh + power` to enter the device recovery menu. Then press `ctrl + d` and as soon as the screen goes black, press the keys to re-open the recovery menu. 

Insert your sh1mmer USB and then choose to boot from it. You may get a no valid image error. If this happens, you need to re-flash the correct keys to the device. Please follow the [Rolled Keys](#fixing-rolled-keys) section.

Re-open the recovery menu and boot into sh1mmer. You should choose `utilities > unenroll` and after it gives an error, open the bash console WHILE MAKING SURE THE PINS ARE STILL BRIDGED and run:
```
flashrom --wp-disable
/usr/share/vboot/bin/set_gbb_flags.sh 0x80b3
flashrom --wp-enable
```
Hit `esc + refresh + power` to go back into the recovery menu and now boot onto your v125 recovery USB and follow its instructions. See the [keyroll steps](#fixing-rolled-keys) on the document if you keyroll again.

After the recovery process is complete, choose to boot into ChromeOS. Then switch to the Vt2 console on the sign-in screen by pressing `ctrl + alt + f2` and run:
```
tpm_manager_client take_ownership
cryptohome --action=remove_firmware_management_parameters
crossystem dev_boot_usb=1
```

Now make sure that the battery is re-inserted on the mainboard and run `gsctool -a -o`. Follow the prompt to push the power button and the system should automatically reboot. When the device turns back on, re-open the recovery menu and re-enable devmode.

Next, go back to the Vt2 console, run `gsctool -a -I AllowUnverifiedRo:always`, and the device should be unenrolled.

## Fixing Rolled Keys
After downgrading or trying to use Sh1mmer, some systems will keyroll and prevent users from booting. This is because the recovery kernel data key will fail to validate the kernel during boot. 

This issue is fixable by flashing the correct keys to the system. Here's how to do it:

Go into VT2 with `CTRL+ALT+F2` or if you can't get to VT2, use a flash programmer (ch341a) and a chip clip connected to the flash chip. After you are [connected to](https://docs.chrultrabook.com/docs/unbricking/unbrick-ch341a.html#prepping-to-flash) your device, bridge pins 3 and 8 run these commands.

```bash
flashrom --wp-disable
futility gbb --recoverykey file.bin
futility gbb -s --recoverykey file.bin # add -p if using a programmer
flashrom --wp-enable
```

## Re-Enrolling
It is possible to re-enroll your device by accessing a Vt2 shell, typing `vpd -i RW_VPD`, and then powerwashing the device by switching from developer mode to verified mode.

## Citations (Not MLA)
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
<br>
[CrOS EC (Embedded Controller) | Software Sync](https://chromium.googlesource.com/chromiumos/platform/ec/+/HEAD/README.md#Preventing-the-RW-EC-firmware-from-being-overwritten-by-Software-Sync-at-boot)
