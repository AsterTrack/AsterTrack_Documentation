# Recording & Replay

## Recording
You can record a raw capture for later replay, including blobs and images from the cameras and IMU samples.
The controls for this can be found in the "Control" View, Section "Recording". <br>
Here, you can record individual sections or the entire past session (as long as it is still in memory). <br>
From the list of recorded sections, you can then save them to disk as two json files:
The raw capture `XX_capture.json` and potentially an accompanying tracking record `XX_tracking.json`, where XX is an increasing recording index.
To facilitate repeatable replay in the future even as calibrations change, an additional `XX_calib.json` is saved alongside with a snapshot of the calibrations of the involved cameras. You may delete that file or explicitly reload calibrations during replay to use the most up-to-date calibrations. <br>
By default, recordings are saved in the `dump` subfolder.

### Extended capture naming scheme
While there exists no tooling for this currently, the capture and tracking file names may be appended by two optional members, in the following order:

- A whole number, to store a recording in chunks and automatically load them together
- A display name, added to at least one of the files of any given recording index

So a single capture named `XX_capture_verification.json` would appear as the entry `Capture XX: verification`, and respectively, a capture comprised of the two files `YY_capture_0_VR_session.json` and `YY_capture_1.json` would appear as the entry `Capture YY: VR_session`.

## Replay
To load a capture, make sure you are currently not in simulation or device mode. <br>
The menu `Replay/Replay capture` will list all captures, potentially with names, which will replace any current replay upon loading. <br>
You can also append another capture if you are already in replay. If they have been captured in the same session, you may use `Append similar capture`, which will use the cameras and calibrations of the currently loaded replay, provided they match exactly. Otherwise, you should use `Append separate capture`, which ensures that all added cameras are unique, replacing their ID with a temporary one if needed. This allows them to have different calibrations, but makes saving any new calibration of them cumbersome. <br>
Appending similar captures is intended to aggregate samples of a session to improve camera calibration in post, and appending separate captures is intended to aggregate samples from a longer time period to improve target calibrations. <br>
Once a replay has started, the "Control" panel should change completely, exchanging the Section "Recording" for "Replay".
Now, you can halt the replay, advance frame-by-frame, to the next image, or as quickly as possible, restart the replay from the beginning or even jump to a specific frame. <br>
There is also a way to halt pipeline processing when a special Breakpoint-Code-Snippet is executed, leaving the UI interactive for inspection.
This is currently not widely used but may be useful on a case-by-case basis, e.g. when inspecting when and why a tracker got lost [using visual tooling already in the UI](../details/tracking_alg.md#frame-inspection).
Similarly, there are simple tools to simulate frame drops or occlusion of markers, and tools to disable cameras entirely, all to aid in tracking development.