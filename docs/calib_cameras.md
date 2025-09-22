# Camera Calibration
*internally also referred to as point calibration* <br>

The cameras need to be calibrated to be useful for tracking.
This involves determining their instrinsics - mostly lens parameters - and extrinsics - like position and rotation in the room. <br>
Ideally, this is only required once by the end user, though as long as continuous calibration is not yet implemented fully, it may be required from time to time, to correct for small errors. <br>
The complexity of the initial calibration depends on what exactly it is you need to calibrate:
If the lenses are known, some parameters (e.g. radial distortion) may already be determined and can just be used.
However, if you installed custom lenses, you may need to collect more data to sufficiently constrain these additional parameters.

### Reason for Calibration
It is important to consider what your reasons are for (re-)calibrating the camera system:

1. the error has increased over time
2. you added a new camera, moved one, or even set up in a completely new room
3. you changed or reinstalled the lens of at least one camera


### Collecting Samples
Make sure you've followed the [general procedures for operating the camera system](operating.md). <br>
Then enter the Camera Calibration Phase in the Pipeline View while streaming and wave a (preferrably small) spherical marker around the room.
If *Record* is checked, you should see samples being accumulated as observations (using the Sequence2D subsystem).
Note this is different from recording a raw capture (see [recording & replay](operating.md#recording-replay)). <br>
The recommended number of samples required varies with camera count and which reasons apply.
For reason 3, you need significantly more samples than for only reasons 1 and 2.
A rough range to aim for is 5000 to 50000 for camera systems of up to 8 cameras.

### Reconstruction
Only if reason 2 applies, you need to press *Reconstruct* after recording.
This will re-estimate the camera transforms and the lens field of view from scratch, but will re-use any existing parameters like lens distortions from prior calibrations.
Still, since this is only an estimation, the error may be around up to 5px for already calibrated cameras, and up to 20px for previously uncalibrated cameras. <br>

### Optimisation
The next step is Optimisation - which parameters you need to optimise depends on the relevant reasons.
See the tooltips for descriptions of each option. <br>
If only reason 2 applies, e.g. you only changed the camera setup, you may only want to optimise Camera Transform and Lens - the exact parameters estimated by reconstruction. <br>
In basically all other cases, it makes sense to optimise the Alignment parameters that are specific to each camera and its lens installation. <br>
Optimising (radial) distortions should generally be disabled - the parameters of the official lenses are known and absolutely fine to use as-is.
Only if you completely reset the calibration on purpose or installed new, unknown lenses should you optimise radial distortions anew. <br>
If all lenses are the same, it is highly recommended to share radial distortions - if the lenses are not all the same, collect a lot of samples and plan for way longer processing times.
In the future, it may be possible to select which lens each camera uses to share and optimise exactly the required parameters.