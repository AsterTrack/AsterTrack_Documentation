# Firmware Update

The firmware of the cameras and controllers can be updated from within the AsterTrack Application without any additional hardware.

### Update Order
The order in which the update is applied only matters for updates where this is explicitly mentioned, e.g. if they change the way the UART protocol works. Follow the instructions then.
In those cases, it should be safe to update the MCU first, then the Pi, then the controller. If you happened to have updated the Pi first, update the controller next, then the MCU. You can always downgrade the controller again to achieve this ordering for all cameras.

## Camera Firmware Update
In the "Device" panel, click on the icon next to the firmware version of the camera to mark it for firmware update. You may mark multiple cameras at once. <br>
In the "Camera Firmware Update" section, you can now select the firmware file to upload. Currently, you need to separately select a .tgz package to update the Raspberry Pi firmware on the camera, or a .bin binary to update the MCU firmware on the camera. <br>
In the future, there may be a descriptive, complete update package format that combines these, and also a way to upload a complete OS image file (.img) to the camera for OS upgrades.

## Controller Firmware Update
Currently you need to manually reboot the Controller into the bootloader, which can be done by pressing the middle Flash button for one second (technically half a second is enough). <br>
After that, it should appear as "WinChipHead", which is the default bootloader. You can then either use the isp55e0 built with the controller project through the provided VS Code task, or use the firmware flashing tool integrated into the AsterTrack Application. <br>
Switch to the "Devices" panel and select the firmware binary to upload in the "Controller Firmware Update" section. Then, with a controller in bootloader mode, hit "Flash Controller in Bootloader".