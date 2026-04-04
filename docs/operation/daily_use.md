# Daily Use

## Background Calibration
After you've connected to the hardware and started streaming using the buttons on the toolbar, it is recommended to calibrate the background to reduce the base load on the cameras and thus improve performance.
The cameras processing may even be overloaded just from close objects covering large parts of the field of view, with their close distance making them appear very bright. <br>

**Background Calibration** is currently a manual process at the start of every session, but there are plans to make this automatic in the future.
Though a fully manual background calibration with the care put into it will always be most reliable. <br>

First, hide or cover any moveable retroreflective markers and targets (trackers) from the view of the cameras as they may cause parts of the image to be ignored for the remainder of the session when they don't need to be. <br> 
Then, you may start the background calibration by using the button in the toolbar, or for individual cameras from the context menu in their camera view. <br>
This should start showing which areas of the camera view are being ignored for the remainder of the session.
Additionally, you should now see three buttons: Retry, Accept, and Discard. <br>
After a short moment after everything has settled, you may press the accept button to store the background mask. <br>

Relevant sources of background noise to be blocked may include the IR LEDs of other cameras, extraneous retroreflective material or clothing laying around, and windows if the sun is too bright - though closing the shutters or curtains is the preferred way to deal with that.

## Verifying Camera Calibration
Since there is currently no continuous calibration system that automatically corrects minor changes in the calibration, it is worth verifying the calibration is still good before every use.
To do so, enter the "Camera Calibration" phase in the "Pipeline" panel and wave around the calibration marker as if you were starting to collect samples.
The current reprojection error should be displayed as you accumulate samples.
While you may want to develop an own intuition on how accurate is accurate enough for tracking, you should definitely correct the calibration when the average pixel error is over 0.5px.
If that is the case, you can just continue collecting a few samples across the tracking volume and hit "Optimise" for a quick correction of the calibration.
You may also want to only calibrate the "Transform" using the Optimisation Options, the other parameters are unlikely to change and this allows you to use fewer samples to remain confident. <br>
Future work will make this either completely unnecessary with continuous calibration, or at least streamline the workflow so it takes less time before every use.