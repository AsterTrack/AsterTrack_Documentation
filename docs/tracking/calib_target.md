# Tracker Target Calibration

A tracker may use a set of markers, called a target, as its optical component.
Determining the marker geometry and - in the case of flat markers - orientation, is required before a target can be tracked. <br>
The current algorithm is suitable for targets comprising of both spherical and flat markers, and works by first reconstructing short segments of observations as Target Views, then iteratively merging them to pinpoint all markers precisely.
This process is currently long and arduous, and might even require manual intervention still. <br>
For this reason, it is **highly** recommended to record your target view aquisition and [perform the processing in a replay](#work-with-recordings).
This reduces the chances of an unforseen crash disrupting the work. <br>
The following is an indepth explanation including some technical insights, as currently such understanding may still be beneficial to resolve issues as they pop up.
This process may be fully automated in the future as the automatic process is improved and confidence is built that it handles all targets well. <br>
To be clear: This process is only needed once for each target - if you design a target and then reproduce it using e.g. 3D printing, the end user will not need to calibrate it themselves - though a slight optimisation might be beneficial to accommodate variations.

## Target View Aquisition
When streaming and entering the "Target Calibration" phase in the "Pipeline" panel, the pipeline automatically searches for good views of the target to be calibrated.
This can be visualised using the "Insights" panel, in the "Target Calibration/Aquisition" tab.
You will need to move the target around for this process to work, and in general it is best to focus on rotation, and just try to keep the target in the view of as many cameras as possible.
While 3 to 4 cameras are generally sufficient for target calibration, more cameras add much needed constraints that make the process easier and more reliable. <br>
Essentially, it monitors continuously visible marker sequences and searches for segments where many markers are visible with sufficient samples per marker to rely on.
Smooth, continuous, and unique rotation of the target to calibrate aid greatly here. <br>
When a segment is deemed valuable, it is added to the "Target Views" list and automatically reconstructed and optimised.
This processing is quite compute intensive and it is not uncommon for 100% of the CPU to be used as more and more target views are added.
If, after optimising, a Target View reaches a sufficient level of quality, it will also be automatically processed further to merge markers, and automatically selected for inclusion in the next stage via the checkmark in the first column.  <br>
Depending on the complexity of the target, 4 to 8 views may be required in the next stage, though you can always return to add more target views if you find them insufficient.
This can be done by recording more target view samples in a separate instance and appending the capture as a replay to the processing instance.
Or you can retry reconstruction of existing target views that didn't make the cut the first time.
The algorithm is non-deterministic due to relying on an unreliable initial estimate and iterative optimisation, so there is a good chance this does work, and should be your first avenue to try. <br>
You can select target views to [visually inspect](#visual-inspection) their current state.
This can be used to verify that a target view with low error is not just overfitted, but actually represents parts of the structure of the target you wish to calibrate.

## Target Assembly
After some target views of sufficient quality have finished processing, you may press 'Auto-Assembly' to start assembling multiple views into one final target.
It will first heuristically pick a base view to start with, which you can override manually using the UI, and then iteratively align other views and merge them one at a time. <br>
This merging is itself done in multiple steps:

1. Partially merging view: Merge markers deemed the same during the alignment step, then adopt all frames of the view that have enough samples from just these shared markers.
2. Integrate all frames: Adopt remaining frames and reevaluate observed marker sequences of those frames against current markers, adding new samples in the process.
3. Adopt new markers: Adopt remaining markers of the target view as new markers.

In between, the target is optimised, which may or may not be stored as its own separate stage, depending on its usefulness. <br>
Notably, next to the *Optimisation* stages, there's the *Reevaluate Markers* stage, which is employed to merge markers that have, over time and optimisation, accumulated evidence that they are one and the same.
It may also add previously unused marker sequences that are now deemed to correspond to a marker, or remove existing ones that have since been deemed to not match. <br>
Similar to target views, you can select assembly stages to [visually inspect](#visual-inspection) their current state.

### Visual Inspection
When you select either a target view or a target assembly stage, you can visually inspect their current state in both the 3D view and the "Insights" panel: <br>
The *Target Calibration* Section in the "Visualisation" panel of the 3D view provides additional visualisation options:

- Showing the currently estimated Field of View of each marker
- Showing the samples associated with each marker as observation rays
- Focusing the view on selected markers

In the "Insights" panel, "Target Calibration/Target Markers" tab you can see the samples of each marker, and by hitting space you can replay the frames used as data. <br>
Both in the 3D view and the "Insights" panel, you can focus and/or select markers, which is primarily relevant for editing. <br>
For stages, you are able to unlock more visualisation options by entering manual editing mode, e.g. to see exactly which marker sequences make up each markers observations, and potentially correct errors.

### Manual Intervention
You may at any point stop the automatic assembly and intervene manually using the provided editing tools.
You may then restart automatic assembly, and it will continue from where it left off. <br>
You may also use the two buttons to the right of a stage to discard progress beyond that stage - e.g. when some incorrect step has been performed, like valid markers being removed or two valid markers being merged. <br>
Further, you may manually insert stages like *Reevaluate Markers* and *Optimisation*, but also ones not used automatically due to being destructive, like *Expand Frames* and *Subsample Data*. <br>
The most powerful editing tool to fix mistakes is *Manually Edit Target*, which allows you to select markers (in both the 3D view and "Insights" panel) and edit them directly.
The individual marker observations that make up the selected markers are listed in the UI, and you can: 

- Delete marker observations, as previously aquired by the Sequence2D subsystem
- Move marker observations between two selected markers
- Merge two selected markers and their marker observations
- Split one selected marker into two, allowing you to move one or more marker observations into it

With a marker selected, you can additionally select individual marker observations and visually check in the 3D view whether they plausibly belong to another marker. <br>
You may use the option *Show Marker Observations* in the "Visualisation" panel of the 3D view (see [visual inspection](#visual-inspection)), adding the additional ability to quickly check a marker observation in the 3D view by merely hovering over it in the UI.
This visualisation also makes marker observations that are misattributed to a different marker very obvious to the human eye.

## Work with Recordings
it is **highly** recommended that you record the target view aquisition by using the [recording & replay tools](../operation/recording.md).
Then, you can load the capture as a replay and continue with Target Calibration as instructed, ensuring maximum stability and flexibility, and ensuring that foundational parts of the processing, like the Sequence2D subsystem, are deterministic. <br>
You can also at any point save the processing of Target Views and Target Assembly: <br>
**Target Views** are stored in one shared file (*target_calib_views.json*) alongside the shared observation sequences (*target_calib_observations.json*) and are only designed for one concurrent Target Calibration. <br>
**Target Assembly Stages** are stored as new files labelled in increasing order (*assembly_stage_XX.json*) alongside the same shared observation sequences (*target_calib_observations.json*) <br>
**Observation Sequences** are the underlying collected data used by all calibration systems, and you need that file (*target_calib_observations.json*) to stay intact in order to be able to load the others.
If you overwrite this file, you may be able to recreate it, but that relies on the Sequence2D subsystem to be deterministic.
This means it might not work across different program versions, and it assumes you started Target View Aquisition in the exact same frame.
That is why it is **highly** recommended you do the initial processing in replay mode as well, and enter Target Calibration before starting the replay, ensuring Sequence2D is active from the very first frame recorded. <br> 
**Restarting Target Calibration** is then simply a matter of loading the exact same replay (including any appended captures you may have recorded), load the Target Views (which may optimise for a short bit to re-establish outliers), load the latest saved assembly stage, and then press *Auto-Assembly*, which should continue where it left off.

## Editing Calibrations
To edit an existing calibration, you need to first collect samples by tracking it in the "Tracking" phase.
Then, in the "Optimisation Database" section, select the entry for the tracker and click "Edit Target".
This will adopt the target as a target assembly stage in the "Target Calibration" phase. <br>
While the data used in the target assembly is usually backed by marker observation sequences, now it is backed up by individually tracked samples.
This excludes this data from being used in some of the algorithms of target assembly. <br>
Tracking easily yields too many samples for optimisation routines to handle, so it is enouraged to hit the "Subsample Data" button if the sample count exceeds 50k.
While this is not a destructive action, some tools that benefit from as many samples as possible, like view cone estimation, will not be able to take advantage of them anymore. <br>
Note that if you plan to amend the calibration with markers you added after initial calibration, you may need to collect more target views, and start auto-assembly again to merge them into the existing calibration.