# Target Calibration
A tracker may use a fixed set of markers, known as a target, as its optical component.
Determining the marker geometry and - in the case of flat markers - orientation, is required before a target can be tracked. <br>
The current algorithm is suitable for both targets comprising of both spherical and flat markers, and works by first reconstructing short segments as Target Views, then iteratively merging them to pinpoint all markers precisely.
This process is long and arduous currently, and might even require manual intervention still.

### Target View Aquisition
When streaming and entering the Target Calibration Phase in the Pipeline View, the pipeline automatically searches for good views of the target to be calibrated.
This can be visualised using the Insights View, in the Target Calibration/Aquisition Tab.
You will need to move the target around for this process to work, and in general it is best to focus on rotation, and just try to keep the target in the view of as many cameras as possible.
While 3 to 4 cameras are generally sufficient for target calibration, more cameras add much needed constraints that make the process easier and more reliable. <br>
When a segment is deemed valuable, it is added to the Target View list and automatically reconstructed, optimised.
This processing is quite intense and it is not uncommon for 100% of the CPU to be used as more and more Target Views are added. <br>
If, after optimising, a Target View reaches a sufficient level of quality - indicated by being automatically checked for inclusion in the next stage - it will also be automatically processed further to merge markers.
Depending on the complexity of the target, 4 to 8 views may be required in the next stage, though you can always go back and retry reconstruction of views that didn't make the cut the first time.
The algorithm is currently mostly brute-force and non-deterministic, so there is a good chance this does work.

### Target Assembly
After some Target Views of sufficient quality are done processing, you may press 'Auto-Assembly' to start assembling multiple views into one final target. <br>
This is done by first picking a base view (which you can override by manually selecting a view and pressing *Pick as Base View*) and then iteratively aligning other views and merging one at a time. <br>
This merging is itself done in multiple steps:

1. Partially merging view: Merge markers deemed the same during the aligning step, then adopt all frames of the view that have enough samples from just these shared markers.
2. Integrate all frames: Adopt remaining frames and reevaluate sequences of those frames against current markers, adding new samples in the process.
3. Adopt new markers: Adopt remaining markers and their samples.

In between, the target is optimised, which may or may not be stored as its own separate stage, depending on its usefulness. <br>
Notably, next to the *Optimisation* stages, there's the *Reevaluate Markers* stage, which is employed to merge markers that have, over time and optimisation, accumulated evidence that they are one and the same.
It may also add previously unassociated sequences that are now deemed to correspond to a marker, or remove existing ones that have since been deemed to not match.

#### Visual Verification
By selecting individual stages, you can visually inspect them usingh both the 3D View and Insights panel. <br>
Tip: Use the *Orbit View* button in the top left of the 3D View to focus on the target. <br>
The *Target Calibration* Section in the Visualisation Panel of the 3D View provides additional visualisation options:

- Showing the currently estimated Field of View of each markers
- Showing the underlying observed samples of each marker
- Focusing the view on selected markers

In the Insights View, *Target Calibration, Target Markers* Tab you can see the samples of each marker, and by hitting space you can replay the frames used as data. <br>
Both in the 3D view and the Insights view, you can focus and/or select markers, which will become relevant for editing. <br>
For very close inspection, it is possible to enter manual editing mode just to see exactly which sequences make up each marker, and potentially correct errors.

#### Manual intervention
You may, at any point, stop the automatic assembly, and intervene manually with provided editing tools.
It may then be restarted from where it left off. <br>
You may also use the two buttons on the right of the stage to discard progress beyond one stage - e.g. when some incorrect step has been performed, like valid markers being removed or two valid markers being merged. <br>
The editing tools include manually adding individual stages like *Reevaluate Markers* and *Optimisation*, but also ones not used automatically (and not recommended due to being destructive) like *Expand Frames* and *Subsample Data*. <br>
But the most powerful one is *Manually Edit Target*, which allows you to select markers (in both 3D view and Insights view) and edit them directly:

- Delete marker observations, as previously aquired by the Sequence2D subsystem
- Move marker observations between two selected markers
- Merge two selected markers and their marker observations
- Split one selected marker into two, allowing you to move one or more marker observations into it

By selecting marker observations, you can visually inspect them in the 3D View and check visually whether they plausibly belong to another marker. <br>
If you check *Show Marker Observations* in the visualisation panel (see above), hovering over a sequence also highlights it in the 3D view.

### Recommended: Use Recording & Replay
I highly recommend you to record the Target View Aquisition by pressing *Start Section* in the Control View, and *Stop Section* after you're done, then saving the section in the table below - see details on [recording & replay](operating.md#recording-replay).
Then, you can load the capture as a replay and continue with Target Calibration as instructed, ensuring that no Device Mode specific bug crashes and interrupts the processing (which may take hours). <br>
You can also at any point save the processing of Target Views and Target Assembly: <br>
**Target Views** are stored in one shared file (*target_calib_views.json*) alongside the underlying observation sequences (*target_calib_observations.json*) and are only designed for one concurrent Target Calibration. <br>
**Target Assembly Stages** are stored as new files labelled in increasing order (*assembly_stage_XX.json*) alongside the same shared underlying observation sequences (*target_calib_observations.json*) <br>
In order to restart target calibration, load the exact same replay, load the views (which may optimise for a short bit to re-establish outliers), load the latest assembly stage, and then press *Auto-Assembly*, which should continue where it left off.
