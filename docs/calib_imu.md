# Tracker IMU Calibration
Trackers may optionally have IMUs associated, either raw or fused.
These may be supplied by a separate driver or external source. <br>
Current hardware is based closely on SlimeVRs nRF-Slimes, modified to include timesync - this is an example of a fused IMU. <br>
External drivers supporting a custom VRPN protocol may also provide IMUs in return for tracking data.
This integration is being worked on for monado, which will be an example of a raw IMU. <br>
Currently, only the gyroscope is being automatically calibrated and partially used for tracking - that is, only raw IMUs are actually used.

### Associating an IMU
Some IMUs may be intrinsically associated to a tracker, e.g. if a VRPN client like monado is configured to connect to the AsterTrack server to override tracking and expose the IMU samples in return, that IMU will automatically be associated to the respective tracker. <br>
Others may need to be explicitly associated - in the future, this may be done automatically by way of detecting similar movement patterns by an optical tracker and connected IMU. <br>
To associate a tracker with an IMU, make sure the IMU appears in Device mode in both the Device View and the 3D view (even without streaming).
Then, open the Trackers View, select the tracker you wish to associate, and find the desired IMU in the dropdown of the IMU Config Section. <br>
After this, multiple stages of IMU calibration may be executed - mostly in the background right now, and not fully implemented yet - after which you may need to save the tracker configuration using the button in the Trackers View.
An UI for visualising the IMU calibration is planned.

### Gyroscope Calibration
Calibrating the rotational offset using gyroscope samples is currently implemented using two methods working alongside each other. <br>
The first uses periods of no movements to get calibrate a gravity vector, which together with optical measurements, can determine a rotational offset.
This requires the user to place the tracker in various orientations and keep it still for a bit. <br>
The other relies a lot more on good timesync, and accurate interpolation at the time of each frame to determine the rotational offset automatically from arbitrary movement. <br>
While this is desireable, currently, an exact, accurate timesync may not be guaranteed for all drivers, as they may exhibit a time offset compared to the optical data.

### Quaternion Calibration
Calibrating the rotational offset for fused IMUs is not currently implemented. <br>
Should end up being very similar to gyroscope calibration, just deriving rotational changes from the fused quaternion samples.

### Accelerometer Calibration
After the rotational offset has been calibrated, the accelerometer samples (provided by both raw and fused IMUs) may be used to calibrated the positional offset from the optical origin to the IMU. <br>
While this may be possible with traditional optimisation techniques, it may also be possible to implement this using a Kalman Filter estimating the offset over time. <br>
Whatever the method that ends up being used, it is not implemented yet.