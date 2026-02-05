# Background Calibration

After you've connected to the hardware and started streaming using the buttons on the toolbar, it is recommended to calibrate the background to reduce the base load on the cameras and thus improve performance.
Sometimes, the cameras may even be overloaded just from close objects covering large parts of the field of view, with their close distance making them appear very bright.
In those cases, background calibration may even be a necessity (see [known issues](issues.md)). <br>
**Background Calibration** is currently a manual process that involves removing any moveable retroreflective markers and targets (trackers) from the view of the cameras as they may cause parts of the image to be ignored when they don't need to be. <br> 
You then need to enter background calibration from the context menu of each camera individually, and press the accept button after a few seconds during which all cells with visible blobs are being blocked from processing.
This process is slated to be made easier very soon. <br>
Relevant sources of background noise to be blocked may include the IR LED ring of other cameras, extraneous retroreflective material or clothing laying around, and sometimes even windows if the sun is very bright (closing the shutters or curtains should mitigate this).
