# Operating the Camera System
Operating an AsterTrack camera system, whether for the first time or on a day-to-day basis, requires some prerequesite knowledge and procedures in order to be safe and effective.

## Safety
The cameras use Infrared LEDs to illuminate the room and filters to exclude visible light, primarily for retroreflective surfaces to appear bright in the image while keeping background illumination minimal. <br>
Infrared light (IR) can be **harmful** to the eyes, and thus it is important to know how and when IR light can cause damage.
Please inform yourself first while we work out official safety recommendations and guidelines. <br>
In general, we rely on the fact that short, but intense pulses of IR light pose the same danger as a continuous IR source with the same average power output.
That means, we can maximise the strength of the pulse while minimising the exposure duration for optimal isolation from other IR sources like windows letting in IR light from the sun. <br>
However, that also means that the hardware is theoretically capable of light output that might be seriously harmful to your eyes, and is only limited by the power regulators and software limitations.
The regulators are not designed to limit it to a safe power level, and the software limitations can not be guaranteed to be safe either before further testing with exact, final hardware. <br>
It is thus **your responsibility** to calculate the maximum exposure times for the exact hardware you are using that still allows for safe use, until we work out official safety guidelines for official hardware.
Until then, be cautious.

## For Every Use
After you've connected to the hardware and started streaming using the buttons on the toolbar, it is recommended to calibrate the background to reduce the base load on the cameras and thus improve performance.
Sometimes, the cameras may even be overloaded just from close objects covering large parts of the field of view, with their close distance making them appear very bright.
In those cases, background calibration may even be a necessity (see [known issues](known_issues.md)). <br>
**Background Calibration** is currently a manual process that involves removing any moveable retroreflective markers and targets (trackers) from the view of the cameras as they may cause parts of the image to be ignored when they don't need to be. <br> 
You then need to enter background calibration from the context menu of each camera individually, and press the accept button after a few seconds during which all cells with visible blobs are being blocked from processing.
This process is slated to be made easier very soon. <br>
Relevant sources of background noise to be blocked may include the IR LED ring of other cameras, extraneous retroreflective material or clothing laying around, and sometimes even windows if the sun is very bright (closing the shutters or curtains should mitigate this).

## Recording & Replay
You can record a raw capture for later replay, including blobs and images from the cameras and samples from the IMUs.
The controls for this can be found in the Control View, Section Recording.
You may start and stop a section which then gets added to the list where you may save it to disk, or you may save all frames of the current session (if they're still in memory) to disk at once. <br>
On disk, the capture is currently stored as in plain json as two files:
The raw capture `dump/XX_capture.json` and potentially an accompanying tracking record `dump/XX_tracking.json`, where XX is an increasing recording index. <br>
If the replay is intended to be loaded in the future, it is recommended to manually copy the current `store/camera_calib.json` to `dump/XX_calib.json`, and it will be automatically loaded with the capture.
This allows you to load and evaluate the replay even as your calibration changes - but as the cameras may be calibrated in retrospect, this is not done automatically to avoid confusion.

#### Extended capture naming scheme
While there exists no tooling for this currently, the capture and tracking file names may be appended by the following in this order:
- A number, to store a recording in parts that are automatically loaded together
- A name to be displayed when loading, added to at least one of the files of any given recording index

So a two-part capture may be comprised of the two files `XX_capture_0_recording_HMD.json` and `XX_capture_1.json`, or a full capture may be named `XX_capture_verification.json`, which would appear as the entries `Capture XX: recording_HMD` and `Capture XX: verification` respectively.

#### Replay
To load a capture, make sure you are currently not in simulation or device mode. <br>
Then, *Simulation/Replay capture* should list all captures, potentially with names, for you to load.
You can also append another capture if you already have one loaded - provided they share the exact cameras. <br>
The Control View should change completely, switching the Section Recording for Replay.
Now, you can Halt the replay, advance frame-by-frame, to the next image, or as quickly as possible, restart the replay from the beginning or even jump to a specific frame. <br>
Theres also a way to halt pipeline processing when a special Breakpoint-Snippet is executed, leaving the UI interactive for inspection.
This is currently not widely used but may be useful on a case-by-case basis, e.g. when inspecting when and why a tracker got lost [using visual tooling already in the UI](tracking.md#frame-inspection).