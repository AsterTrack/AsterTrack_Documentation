# Software

The AsterTrack Software consists of several more or less distinct projects, including four main ones required for the base functionality of the current hardware:

1. The <a href="https://github.com/AsterTrack/AsterTrack/blob/main/server/README.md">**AsterTrack Server Software**</a> supports both Windows (msvc) and Linux (clang, gcc).

2. The <a href="https://github.com/AsterTrack/AsterTrack/blob/main/controller/README.md">**AsterTrack Controller Firmware**</a> for the CH32V307 requires the standard GNU RISC-V toolchain to compile. You can compile and flash from Window, but the development tools are much better developed on Linux.

3. The <a href="https://github.com/AsterTrack/AsterTrack/blob/main/camera_mcu/README.md">**AsterTrack Camera-MCU Firmware**</a> for the STM32G030 in the Camera requires the standard GNU ARM toolchain to compile. You can compile, flash and develop on both Windows and Linux, though the development tools are still better on Linux.

4. The <a href="https://github.com/AsterTrack/AsterTrack/blob/main/camera/README.md">**AsterTrack Camera-SBC Firmware**</a> for the Raspberry Pi runs on a piCore based OS, and can only be compiled on the Pis themselves - though a cross-compile toolchain is planned. The custom build system needed to configure the image itself currently requires Linux (or WSL on Windows), though you can manually configure an existing development image on Windows.

**Build Instructions** for these can be found by following the links to their respective READMEs in the main [AsterTrack repository](https://github.com/AsterTrack/AsterTrack).

Additionally, there are (or will be) further relevant projects:

- For IMU trackers complementing the optical tracking, custom SlimeVR-based nRF firmware is used, adding timesync compared to upstream. This consists of both a receiver and a tracker build based on the exact hardware used. Both require a nRF-specific, zephyr-based toolchain, as well as specific flashing tools - see the [SlimeVR docs](https://docs.slimevr.dev/smol-slimes/firmware/index.html) for details. The source code of the custom firmware will be published once its finalised and rebased properly.

- Example client software to interact with AsterTracks tracking output, applicable to multiple languages (C++ and python), can be found in [this repo](https://github.com/AsterTrack/Clients). There is also a `viewer` application not unlike the main AsterTrack Server Interface which aims to implement all supported AsterTrack I/O protocols in C++ for immediate viewing.
