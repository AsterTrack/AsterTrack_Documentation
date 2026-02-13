# Tracker Configuration

## Tracker Types

**Targets** are 6-DOF trackers, optionally with an IMU, and refers to a static set of markers, with no moving parts within. <br>
Notably, Targets may use flat markers which can not be triangulated easily - refer to [Tracking](../tracking/tracking_alg.md) for details.

##### Future Trackers
In the future, other types of trackers may be implemented - anything that can be described and predicted by a model that helps to identify and keep track of a set of spacial parameters by observing markers. For example:

**Skeletal trackers** specifically are desired for motion capture purposes, referring to a skeletal model described by limbs constraints that allow single markers attached to the body to be labelled and tracked over time to derive a human pose.
Several traditional and neural-network based labelling solutions exist in research, but none are implemented yet.

**Sphere Trackers** are trackers using a single marker, intended to be identified either by a unique size, or via association with an IMU. With an IMU they are 6-DOF, otherwise 3-DOF.
Their use is niche and they are currently not fully implemented.

**Non-Rigid Targets** refers to any Target that is not completely static, but follows specific rules, e.g. two targets joined with a hinge. Their use is specific and niche and there are currently no plans to implement this.

##### Marker Point Cloud
In addition to these trackers, which are intended to have a unique identification that can be retained over time, AsterTrack is constantly triangulating individual markers and attempts to track them over time on a best-effort basis, assigning temporary identifications to them.
These can then be re-labelled and used externally, either in real-time or in post-processing.

## Tracker Configuration
You can find the trackers listed in the "Trackers" panel, though currently, only target trackers are well-supported.
There, you can configure their individual behaviour and role during tracking.

### Trigger & Expose Conditions

##### Trigger Conditions
When a tracker is **triggered**, it is assumed to be in the tracking volume to be tracked. For Targets, that means their [configured detection methods](#detection-config-target) is actively used. <br>
Trigger Conditions include:

- always assume it is trackable, e.g. when detection method is cheap
- when triggered manually, e.g. if detection method is expensive and you want to pick which trackers are involved
- when the IMU uniquely paired to that tracker is connected, indicating presence of that tracker
- when a client connects through one of the configured [I/O protocols](integrations.md#io-protocols) and requests tracking data from that tracker

##### Expose Conditions
When a tracker is **exposed**, it is available on configured [I/O protocols](integrations.md#io-protocols) for clients to request tracking data from. Depending on the I/O protocol, this may be actively advertising them as available, or only passively exposing them. <br>
Expose Conditions include:

- always, to e.g. allow external programs to determine the trackers to track
- when a target is triggered, copying their trigger condition
- when a target is first detected and tracked, potentially after they were first triggered

### Target Calibration (Target)
See [Target Calibration](calib_target.md) page for instructions on how to calibrate a new target or refine the calibration of an existing one. <br>
**Inspect** allows you to see the geometry of the calibrated markers in the 3D view.
Additional visualisation options may be available in the 3D views visualisation panel. <br>
**Scale adjustment** may be relevant in case a target was calibrated with a wrong room calibration - during room calibration, you can see the corrective scale factor used in the tooltip of the "Distance 1-2" field, or in the log "Scaled calibration by XX during floor calibration". <br>
**FoV adjustment** is used to adjust the automatically determined field of view of each marker during target calibration, and is previewed in the 3D view.
Markers can be seen from different angles based on their design, e.g. flat retroreflective stickers, retroreflective spheres, or LEDs with diffusers.
Experimenting with adding or removing FoV may improve detection and tracking performance, though this may mature in the future. <br>
**Export Target** saves a Object file (.obj) representation of the calibrated target to be used as reference.

### IMU Calibration
See [IMU Calibration](calib_imu.md) page for instructions on how to calibrate a trackers IMU once you calibrated the trackers optical component (target or marker).
In this section, you can associate an IMU from an external source to that tracker.
Some sources may already be intrinsically linked, e.g. when an IMU is provided by the I/O protocol that connected to this tracker.
In case that is the wrong IMU, ensure the external I/O protocol connects to the correct tracker instead - this may rely on the trackers name, depending on the I/O protocl. <br>
The IMU displayed here may not necessarily be connected, but configured and saved prior.
See the "Devices" panel for a list of currently connected IMUs in Device Mode. <br>
Some IMU sources may have difficulties with time sync, and since properly aligned timestamps are critical to proper fusion, you may need to adjust the timestamp offset to compensate.
This may eventually be possible to continuously estimate, but that is a difficult thing to just "fix in post".
Most integrated and officially supported IMU sources will try to provide accurate timestamps.

### Detection Config (Target)
There are several ways the tracking pipeline may attempt to initially detect a target before it can be tracked, and which is best depends on the target geometry and markers used. <br>

**3D Matching with Triangulations** is best for targets using spherical markers visible by multiple cameras at once.
It works by checking the triangulated point cloud not already claimed by tracked trackers against the marker geometry of the target.
If a target uses flat markers, and your camera setup is not set up to be able to triangulate even those reliably, then using this method is unlikely to be effective and may increase processing overhead. <br>

**2D Brute-Force Searching** is best for large targets with flat markers.
It works by testing three pronounced image points against three-tuples of the targets markers in a brute-force manner (using multiple threads).
Since this is done per-camera, it also requires good matching of other points in that camera, and is thus best suited for targets with many markers visible per camera, not small targets. <br>

**2D Brute-Force Probing** is best for small targets with flat markers.
It works by first estimating the 3D position by tracking 2D clusters of points in 3D (which can be visualised in the 3D View).
It then tests a certain number of rotations in a brute-force manner (using multiple threads), and uses the complex Blob Matching Stage used in tracking to match the expected projected markers to the image points.
This works even if no single camera sees enough points for tracking (or the 2D searching) to work.
However, the performance characteristics of it make it unsuitable for larger targets, but are great for smaller targets.
If desired, it can be used for larger targets by adjusting the Probe Count parameter, which adjusts the number of rotations probed. <br>
In the future, this method may also be used to re-detect recently lost targets, as an estimate (especially if supported by an IMU) may still be good enough, and the rotations probed may be greatly reduced using the rotational predictions.
