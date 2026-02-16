# AsterTrack Documentation

<img alt="Banner image with AsterTrack hardware" src="assets/Banner_AT_Rendered.png?v=2">

## What is AsterTrack?

AsterTrack is a custom multi-camera system designed to track markers and targets in 3D space for a variety of purposes like virtual reality and motion capture.
This so-called marker-based optical tracking is commonly used in professional studios, but typically costs several thousands of euros for even the most basic setups. <br>
AsterTrack implements the same concept with much less expensive hardware, and tries to pioneer a user-friendly multi-camera tracking experience.
It aims to be as accurate as the best consumer VR tracking systems, with similarly low latency, while being very affordable for what it does. <br>
Trackers can be anything you can attach retroreflective markers to, even 3D prints or existing objects without a battery.
This allows them to be much lighter, cheaper, and more flexible than any other type of tracking. <br>
Notably, AsterTrack does not just rely on triangulation, fully supporting the use of flat marker targets, setting it apart even from most professional optical tracking systems.

## Current State
The base multi-camera experience of AsterTrack is solid and ready for use, though some usability improvements can still be made.
The rest depends heavily on the use case: <br>
As a tracking system originally designed for consumer-first VR use, the support of flat marker targets is well developed, but to truly be useable for VR, the IMU integration needs to be completed first, and common tracker types developed and standardised. <br>
For use cases just relying on a triangulated point cloud of markers, this has been working for a long time - though integration support like C3D export is still being worked. <br>
Hardware is in the process of receiving its final major iteration, adding protections, more mounting points, and support for future wireless features.
Once it is ready, it will be available as a [dev kit](building/hardware.md#hardware), and [DIY versions](building/hardware.md#going-forward-with-diy) are planned as well.
