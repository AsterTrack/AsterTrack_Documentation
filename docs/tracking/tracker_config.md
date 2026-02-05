# Tracker Configuration

## Tracker Types

Currently supported are only **calibrated targets**, optionally with an IMU.
A target refers to a static set of markers, so no moving parts within. <br>
In the future, it may be possible to explicitly support trackers comprised of targets in a non-static relation, e.g. a hinge, but that is not planned. <br>
More relevantly, individual, typically larger, **spherical markers** may be tracked as well, though support for that is lacking right now.
In the future, it will be a supported configuration, especially in combination with an IMU. <br>
In the future, other types of trackers may be implemented - anything that can be described and predicted by a model that helps identify and keep track of a set of spacial parameters by observing markers. <br>
Specifically, for motion capture purposes, a **skeletal tracker** using single markers as reference is very much desired.
Limbs are described by a skeletal model, and markers attached to said limbs, then labelled and tracked over time.
Several traditional and neural-network based labelling solutions exist in research.

## Tracker Configuration
You can find the trackers listed in the Trackers View, though currently, only target trackers are well-supported.
There, you can configure their individual behaviour and role during tracking.

#### Trigger & Expose Conditions

When a tracker is **triggered**, it is assumed to be in the tracking volume and selected detection methods are used.
Trigger Conditions include:
- when the IMU uniquely paired to that tracker is connected (which may mean several things depending on the IMU Driver)
- when a client connects to the AsterTrack server using one of the I/O protocols and requests tracking information about that tracker (requires it to be exposed)

A tracker may be **exposed** using configured I/O protocols, e.g. advertised as available depending on the I/O protocol.
Expose Conditions include:
- when a target is triggered
- when a target is first tracked

#### Target Calibration

See [Target Calibration](calib_target.md) page for instructions on how to calibrate a new target or refine the calibration of an existing one. <br>
**Scale adjustment** may be relevant in case a target was calibrated with a wrong room calibration - the scale adjustment can be seen right before committing to calibrate the room in the scale tooltip. <br>
**Export** to .obj may be used as reference.

#### IMU Calibration
See [IMU Calibration](calib_imu.md) page for instructions on how to calibrate a trackers IMU once you calibrated its optical target. <br>
Here, you can only assign it in case there isn't already an IMU intrinsically linked, e.g. when an IMU is provided by the I/O protocol that connected to this tracker. <br>
See the Devices View for a list of currently connected IMUs in Device Mode.

#### Detection Config
There are several ways the tracking pipeline may attempt to initially detect a target before it can be tracked, and which is best depends on the target. <br>
**3D Matching with Triangulations** is best for targets using spherical markers visible by multiple cameras at once.
It works by checking the triangulated point cloud not already claimed by tracked trackers against the marker geometry of the target.
If a target uses flat markers, and your camera setup is not set up to be able to triangulate even those reliably, then using this method is unlikely to be effective and may increase processing overhead. <br>
**2D Brute-Force Searching** is best for large targets with flat markers.
It works by testing three pronounced image points against three-tuples of the targets markers in a brute-force manner (using multiple threads).
Since this is done per-camera, it also requires good matching of other points in that camera, and is thus only applicable for targets with many markers visible per camera, not small targets. <br>
**2D Brute-Force Probing** is best for small targets with flat markers.
It works by first estimating the 3D position by tracking 2D clusters of points in 3D (which can be visualised in the 3D View).
It then tests a certain number of rotations in a brute-force manner (using multiple threads), and uses the complex Blob Matching Stage used in tracking to match the expected projected markers to the image points.
This works even if no single camera sees enough points for tracking (or the 2D searching) to work.
However, the performance characteristics of it make it unsuitable for larger targets, but are great for smaller targets.
If desired, it can be used for larger targets by adjusting the Probe Count parameter, which adjusts the number of rotations probed. <br>
In the future, this method may also be used to re-detect recently lost targets, as an estimate (especially if supported by an IMU) may still be good enough, and the rotations probed may be greatly reduced using the rotational predictions.
