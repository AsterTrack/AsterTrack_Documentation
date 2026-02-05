# Software

The AsterTrack Software consists of several more or less distinct projects, including four main ones required for the base functionality of the current hardware:

1. The **AsterTrack Server Software** (and its interface) running on the host system can be compiled and run on both Windows and Linux, though at the current stage, Windows support is lagging behind.

2. The **AsterTrack Controller Firmware** requires the standard RiscV toolchain to compile, and can thus be compiled on all platforms, though instructions may be Linux-specific for now.

3. The **AsterTrack Camera-Pi Firmware** is the main program running on the Raspberry Pi, and can currently only be compiled on the Pis themselves - a cross-compile toolchain is planned. The OS image - based on piCore 13.x - can currently only be created on Linux using a setup script system, though it should be possible to package new builds for firmware updates on other platforms, too.

4. The **AsterTrack Camera-MCU Firmware** runs on the STM32 MCU in the official hardware, and is compiled with the standard ARM toolchain and can thus be compiled on most major platforms, though again instructions may be Linux-specific for now. In the DIY Hardware the MCU may potentially be omitted, though this would change multiple aspects of the design and affects compatibility and should thus be avoided.

**Build Instructions** for all 4 main projects can be found in their respective READMEs in the main [AsterTrack repository](https://github.com/AsterTrack/AsterTrack).

Additionally, there are (or will be) further relevant projects:

- For IMU trackers complementing the optical tracking, custom SlimeVR-based nRF firmware is used, adding timesync compared to upstream. This consists of both a receiver and a tracker build based on the exact hardware used. Both require a nRF-specific, zephyr-based toolchain, as well as specific flashing tools - see the [SlimeVR docs](https://docs.slimevr.dev/smol-slimes/firmware/index.html) for details. The source code of the custom firmware will be published once its finalised and rebased properly.

- Example client software to interact with AsterTracks tracking output, applicable to multiple languages (C++ and python), can be found in [this repo](https://github.com/AsterTrack/Clients). There is also a `viewer` application not unlike the main AsterTrack Server Interface which aims to implement all supported AsterTrack protocols in C++ for immediate viewing.
