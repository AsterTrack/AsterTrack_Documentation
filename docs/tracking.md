# Tracking

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

## Tracking Algorithm
A brief explanation of tracking (specifically for targets) follows - not that detection is a separate, more compute intensive process.
Target tracking refers to when we already know a target is roughly in a certain position and orientation.

#### Prediction
We use a filter to estimate and predict the state of each tracker at the time of the current frame. This gives us an initial position and orientation to start tracking.

#### Matching
The pipeline projects the targets markers using that prediction into all cameras and matches them against the image points, where each matching stage is done for all cameras. <br>
First, a **fast blob matching stage** is tried which matches points based on distance.
With a bad prediction, this quickly becomes insufficient even at high framerates, so it is only used as an early-out stage. <br>
Next, several iterations of a **complex blob matching stage** may be performed.
This stage is mostly position-invariant up until a preconfigured pixel distance that serves as one of multiple performance regulators.
It first calculates the offsets to all nearby points, then scores matching based on the similarity of matchings of other nearby points.
This is not fully invariant to rotation, but in practice, most prediction issues stem from positional errors.
The matching of the points - determined by a combination of that similarity value and distance - then informs a positional offset to apply.
Used in iteration, this stage experimentally proved very effective in resolving quite complex point matching problems. <br>
In between stages, if sufficient points have been matched, a quick optimisation may also be used to correct for rotational errors.
Then, after all stages have concluded, and the final point matching has been determined (and potentially optimised), the number of points matched determines how to proceed.

#### Filter Update
If the number of matched points is insufficient for optimisation (or, in practice, a number even higher than that), we cannot technically determine a full new pose for that frame.
This is where a filter comes in.
We currently use an Unscented Kalman Filter, and we may update it using a SCAAT-like approach of directly feeding the measurements as observations of matched points and their predictions.
This enables the filter to update its estimate of the true pose even with insufficient data in each frame. <br>
However, if enough points are matched, we may optimise using those alone and determine a fully updated pose.
In that case, we updated the filter using the pose itself as the measurement. <br>
Updating the filter is an important and sensitive part of the tracking pipeline, as a good prediction for the next frame is crucial for tracking to work.

##### IMU Filter Update
Of course, optical observation by the cameras is not the only information we may have about a tracker.
If an IMU is assigned and calibrated, we may update the filter using its measurements.
Which are used exactly depends on whether the IMU is providing raw sensor values (accelerometer and gyroscope) or fused sensor values (accelerometer and orientation).

### Debugging Tracking
There are several ways to debug the tracking pipeline, and many of them are only really relevant during Replay, when frames can be halted and examined in detail.

#### Insights
The Insights View provides a quick overview of the frame-by-frame quality of the tracking - though you may need to select the tracker you want to inspect manually.
The graph shows both the number of tracked markers (matched points) as well as the reprojection error of those - a low reprojection error, doesn't mean much by itself if the marker count is low, especially if only one camera contributes. <br>
The bottom bar additionally shows the state of the tracker:

- Purple is a failed detection attempt
- Pink is a successful detection
- Blue is asynchronous tracking after a detection to catch up with realtime
- Green is real-time tracking
- Orange is tracking failure due to insufficient data or incorrect matching
- Red is complete tracking loss after a few frames of failed tracking

#### Frame Inspection
When in replay or simulation mode, and frame progression is halted in the Control View, and a tracker is selected in the Pipeline View, Section *Tracking*, then the Section *Tracking Debug* appears.
Entering debug mode redoes the tracking of that tracker for the current frame and stores internal data for display in the Pipeline and Camera Views. <br>
Additional display options are in Visualisation View, Section *Target Tracking*, allowing you to step through the individual matching stages.
Pressing *Edit* in the Section *Tracking Debug* also allows you to inspect internal values, edit point matches (after pressing the *Matches* button), edit tracking parameters, and evaluate what would have happened had the matching stage performed differently.
The UI is not polished, and knowledge of the internal algorithm is required, but it serves as a good tool when working on improving tracking.

#### Parameter Optimisation
There are a large amount of parameters affecting tracking quality, and tuning them is an arduous process.
For that reason there exists an extensive configuration UI as well as tooling that may re-run tracking on critical sections of a replay whenever a parameter is changed. <br>
To access this tooling, open the Tracking Params View at View/Parameters/Tracking, and the Section *Optimising Tracking Parameters* should appear in the Control View when in Replay mode.
This tool stores several frame ranges that may be manually or automatically added (based on points where tracking is lost).
Then, whenever a tracking-relevant parameter is changed, these ranges are automatically revisited.
To aid in comparision between changed parameters, a baseline may be stored to compare against. <br>
It should be noted that just looking at a few short frame ranges may be fast and convenient, but may hide other areas where tracking was fine before, but critical after a parameter change.
Thus, before committing to a new set of parameters, it may be prudent to re-run tracking on one or multiple captures and comparing against previously stored tracking results across the whole replay.
This is discussed next.

#### Comparing Tracking Results
In order to compare tracking results across a whole replay to a prior state (e.g. when changing parameters or tracker configuration), there exists a current tracking record, and a stored tracking record (originally created on initial capture of the replay).
These are visualised in the Insights View, in the Tracking Tab, using bright and muted colors respectively. <br>
Additional tooling for this can be found in the Pipeline View, Section *Tracking Results for this replay*.
You can update the stored tracking record here, either temporarily to compare against, or overwriting the prior stored tracking record on disk.
You can also compare the current and stored tracking record numerically by pressing *Update Tracking Results* and expanding the resulting section.