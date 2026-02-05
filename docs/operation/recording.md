# Recording & Replay

### Recording
You can record a raw capture for later replay, including blobs and images from the cameras and samples from the IMUs.
The controls for this can be found in the Control View, Section Recording.
You may start and stop a section which then gets added to the list where you may save it to disk, or you may save all frames of the current session (if they're still in memory) to disk at once. <br>
On disk, the capture is currently stored as in plain json as two files:
The raw capture `dump/XX_capture.json` and potentially an accompanying tracking record `dump/XX_tracking.json`, where XX is an increasing recording index. <br>
If the replay is intended to be loaded in the future, it is recommended to manually copy the current `store/camera_calib.json` to `dump/XX_calib.json`, and it will be automatically loaded with the capture.
This allows you to load and evaluate the replay even as your calibration changes - but as the cameras may be calibrated in retrospect, this is not done automatically to avoid confusion.

### Extended capture naming scheme
While there exists no tooling for this currently, the capture and tracking file names may be appended by the following in this order:

- A number, to store a recording in parts that are automatically loaded together
- A name to be displayed when loading, added to at least one of the files of any given recording index

So a two-part capture may be comprised of the two files `XX_capture_0_recording_HMD.json` and `XX_capture_1.json`, or a full capture may be named `XX_capture_verification.json`, which would appear as the entries `Capture XX: recording_HMD` and `Capture XX: verification` respectively.

### Replay
To load a capture, make sure you are currently not in simulation or device mode. <br>
Then, *Simulation/Replay capture* should list all captures, potentially with names, for you to load.
You can also append another capture if you already have one loaded - provided they share the exact cameras. <br>
The Control View should change completely, switching the Section Recording for Replay.
Now, you can Halt the replay, advance frame-by-frame, to the next image, or as quickly as possible, restart the replay from the beginning or even jump to a specific frame. <br>
Theres also a way to halt pipeline processing when a special Breakpoint-Snippet is executed, leaving the UI interactive for inspection.
This is currently not widely used but may be useful on a case-by-case basis, e.g. when inspecting when and why a tracker got lost [using visual tooling already in the UI](../tracking/tracking_alg.md#frame-inspection).