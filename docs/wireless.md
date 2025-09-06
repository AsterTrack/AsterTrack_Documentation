# Wireless Features

## Enable Wifi
Wifi (and SSH) can be great tools to debug a camera if it misbehaves, or even to start developing on it. <br>
In the future, a direct server-to-camera connection my be used to supplement or even replace the wired connection through the controller. <br>
Any image that has the wifi packages installed and wifi or SSH not explicitly forbidden via config, can be made to connect to a wifi network using the "Wireless Setup" view panel.

### Wireless Setup
The wifi credentials currently need to be provided every time, as there is no support within the server for either storing them on the host via a secrets API, nor on the camera. <br>
To set the credentials, use a tool like wpa_passphrase (on linux) or write the wpa_supplicant.conf manually (on windows) and enter it in the wpa_supplicant.conf field hidden behind a collapsing header. Details on wpa_supplicant.conf can be found [here](https://wiki.archlinux.org/title/Wpa_supplicant).

## SSH for Development
If the camera is connected to a wifi network, it will show its IP and the SSID of the network (shown in the tooltip of the IP). <br>
When trying to connect, you will have to enter "piCore". <br>
Using dev images, you have all capacities to start developing:
- stop.sh to stop the currently running instances of the program
- sources of the current build (always updated with firmware updates) 
- `build_release.sh` / `build_debug.sh` to compile the program
- active sftp service to transfer new sources and extract compiled builds
- debugging with `sudo gdb --args TrackingCamera/TrackingCamera_armv7l -u`