# Camera Calibration

*internally also referred to as point calibration* <br>

The cameras need to be calibrated to be useful for tracking, which involves determining a multitude of lens-specifc, camera-specific, and setup-specific variables. <br>
Ideally, this is only required once for initial setup, though as long as continuous calibration is not yet implemented fully, it may be required from time to time, to correct for small errors, and will always be beneficial before important work.

## Calibration Steps

Follow these steps to calibrate the camera system.
Note that the **Reconstruction** and **Room Calibration** are generally only required on first setup, though their exact requirements vary.

**Collecting Samples:** First you need to collect samples to use for calibration. Make sure you've followed the general procedures for operating the camera system, especially [background calibration](../operation/background.md). <br>
Then enter the "Camera Calibration" Phase in the "Pipeline" panel while streaming and wave a small spherical calibration marker around the room.
As long as *Collect* is checked, you should see samples being accumulated from continuously visible marker sequences.
Try to get good coverage of all cameras, and cover all the space you intend to track.
Do this relatively slowly in the corners to allow for enough samples to be accumulated there.

**Reconstruction:** Only required if you are calibrating after initially setting up the cameras, added cameras, or moved one or more cameras.
If you are only looking to improve an existing calibration, skip this step entirely.
Otherwise, you need to estimate the rough positions of the new cameras by hitting "Reconstruction". <br>
Reconstruction will re-estimate the camera transforms and the lens focal length from scratch, but will not touch any other previously calibrated parameters.
Still, since this is only an estimation, the pixel error may be significant, especially for cameras with yet unknown lens parameters. <br>

**Optimisation:** To fine-tune the calibration, you need to press "Optimisation" to start the compute-intensive process of iteratively improving the calibration. <br>
By default, this will only optimise some parameters, fixing radial lens distortions to factory defaults to speed up optimisation and making it possible to use fewer samples.
In case you want to change this or other optimisation parameters, see [below for optimisation options](#optimising-options).

**Room Calibration:** Only required if you are calibrating after initially setting up the cameras or after moving most of them, such that no 2 cameras remain untouched.
In that case, the room calibration can not be transferred from your prior calibration to your new one.
Though if in doubt, redoing them when changing many cameras is not a bad idea. <br>
The purpose of room calibration is two-fold: First, the scale of the camera system has to be determined.
This can not be done with just the one calibration marker alone, and so the system can not properly detect and track precalibrated targets as long as the scale is uncalibrated.
Second, the orientation and floor plane of the camera system needs to be determined, to ensure proper alignment. <br>
To do both, you need to place the calibration marker on the floor one by one, recording each position as you do so in the UI.
The first two markers need to be placed with care, preferrably both in view of many cameras, and their distance needs to be measured precisely, from one marker center to the other.
By default, their distance is expected to be exactly 1000mm (1m) apart.

## Custom Lenses

After following the steps to [install a custom lens](camera_setup.md#custom-lenses), you need to calibrate that lens to determine its intrinsic parameters.
The calibration optimisation system makes use of lens presets to know which lens are expected to be the same and, if desired, share their radial distortion parameters to constrain the solution.
This means that, when changing the lens, you need to explicitly set the lens used - or at least ensure it is not using another lens preset.

#### Setting Lens Presets
You can see the individual cameras calibrations by expanding the "Cameras" menu in the "Camera Calibration" section, showing detailed information in the info tooltip.
Here, you can also inspect the lens preset associated with each camera, change them, or create a new one from the current calibration - you can update that preset later as you calibrate the new lens. <br>
When setting up a new custom lens, create a new lens preset with "Make Preset" from the dropdown, and assign it to the relevant cameras using that lens.
This will improve the speed and reliability of the optimisation later on.

#### Optimising Options
To calibrate a new lens, you need to open the "Optimisation Options" and enable the toggle "Distortions". The rest is recommended to be kept as-is. <br>
*Transform*, *Lens*, and *Align* are the basic set of parameters describing the specific setup and lens installation. <br>
*Distortions* enables optimisation of radial lens distortions.
If you keep *Share* enabled, the radial distortion parameters of all cameras sharing a lens preset will be grouped and optimised together.
Any cameras with no lens preset associated will always be optimised individually.
The last set of buttons selects the order of radial distortions used - if you lower this, the remaining parameters will be reset to 0, not just ignored. <br>
In practice, radial distortions are completely fine to share between lenses of the same make, and it requires a lot of samples to improve upon the shared default.
Worse, if radial distortions are calibrated with too few samples, or not shared across multiple cameras, their parameter space may be underconstrained, allowing the optimisation routine to overfit, resulting in inaccurate parameters despite having a low error.

#### Updating Lens Presets
After the calibration of the camera system with sufficient samples, you may want to update any shared lens presets so you can re-use it in the future if you install this lens in more cameras. <br>
You can do that by expanding "Lens Presets", and hitting the update button of the lens preset you wish to update.
This will fetch the radial distortions of all cameras used in this session that reference this lens preset, and average them if required (e.g. Share was not enabled). 
You can also edit the lens preset name or delete them by first entering edit mode with "E", and you can select the default lens preset using the right column. <br>
Finally, to save your changes to the presets to disk, you need to explicitly hit the "Save Lenses" button.
If you ever accidentally overwrite the default lens presets, they can be manually restored from the `lens_presets_builtin.json` file.