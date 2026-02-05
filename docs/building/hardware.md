# Hardware

If you are interested in Prototype Hardware, please contact [devkits@astertrack.dev](mailto:devkits@astertrack.dev). <br>
Building the hardware yourself is theoretically possible, but in practice quite unattainable for a reasonable price for individuals.
We are aiming to work on more DIY-compatible versions of the PCBs in the future after the current design is finalised. <br>
The block diagram of the system is rather simple (TBD), but in order to make the system actually stable and useful in real world use, the custom PCBs are a necessity, with their ESD protections, voltage regulations, and additional usability features.

### Implementation Details
AsterTrack has been designed around the custom hardware from the very beginning and it is essential for a performant and user-friendly multi-camera experience. <br>
Most importantly, the images need to be processed on-board the AsterTrack Cameras for a scalable, low-latency system that isn't constrained by data-throughput and host system capabilities.
That is what the Raspberry Pi Zeros are doing: They run a custom QPU assembly program processing the images at high speed and great efficiency, faster than a CPU could. <br>
After further refinement, the detected image blobs are then sent over a custom UART protocol through a CAT5e cable to the AsterTrack Controller, which is a custom board based on the CH32V307 chip.
Its purpose is to synchronise and communicate with up to 8 AsterTrack Cameras, and connecting to the host PC through a USB 2.0 HS connection.
While a direct protocol between cameras and host PC would have made some things easier (like an Ethernet connection), it would have made the setup UX much worse compared to a simple USB 2.0 device. <br>
Finally, the host PC receives the time-synchronised blob data from the cameras and can use it to calibrate and track objects in 3D space - see [camera calibration](../setup/camera_calib.md), [target calibration](../tracking/calib_target.md), and [tracking](../tracking/tracking_alg.md),

### DIY

A DIY version using off-the-shelf camera sensors is planned, but will always require custom PCBs.
While it is theoretically possible to build the hardware with just a CH32V307 EVT board, a Raspberry Pi, an OV9281 camera module from innomaker and some wires, it is insufficient for real-world use. <br>
For reliable communication over the CAT5e cables, the UART communication is wrapped in RS-422 differential signaling, and for stable operation, the power supply voltage through the cable is 12-24V - passing 5V straight through is not sufficient as the Raspberry Pi will suffer from undervoltage crashes. <br>
In addition to addressing these fundamental problems, the custom hardware also adds support for passive retroreflective markers via an IR LED ring around the camera lens, various buttons and signaling LEDs, power regulation with external power and USB PD support, external sync I/O, and other smaller amenities. <br>
All these require the custom PCBs, and thus sadly make true DIY (without an expensive PCBA-order) a lot less attractive - though if you're aware of the difficulties and risks, you are free to try.