# Known Issues

## Firmware

#### Unreliable UART protocol
The camera currently relies on the PL011 UART to receive packets, but due to underlying implementation problems in the driver and the UART hardware itself, this is not reliable at 8MBaud. We are planning to route all UART packets through the MCU to make this rock-solid - all time-critical packets are already handled exclusively by the MCU. Until this has been implemented, there might be rare packet drops leading to misconfigured cameras or failure to start streaming entirely.

#### Controller Shared Packet Stalling
The controller has a [complex queueing system](../details/architecture.md#packet-hub-controller) to distribute UART packets from 8 ports to (potentially multiple) USB interrupt endpoints, often split up in parts. <br>
Shared Packets, as part of that system, may currently "stall" (be stuck in writing state) and fill up the queue.
This shows as "SharedFull" errors by the controller and is currently being investigated.

## Hardware

#### Slight whirring in the camera
The camera currently exhibits a slight whirring noise, likely some form of coil whine. This is only audible very close to the camera, and we are investigating the specific source of the noise and hope to eliminate it in a future version.

#### OV9281 Pixel Instability
During development it was discovered that the OV9281 has an inherent hardware fault, which can causes pixel instability at high pixel-to-pixel contrasts.
This has been verified using modules from other manufacturers and using standard software.
It is not detrimental to tracking quality but requires extra care when [focusing the lenses](../setup/camera_setup.md#focusing-the-lens), by essentially focusing the lenses well past the intended tracking range - a slight blur is intended and does not noticeably harm tracking quality beyond limiting how close markers can be together.

## Server Software

#### Camera Calibration Reconstruction may fail
The reconstruction may currently fail for non-obvious reasons, requiring you to re-aquire the calibration samples.
Whether this is a regression or caused by using many cameras is currently being investigated. <br>
Apart from that, there are also known unsupported configurations, like reconstructing cameras spread across multiple rooms.
This is just a matter of implementing the full set of recovery strageties and applying them recursively.
Currently, only one camera is selected as the center, and all cameras have to have some overlap with it.
Contact me (Seneral) if multi-room support is required and I will reprioritise it.