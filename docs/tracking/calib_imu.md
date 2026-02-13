# Tracker IMU Calibration

Trackers may optionally have IMUs associated, which provide either raw or fused samples.
These may be supplied by an integrated driver or an external source like an [I/O protocol](integrations.md#io-protocols). <br>
The integrated drivers currently support experimental hardware based closely on SlimeVRs nRF-Slimes, modified to include timesync, providing fused IMU samples. <br>
External sources supporting a custom [VRPN](integrations.md#vrpn) protocol may also provide IMUs in return for tracking data.
Such an integration is currently being worked on for monado, providing raw IMU samples. <br>
These IMUs need to be calibrated to align with the optical tracker, which requires extensive algorithms to do automatically.
Currently, only the gyroscope is being automatically calibrated and partially used for tracking - meaning only IMUs providing raw samples can currently be used.

## Associating an IMU
Visit the "Trackers" panel to see and edit a [trackers IMU association](tracker_config.md#imu-calibration). <br>
Some IMUs may be intrinsically associated to a tracker, e.g. if a VRPN client like monado is configured to connect to the AsterTrack server to override tracking and expose the IMU samples in return, that IMU will automatically be associated to the respective tracker. <br>
Others may need to be explicitly associated - in the future, this may be done automatically by way of detecting similar movement patterns by an optical tracker and connected IMU. <br>
After this, multiple stages of IMU calibration may be executed - mostly in the background right now, and not fully implemented yet - after which you may need to save the tracker configuration using the button in the "Trackers" panel. <br>
An UI for visualising the IMU calibration is planned.

## Gyroscope Calibration
Calibrating the rotational offset between the IMU and the optical tracker using gyroscope samples is currently implemented using two methods working alongside each other. <br>
The first uses periods of no movements to calibrate a gravity vector, which together with optical measurements, can determine a rotational offset.
This requires the user to place the tracker in various orientations and keep it still for a bit. <br>
The other relies a lot more on good timesync, and accurate interpolation at the time of each frame to determine the rotational offset automatically from arbitrary movement over time. <br>
While the latter method is more approachable, an exact, accurate timesync may not be guaranteed for all drivers, as they may exhibit a time offset compared to the optical data.
In that case, the former method is much more reliable and may in turn become a crucial part in calibrating that time offset.

## Quaternion Calibration
Calibrating the rotational offset for fused IMUs is not currently implemented. <br>
This should end up being very similar to gyroscope calibration, just deriving rotational changes from the fused quaternion samples.

## Accelerometer Calibration
After the rotational offset has been calibrated, the accelerometer samples (provided by both raw and fused IMUs) may be used to calibrated the positional offset from the optical origin to the IMU. <br>
While this may be possible with traditional optimisation techniques, it may also be possible to implement this using a Kalman Filter estimating the offset over time. <br>
Whatever the method that ends up being used, it is not implemented yet.