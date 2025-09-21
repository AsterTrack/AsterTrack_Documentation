# Known Issues

## Hardware / Firmware

#### USB Power Delivery only works when plugged after controller is powered
On startup, the controller currently cannot verify with the USB PD Controller if it already has set up PD, and Voltage Reading via ADC is also not implemented yet, so currently it just assumes PD has been configured previously. <br>
In the future, we may want to consider whether a controller reset (which should not be necessary) should cut the power deliberately and the controller should always reset and reconfigure PD on startup - this has previously been undesireable to allow for quick resets during development without restarting the cameras. <br>
To make it work right now, you need to plug in PD after the USB Data cable that powers the controller, and PD should continue working until you unplug it.

#### Unreliable camera streaming startup
Starting the streaming of cameras can still fail, and with increasing number of cameras it seems to be worse.
Only remedy right now is to stop and start streaming until it works.
The reasons for this are, to my knowledge, three-fold, described as follows in their own issues:

#### Unreliable UART protocol
The UART protocol has a not insignificant amount of bit-errors (whether 8MBaud or 4MBaud), and the protocol currently doesn't acknowledge packets or resend erroneous ones.
This most often only affects the larger image packets and is thus non-critical, but sometimes it does affect streaming packets and, worse, setup packets, which may be silently dropped, leaving the camera unable to start.
This will be addressed.

#### Frame number desync
Synchronisation of frameIDs can currently break, resulting in a flood of "missed" frames when in reality they're just labelled wrong.
The reason for this is that the Pi estimates the frame ID to assign to a frame using past frames and the SOF packets send by the controller.
However with the possibility of dropped frames, irregular frames intervals (when using external sync for professional setups), and uncertain arrival times of the frames from the camera stack (which are not labelled), there's quite a bit of uncertainty. <br>
This may in the future be remedied by better estimations using precise times of the underlying camera stack or, if possible, exact frame timestamps by V4L2.

#### Rare camera crashes
When cameras are overloaded by too much bright area, they are supposed to abort processing before they drop frames.
But the estimates for that are rough, and this limit is checked before the refinement stage, so processing time regularly exceeds frame times anyway, resulting in frame drops. <br>
Sometimes, this enables the frame number desync issue to appear, and rarely, this may even crash the camera - the exact cause is still unknown.
Often, this may be a QPU stall, which is notoriously hard to debug. <br>
The best way to counter this right now is to calibrate the background to avoid excess bright areas due to static objects.
If this issue happens right when streaming starts however, you may need to temporarily lower the exposure times, then during background calibration, put them back up to properly calibrate the background. <br>
In the future, the processing headroom in the cameras will greatly increase due to further optimisations that are easily possible, and the background may even be automatically calibrated.

#### Controller Shared Packet Stalling
Shared Packets may currently "stall" (be stuck in writing state) and fill up the queue.
This shows as "SharedFull" errors by the controller and is currently being investigated. <br>
For context, Shared Packets are used for small controller->host and camera->host packets to share space in a single USB packet.
They are immediately queued after writing, but any future writer may "lock" any packet in the queue except the one in front in a scheme to reduce latency but maximise throughput.
This additional complexity may introduce said bugs.
Note that they share the queue with DMA packets (larger packets from camera->host, DMAd straight from the UART receive buffer) and thus compete for USB bandwidth.

## Server Software

#### Camera Calibration Reconstruction may fail
The reconstruction may currently fail for non-obvious reasons, requiring you to re-aquire the test data.
Whether this is a regression or caused by using many cameras is currently being investigated. <br>
Apart from that, there are also known unsupported configurations, like reconstructing cameras spread across multiple rooms.
This is just a matter of implementing the full set of recovery strageties and applying them recursively.
Currently, only one camera is selected as the center, and all cameras have to have some overlap with it.
Contact me (Seneral) if multi-room support is required and I will reprioritise it.