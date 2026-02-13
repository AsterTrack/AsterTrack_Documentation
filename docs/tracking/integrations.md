# Integrations 

## I/O Protocols
I/O protocols are the way to stream tracking data out of the AsterTrack Server Application and towards various clients in real-time.
Clients may include VR drivers, intermediate MoCap software, motion analysis programs, or any custom program developed by you. <br>
There are some [examples of custom clients](https://github.com/AsterTrack/Clients) interfacing with these protocols in a selection of languages.

#### VRPN
This is currently intended to be the primary I/O protocol for realtime streaming.
[VRPN](https://github.com/vrpn/vrpn/wiki) was initially designed for Virtual Reality on the local network, and it uses mostly UDP to send tracker updates.
While mostly designed for 6-DOF trackers, it is also being used to send IMU samples in the reverse direction (from client to AsterTrack), and will soon also support sending marker point clouds. <br>
VRPN has language bindings for C++, C#, Java, and python (among some others).


## Direct Integrations
This is a list of realtime streaming integrations with third-party software (or examples thereof).
These are developed with explicit support for AsterTrack, but they may use a common [I/O protocol](#io-protocols) and be used by other servers of such protocols as well.

#### Example Viewer
A [demo client written in C++](https://github.com/AsterTrack/Clients/tree/main/viewer) intended to support all I/O Protocols. It is based off of the main server interface and should be easy to reference and/or modify.

#### Blender Plugin
A [blender integration script](https://github.com/AsterTrack/Clients/blob/main/examples/python/blender_vrpn.py) written in python that allows mirroring a trackers movement to an object in blender using the [VRPN I/O protocol](#vrpn).
This is intended to get full support as an addon in the future - for now, follow the [setup instructions](https://github.com/AsterTrack/Clients/blob/main/examples).

#### SteamVR Driver (Planned)
A SteamVR driver for exposing configured AsterTrack trackers to other VR drivers to use with [Tracking Overrides](https://github.com/ValveSoftware/openvr/wiki/TrackingOverrides), and in some form also interact with DIY VR Hardware providing IMU data. It is intended to be using the [VRPN I/O protocol](#vrpn).

#### Monado Driver (WIP)
A monado driver intended to override tracking of other VR hardware supported by monado, including DIY headsets.
While technically just implementing the [VRPN I/O protocol](#vrpn), it supports AsterTrack specific features like sending IMU samples to the AsterTrack server. <br>
This is mostly done, with both tracking and IMU samples being exchanged, and the general overriding structure being complete.
But without proper IMU integration, the tracking spaces are misaligned, and thus it is not particularly useful yet.
The plan is that you can retrofit any monado-supported VR hardware with tracking markers, calibrate them, set up the VRPN connection, and the IMUs are automatically calibrated so the tracking spaces align themselves.


## File Formats
For offline, non-realtime post-processing work, a capture and/or tracking results can be exported to file formats used by relevant third-party software.
If you require a format to be supported for your use case, feel free to reach out.

#### Custom Capture
The internal format used for saving captures is still evolving and should be considered unstable. Until it matures, it is based on JSON, with plans to switch to a binary format in the future.
It is designed to hold raw 2D points and IMU samples only, with tracking results being saved separately.

#### C3D Export (Planned)
As a widely-adopted file format for 3D point clouds, support for [C3D export](https://www.c3d.org/) is naturally planned.
It is quite limiting in the data it can hold, mostly triangulated points, lacking standard ways to describe 6-DOF trackers.
This means any target using flat markers will be "invisible" to an application reading the C3D files, as most markers used for tracking such a target are not triangulated. <br>
Although implementation has not started yet, please contact us with the application you intend to import C3D files into so we can verify support.


## Planned Support
This is a list of applications and use cases we are aware of and intent to support in the future to some degree, with some getting direct integrations (see [Clients](#clients) above), some supported via common [I/O protocols](#io-protocols), and some supported via [file export](#file-formats) only.
If an application you want to use is not on here or in the list above, feel free to contact us with your use case.

**Unreal/Unity/Godot:** Realtime mirroring of tracked objects, at first planned via SteamVR only, relying on existing plugins.

**Unreal:** Planned to directly integrate via a LiveLink Plugin, providing Live Link Controllers (Transform, Camera).

**VMC Protocol:** Planned to integrate with any VMC Performer () by acting as a VMC Assistant and streaming 6-DOF tracking data.

**Motion Builder:** Planned support via C3D file import.

**Biomechanics Analysis** Planned support for Visual3D / Sift, AnyBody, BoB Biomechanics, OpenSim and perhaps others via C3D file import.