# USB-C_akg_sm-t220-fix
USB c akg fix for Roms in Galaxy Tab a7 Lite WiFi 
usbfix: USB audio fix for the Galaxy Tab A7 Lite Wi-Fi (SM-T220)

Made for: crDroid 11.9, Android 15, build 2025-09-23 (ROM by Duhen)
Module author: RJABA26
Status: works on my unit (one device tested)

WHAT IT DOES
- Fixes USB-C headphones/earphones that are detected but play no sound.
- Makes Android send USB audio through its generic USB audio HAL.
- The MediaTek HAL (the default path) fails to open the USB device.

THE PROBLEM
- The tablet detects the USB-C audio device and lists it as the output.
- The kernel sees the sound card, but playback never starts.
- Logcat shows the MediaTek audio HAL failing:
    AudioALSAPlaybackHandlerUsb: write(), mPcm is NULL!
    AudioUSBCenter: sample_rate 48000 not support (falls back to 44100)
- The 3.5 mm jack works normally.

HOW IT WORKS
- /vendor/etc/audio_policy_configuration.xml declares USB output twice:
  in the MediaTek "primary" module and in the generic "usb" module.
- The module removes "USB Device Out" and "USB Headset Out" from the
  primary module only (2 device ports and 2 routes, 8 lines).
- Android is left with one USB output path: the generic USB HAL.
- That HAL has no fixed sample rate, so the system can resample to
  whatever the device supports.
- The XML is generated on your tablet at install time from your own
  vendor file. The module does not ship a modified copy.
- It is applied systemlessly. /vendor is not modified.

WHAT IT DOES NOT TOUCH
- The audio HAL binaries.
- USB input entries (microphones).
- Speakers, 3.5 mm jack and Bluetooth routes are left as they were.
  I have not tested each of them for side effects.

SAFETY CHECKS
The installer aborts and installs nothing if:
- /vendor/etc/audio_policy_configuration.xml does not exist,
- it would not remove exactly 8 lines,
- the number of device ports or routes is not what it expects,
- any "USB Device Out" or "USB Headset Out" reference is left.
It also saves the original file to /sdcard/audio_policy_backup.xml
and gives the generated file the original SELinux context.

REQUIREMENTS
- Galaxy Tab A7 Lite Wi-Fi (SM-T220, gta7litewifi).
  Not for the SM-T225 or SM-T227U.
- Tested on crDroid 11.9, build 2025-09-23. Other builds may work
  because it reads your own file, but that is untested.
- A root manager with module support (Magisk-style).

INSTALL
1. Download usbfix-audio.zip (link below).
2. Root manager > Modules > Install from storage > pick the zip.
3. Read the output. The messages are in Spanish. The last line tells
   you to reboot.
4. Reboot, connect the USB-C headphones and play music.

VERIFY
  su
  dumpsys media.audio_policy | grep -n 'Mix; Sink: "USB'
  grep -i status /proc/asound/card1/stream0
- USB routes should now appear only under the "usb" module.
- Playback status should show Running while music plays.
- The card number (card1) can be different on your device.

UNINSTALL
- Disable or remove the module in your root manager and reboot.
- If you cannot open the manager, run from adb:
  touch /data/adb/modules/usbfix/disable
  and reboot.
- The original file stays in /sdcard/audio_policy_backup.xml.

AFTER A ROM UPDATE
- Install the module again. It rebuilds the file from the new vendor.

LIMITATIONS AND RISKS
- It is a workaround, not a fix of the HAL itself.
- Tested only on one SM-T220 with AKG USB-C earphones.
- USB DACs and adapters should use the same path, but I have not
  tested them.
- Editing audio policy can leave a device without sound if something
  goes wrong. Keep adb or recovery access before you reboot.
- Use at your own risk.

DOWNLOAD
usbfix-audio.zip: <add your link here>

CREDITS
- RJABA26: module and documentation
- Duhen: the ROM for this tablet
- crDroid team and LineageOS: the base
