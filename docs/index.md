# AsterTrack Documentation

<img alt="Banner image with AsterTrack hardware" src="assets/Banner_AT_Rendered.png?v=2">

## What is AsterTrack?

AsterTrack is a custom multi-camera system designed to track markers and targets in 3D space for a variety of purposes like virtual reality and motion capture.
This so-called marker-based optical tracking is common in professional studios, but typically costs several thousands of euros for even the most basic setup. <br>
AsterTrack implements the same concept with much less expensive hardware, and tries to pioneer a user-friendly multi-camera tracking experience.
It aims to be as accurate as the best consumer VR tracking systems, with similarly low latency, while being very affordable for what it does. <br>
Targets can be anything you can attach retroreflective markers to, even 3D prints or existing objects without a battery.
This allows them to be much lighter, cheaper, and more versatile than any other type of tracker. <br>
Notably, AsterTrack does not just rely on triangulation, but fully supports the use of flat marker targets, setting it apart even from most professional optical tracking systems.
This makes tracked objects much less intrusive as it does not require attaching big spherical markers, which would be impractical in consumer VR applications.

## Current State
The base multi-camera experience of AsterTrack is solid and ready for use, though some usability improvements can still be made.
The rest depends heavily on the use case: <br>
As a tracking system originally designed for consumer VR use, the support of flat marker targets and 6-DOF tracking in general is well developed.
But to truly be usable for VR, the IMU integration needs to be completed, and common trackers designed for 3D printing.
IMU integration will also help track small trackers more reliably. <br>
For use cases relying on a triangulated point cloud of markers, this is still being developed - both the frame-to-frame labelling of markers as well as integrations like C3D export. <br>
The hardware is in the process of receiving its final major iteration, adding protections, more mounting points, and support for future wireless features.
Once it is ready, it will be available as a [dev kit](building/hardware.md#hardware), and [DIY versions](building/hardware.md#diy-version) are planned as well.
