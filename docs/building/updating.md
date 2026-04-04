# Firmware Update

The firmware of the cameras and controllers can be updated from within the AsterTrack Server Software without any additional hardware.

### Update Order
Certain updates with breaking changes may require a specific order of updates.
In the future, this should be automatically communicated and handled.
For now, the preferred order is to update the MCU first, then the Pi, then the controller. <br>
If you want to update other cameras at a later point in time, you may have to temporarily downgrade the controller again.

## Camera Firmware Update
In the "Device" panel, click on the icon next to the firmware version of the camera to mark it for firmware update.
You may mark multiple cameras at once.
In the "Camera Firmware Update" section, you can now select the firmware file to upload. <br>
To update the camera MCU firmware, select the ".bin" binary and hit "Flash Cameras".
Confirm before flashing that you do not mix up the firmware for the camera MCU and the controller MCU - doing so may brick the camera. <br>
After any camera MCU firmware upgrade, update the camera Raspberr Pi firmware by selecting the ".tgz" package and hit "Flash Cameras". <br>
In the future, there may be a descriptive, complete update package format that combines these, and also a way to upload a complete OS image file (.img) to the camera for OS upgrades.

## Controller Firmware Update
Similarly in the "Device" panel, the "Controller Firmware Update" section guides you through updating the controller firmware. <br>
Currently you need to manually reboot the Controller into the bootloader, which can be done by pressing the middle Flash button for at least half a second. <br>
After that, the bootloader should appear as a USB device named "WinChipHead". On Windows, you may need to follow additional instructions listed in the UI to install the "WinUSB" driver for the bootloader using Zadig. <br>
If you are having troubles getting the "WinChipHead" bootloader to appear, the controller firmware may be bricked.
To forcefully boot into the bootloader, disconnect all cables from the controller, press and hold the middle Flash button, plug the controller in using the data port, and only then release the flash button. <br>
Once you can confirm the controller is in the bootloader, you may select the ".bin" firmware binary to upload in that "Controller Firmware Update" section, before hitting "Flash Controller".
Again, confirm you are not uploading th
