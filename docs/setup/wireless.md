# Wireless Features

The cameras can connect to a local wireless network to use as an alternative communication channel for high-throughput data like a high-quality video stream. That, in addition to allowing for remote development via SSH, has been its primary purpose. <br>
However, there are plans to extend this to full wireless operations via a separate wireless frame sync, [as described below](#wireless-camera-operation). <br>
NOTE: It is possible to build minimal images that don't have the packages for wireless use, and it is also possible to disable Wifi / SSH permanently (as long as you don't upload firmware that ignores these checks).

## Wifi Credentials
The cameras, once explicitly told to do so, store the wifi credentials on the SD card.
To set or update these wifi credentials, simply open the "Wireless" panel and edit the *wpa_supplicant* configuration.
This will only be stored in the software for this session and not retained except on the cameras. <br>
For normal networks, writing the network name in the "SSID" quotes and the password into the "PASSWORD" quotes suffices, though you may want to use `wpa_passphrase` on Linux to generate a PSK to use instead of the plaintext password. <br>
For public, corporate or institutional networks, refer to [the arch wiki](https://wiki.archlinux.org/title/Wpa_supplicant) for more information about the *wpa_supplicant.conf* format.
You can also add multiple networks in case you move the cameras around. <br>
These credentials are sent to the cameras only when you change their wireless config, and stored/updated only when you explicitly apply their config in the dropdown menu.

## Wireless Config
In the wireless camera table, you can configure and inspect the wireless status of all connected cameras. <br>
<br>
The first few columns are wireless feature toggles (use "Apply Config" to make them persist): <br>
**Toggle Wifi:** Connection to a configured wireless network. <br>
**Toggle SSH:** OpenSSH access for [remote development work](#remote-development-via-ssh) on the camera. <br>
**Wireless Server Connection:** Connect to the [wireless server](#server-config) to use for high-bandwidth data. <br>
**Realtime Data over Server:** Use the wireless server for the [realtime data stream](#wireless-camera-operation) (experimental). <br>
Each wireless feature toggle has a tooltip that shows its current status. <br>
<br>
**Status:** If the camera is connected to a wireless network, it will show its own local network IP in the status field. <br>
<br>
**Actions:** The last column allows you to perform actions that change the persistent wireless config. <br>
"Apply Config" will store the current config on the SD card, including the wifi credentials and server config. Without this, the wireless config will not persist when the camera is restarted. <br>
"Clear Credentials" removes any wifi credentials stored on the camera - just toggling wifi off does not clear them. <br>
Finally, you may permanently disable Wifi and SSH. These can only be undone by either flashing firmware onto the camera that ignores/undoes these checks, or by taking the camera apart and modifying the internal SD card, so be careful. <br>

## Server Config
Here you may edit the default host and port the cameras try to connect to, though these should be set up automatically. <br>
There are currently no tools to permanently change either the server host or port - the host will be set to the hostname of the PC automatically. But the server information is currently only sent to the cameras when changing their wireless config, so changes should persist if you applied the config.

#### Troubleshooting
If the cameras can not connect to the wireless server despite connecting to the wifi network, there are two likely possibilities: <br>
**Network Permissions:** The application may not have sufficient privileges to communicate with devices on the network. <br>
On Windows, when the wireless server first starts up, a popup will have asked you to give the application permission to communicate on public and/or private networks.
You can check this in the firewall rules, but be aware that the interface may be very buggy.
Your best bet is to remove AsterTrack from the list entirely, restart the AsterTrack Application, and start the wireless server again ("Connect") to bring up the popup again. <br>
On Linux, if you have a firewall set up, you may need to allow incoming packets on port 48532 (both TCP and UDP). <br>
**Hostname Resolution:** The camera may not be able to resolve the IP from the hostname.
This may be a failure of DHCP on your system making the router aware of your hostname in the first place, or a DNS failure due to misconfiguration of the router.
To test this, find the local IP address assigned by the router - preferrably the IP of your ethernet interface if you use it, otherwise that of your wifi interface.
Enter that local IP into the "Server Host" field and hit the button to update.
You may also want to visit your routers interface to check if your PC does have its hostname assigned.

## Wireless Camera Operation
While the camera currently only supports wired operation, newer iterations of the hardware are designed to allow for wireless operation in the future.
There are three data streams that need to be made wireless: <br>
**1. High-Throughput Data:** This includes image streaming, with high bandwidth but loose latency requirements.
This is already preferrably sent over wifi when the wireless server is connected. <br>
**2. Low-Latency Data:** This is the realtime-relevant data like detected blobs, and requires some throughput as well.
This can be experimentally toggled in the [Wireless Config](#wireless-config) to also be sent over the wireless server connection. <br>
**3. Time-Critical Frame Sync:** This needs to have very predictable timing, low latency, low jitter, but also with very low throughput. <br>
This currently still requires a wired connection, but there are concrete plans for wireless frame sync using Nordics Proprietary 2.4GHz ESB protocol via a nRF24L01 in the camera hardware.

While we do plan to allow for fully wireless operation in the future, it does not have any latency guarantees, and does not scale to larger setups, so should not be used for demanding applications.
It is primarily intended for home use where setting up certain cable runs is prohibitive, and turning one or multiple cameras wireless is preferrable.


## Remote Development via SSH
When SSH is enabled in the [Wireless Config](#wireless-config), you may connect to the camera.
The default username/password is currently "tc"/"piCore".
It is recommended to only enable SSH when you need it, and only use it on a secure network with trusted devices.
Here are some tools/instructions for development on the camera:

- `stop.sh` to stop the currently running instance of the program (only one may run at once)
- `sudo ./run.sh &` to resume normal camera operations in the background
- `sources` contains sources of the current build (updated along with firmware updates)
- `build_release.sh` / `build_debug.sh` to compile the program
- `sudo gdb --args TrackingCamera/TrackingCamera_armv7l -u` to debug
- use sftp service to transfer new sources and extract compiled builds quickly
- `filetool.sh -b` to saving your work (changes, build, scrollback) to SD card<br>
- configuration is stored in `/mnt/mmcblk0p2/config`
- additional storage is available at `/mnt/mmcblk0p4`
- to recompile camera kernel drivers (e.g. to modify them):
    - `./drivers_auto_build.sh`: download and prepare the kernel, download, patch and build the drivers
    - `./drivers_load_modules.sh`: registers and loads the newly built driver module (at startup)
        - You can not replace the currently loaded driver. Ensure this script refers to the driver you want loaded, save your work with `filetool.sh -b`, and restart.
