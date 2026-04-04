# Tracking Algorithm

A brief explanation of tracking (specifically for targets) follows.
Do note that detection is a separate, more compute intensive process that is performed asynchronously.
Target tracking refers to when the target is already known to roughly be in a certain position and orientation.

## Tracking Step

### Prediction
A Kalman Filter is used to estimate and predict the state of each tracker at the time of the current frame, using state estimated over past frames.
This gives us an initial position and orientation estimate to continue tracking in this frame.

### Matching
The pipeline projects the targets markers into all cameras using that prediction and matches them against the image points, where all of the following matching stages are done for each camera. <br>
First, a **fast blob matching stage** is tried which matches points based on distance.
With a bad prediction, this quickly becomes insufficient even at high framerates, so it is only used to early-out in simple situations. <br>
Next, several iterations of a **complex blob matching stage** may be performed.
This stage is mostly position-invariant up until a preconfigured pixel distance that serves as one of multiple performance regulators.
It first calculates the offsets to all nearby points, then scores matching based on the similarity of matchings of other nearby points.
This is not fully invariant to rotation, but in practice, most prediction issues stem from positional errors.
The matching of the points - determined by a combination of that similarity value and distance - then informs a positional offset to apply.
Used in iteration, this stage proved very effective in resolving quite complex point matching problems. <br>
In between stages, if sufficient points have been matched, a quick optimisation may also be used to correct for rotational errors.
Then, after all matching stages have concluded across all cameras, and the final point matching has been determined (and potentially optimised), the number of points matched determines how to proceed.

### Filter Update
If the number of matched points is insufficient for optimisation (or, in practice, a number even higher than that), a full new pose cannot technically be determined for that frame.
This is where the filter comes into play again:
It is corrected (EKF update) using a SCAAT-like approach of directly feeding the measurements as observations of matched points and their predictions.
This enables the filter to update its estimate of the true pose even with insufficient data in each frame. <br>
However, if enough points are matched, the target may be optimised using those alone to determine a fully updated pose.
In that case, the filter is corrected (UKF update) using the pose itself as the measurement. <br>
Updating the filter is an important and sensitive part of the tracking pipeline, as a good prediction for the next frame is crucial for tracking to work.
This means, the level of smoothing that you would expect from the final tracking output can not yet be applied, as a quick reaction to changes is more important than lower perceived jitter.

#### IMU Filter Update
Of course, optical observation by the cameras is not the only information that may be known about a tracker.
If an IMU is associated to the tracker and calibrated, the kalman filter may be updated using its measurements as well.
Which samples are used exactly depends on whether the IMU is providing raw sensor values (accelerometer and gyroscope) or fused sensor values (accelerometer and orientation). <br>
Usually, this is done ahead of prediction, to get a better initial estimate, and after tracking solely for output purposes.

## Debugging Tracking
There are several ways to debug the tracking pipeline, with some of them only relevant during [replay](../operation/recording.md#replay), when frame processing can be halted and examined in detail.

### Insights
The "Insights" panel provides a quick overview of the frame-by-frame quality of the tracking in the "Tracking" tab - though you may need to select the tracker you want to inspect manually.
The graph shows both the number of matched markers as well as the reprojection error of those - a low reprojection error doesn't mean much by itself if the marker count is low, especially if only one camera contributes. <br>
The bottom bar additionally shows the state of the tracker:

- Purple is a failed detection attempt
- Pink is a successful detection
- Blue is asynchronous tracking after a detection to catch up with realtime
- Green is real-time tracking
- Orange is a tracking failure due to insufficient data or incorrect matching
- Red is a complete tracking loss after a few frames of failed tracking

### Frame Inspection
When in replay or simulation mode, and frame progression is halted in the "Control" panel, and a tracker is selected in the "Pipeline" panel, section "Tracking", then the section "Tracking Debug" appears.
Entering debug mode redoes the tracking of that tracker for the current frame and stores internal data for further display in the UI. <br>
Additional display options are in the "Visualisation" panel, section "Target Tracking", allowing you to step through the individual matching stages.
Pressing *Edit* in the section "Tracking Debug" also allows you to inspect internal values, edit point matches (after pressing the *Matches* button), edit tracking parameters, and evaluate what would have happened had the matching stage performed differently.
The UI is not polished, and knowledge of the internal algorithm is required, but it serves as a good tool when working on improving tracking.

### Parameter Optimisation
There are a large amount of parameters affecting tracking quality, and tuning them is an arduous process.
For that reason there exists an extensive configuration UI as well as tooling that may re-run tracking on critical sections of a replay whenever a parameter is changed. <br>
To access this tooling, open the "Parameters/Tracking" panel, and the section "Optimising Tracking Parameters" should appear in the "Control" panel when in Replay mode.
This tool stores several frame ranges that may be manually or automatically added (based on points where tracking is lost).
Then, whenever a tracking-relevant parameter is changed, these ranges are automatically revisited.
To aid in comparison between changed parameters, a baseline may be stored to compare against. <br>
It should be noted that just looking at a few short frame ranges may be fast and convenient, but may hide other areas where tracking was fine before, but critical after a parameter change.
Thus, before committing to a new set of parameters, it may be prudent to re-run tracking on one or multiple captures and comparing against previously stored tracking results across the whole replay.
This is discussed next.

### Comparing Tracking Results
In order to compare tracking results across a whole replay to a prior state (e.g. when changing parameters or tracker configuration), there exists a current tracking record, and a stored tracking record (originally created on initial capture of the replay).
These are visualised in the "Insights" panel, in the "Tracking" tab, using bright and muted colors respectively. <br>
Additional tooling for this can be found in the "Pipeline" panel, section "Tracking Results".
You can update the stored tracking record here, either temporarily to compare against, or overwriting the prior stored tracking record on disk.
You can also compare the current and stored tracking record numerically by pressing *Update Tracking Results* and expanding the resulting section. <br>
To get the most out of this tooling, the tracking needs to be deterministic.
Replay mode already does not skip frames, and instead slows down replay if required, but detection may still be asynchronous and thus differ from run to run.
So you may want to turn asynchronous search/probe off in the "Parameters/Tracking" panel, section "Target Detection".