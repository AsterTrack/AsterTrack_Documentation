# Architecture

This page serves to give you a rough understanding of the hardware and software architecture, focused on components involved in frame processing.
Some selected, complicated subsystems deserve a more detailed explanation.

## Hardware

<figure style="width:100%">
	<svg viewBox="0 0 1256 378" alt="A block diagram of major hardware components within camera, controller and host system.">
		<use xlink:href="../assets/details/HW_BlockDiagram.svg#document" />
	</svg>
	<figcaption>Hardware Block Diagram of major components <a href="../assets/details/HW_BlockDiagram.svg" target="_blank">View &#x2197;</a></figcaption>
</figure>

#### Power Management
By default, the cameras should receive only 5V, which powers the MCU for communication with the controller.
Only after identification may 10-24V be send to the camera, at which point the SBC (Raspberry Pi) may turn on.
This is done to not send high voltages to unsupported hardware accidentally plugged in (e.g. an ethernet device). <br>
Additionally, for wireless operation, 5V may also be supplied directly to the camera via a USB-C port.
The switching of these power sources on the camera is done purely in hardware. <br>
The controller is responsible to actively switch a camera port from 5V to VCC.
VCC is in turn being selected from the USB PD or External Power port which are being constantly monitored using two ADCs.
If configured, and External Power is not otherwise provided, the USB PD power may also be exposed on the External Power port for other controllers to be chained.

#### UART Communication
Both the SBC (Raspberry Pi) and MCU on the camera are set up to be able to receive and transmit on the UART. <br>
The MCU needs to communicate with the controller during startup, and the SBC needs to communicate with the controller once it has booted up.
During streaming, the MCU needs to receive timing packets, and the SBC needs to receive control packets from the controller. <br>
These are the design constraints, though they are further complicated by the unreliable PL011 UART Transceivers used in the Raspberry Pis.
They do not use DMA in the driver due to implementation issues, and the FIFO is too small to reliably receive packets at 8MBaud when the CPU is under load.
This means the Pi cannot be relied upon to handle any UART RX during streaming, and thus, all control packets are ideally routed through the MCU as well.
This is still in development. <br>
In practice, the UART is MUXed, defaulting to the Pi, allowing the Pi to communicate while the MCU is reset, flashed, or bricked.
But the UART RX bypasses the MUX, allowing the MCU to listen at all times.
While the MCU is running but the Pi has not yet connected via I2C, the MCU switches the MUX to claim UART TX control.

## Software

<figure style="width:100%">
	<svg viewBox="0 0 1268 361" alt="A block diagram of software components involved in frame processing across camera, controller and host system.">
		<use xlink:href="../assets/details/SW_BlockDiagram.svg#document" />
	</svg>
	<figcaption>Software Block Diagram focused on frame processing <a href="../assets/details/SW_BlockDiagram.svg" target="_blank">View &#x2197;</a></figcaption>
</figure>

#### QPU Processing (Camera)
Most of the image processing happens on the QPU of the [Video Core IV](https://github.com/hermanhermitage/videocoreiv) with the custom [VC4CV](https://github.com/Seneral/VC4CV) computer vision framework using the [vc4asm](https://github.com/maazl/vc4asm) assembler to be able to run handwritten assembly programs.
The QPU (Quad Processing Unit) is typically used for graphics processing, with 12 Single Instruction Multiple Thread (SIMT) cores, each operating on 16-way 32bit vectors (with 4 clock cycles per instruction), and extensive support for 8bit integer vector operations.
This means the throughput, and potential for optimisation, is immense compared to any CPU or GPGPU implementation.
Without the QPU and the extensive community-built tooling around it, this would never have been possible without FPGAs - and this project would not exist. <br>
The actual blob masking algorithm is using a combination of an absolute threshold, and relative threshold over the minimum of the 5x5 pixel neighbourhood of each pixel.
The idea is to detect faint blobs and edges with the relative threshold, and bright blob centers with the absolute threshold. <br>
This results in a bitmask that has to be searched for active bits.
Since the CPU currently spends over 2ms doing just that (even on images without any blobs), this is a prime target to offload to another coprocessor, the VPU - see e.g. [Frappe](https://bitbucket.org/simonj23/frappe) for examples of that.
After that, a standard tiled connected component labelling algorithm is applied on the CPU.

#### Blob Refinement (Camera)
After initial detection, blob refinements goal is to separate merged blobs and ensure the individual blobs position are accurately determined.
It can be visualised easily using the Blob Emulation feature on streamed (or recorded) camera images, so it won't be discussed in detail here.
While the accuracy is generally good, there is a lot of potential to improve accuracy in the cases of overlapping blobs.
Considering the strict performance requirements, it is unclear how much further it can be pushed.
Though currently, even this algorithm, called "resegmentation", is applied to all small blobs, so there is simiarly a lot of potential to increase performance headroom. <br>
For large blobs (comprised of more than 1000 pixels currently) there is an alternate blob refinement strategy that fits a circle to the image blob edge.
This can not deal with distortions well until it is upgraded to fit an ellipse, is not as accurate as a small marker is, and is not relied upon for most tracking tasks.

#### Packet Hub (Controller)
There is some consideration put into distributing the incoming UART packets from all 8 ports to the USB interface, though with the switch from multiple Full-Speed Interrupt Endpoints to a single High-Speed Endpoint, this has gotten a lot less complicated, with less focus on predicting the exact time of the next USB transfer. <br>
Still, incoming packets need to be efficiently distributed across the limited USB packets (a maximum of 8 packets of 1024 bytes every millisecond).
Large packets (currently of size 256 or higher) are being sent right from the UART buffer using the DMA controller.
Smaller packets are copied (CPU or DMA) into a shared packet to be sent together.
There is potential to extensively instrument, monitor, and optimise this subsystem, in an effort to improve both throughput and latency, but the current solution has in practice no troubles handling the full throughput of the 8 UART ports already.

#### Streaming Subsystem (Server)
The server receives the streaming data (referring to the realtime stream of blobs for each frame) from each camera in chunks.
To ensure low-latency processing, these chunks have to be timed individually, so that an informed decision can be made when processing should start, even in the absence of complete data by a camera.
Part of this is also the requirement that cameras announce that they are processing a frame, to prevent waiting on cameras that had to drop a frame due to potential processing shortfalls.
Currently, the system is configured to prioritise realtime use, with processing being started even when some cameras are still expected to send data if they exceed certain thresholds, and delayed data being discarded. <br>
In the future, delayed data will be properly recorded so that tracking in post using the [replay system](../operation/recording.md#replay) may perform better than in realtime.
Additionally, the thresholds determining the line between focus on low latency and focus on high quality tracking will be configurable in the future.

#### Camera Calibration (Pipeline)
While the final calibration (including continuous calibration) is mainly a result of optimisation alone, the initial reconstruction of camera transforms and linear lens parameters is more complex.
It is also a major part in determining how easy-to-use the camera system is, and what it may support in terms of large, multi-room tracking volumes - the latter should be supported already, though real-life testing has been sparse. <br>
The implemented reconstruction method only relies on a single marker and its correspondences across cameras.
Proper data aquisition is critical, and is handled in the Sequence2D subsystem - for both camera- and target calibration. <br>
This has been extensively researched and implemented as part of a Bachelor Thesis.
The final method being used is largely described in "A Convenient Multicamera Self-Calibration for Virtual Environments" by Tomáš Svoboda, Daniel Martinec, and Tomáš Pajdla, with prior work from, among others, David Jacobs, Peter Sturm, Bill Triggs, Hugh Christopher Longuet-Higgins and Richard I. Hartley.

#### Target Calibration (Pipeline)
While targets consisting of a set of spherical markers are easy to calibrate due to the markers being triangulatable, AsterTrack supports targets with flat markers as well, with the explicit intention being that they don't need to be seen by more than one camera to be useful. <br>
However, this greatly complicates target calibration of hand-crafted trackers, with the markers 3D position being initially estimated not by a reliable triangulation at a single point in time, but by multiple observations over time during movement that itself is estimated in an unreliable way.
The initial reconstruction is also currently completely insufficient, with the resulting estimate rarely being more than a random arrangement of markers, though quite some effort has been put into this algorithm.
That means, current target calibration mostly relies on good data aquisition, and compute-intensive optimisation, in hopes of getting matching estimates that can be merged to assemble the full set of markers that make up the target. <br>
A simpler and faster calibration method is planned for targets made up of just spherical markers that can all be triangulated at once. <br>
On top of that, designing a target in CAD allows you to bypass this entirely, as the marker data can be turned into a calibration directly.
There exists a exchange pipeline with Blender via Vertex Groups of a .obj mesh file that allows you to easily iterate on a target design in Blender and test it in simulation before 3D-printing it, though future work could make this even more useful and seamless.

#### Target Tracking (Pipeline)
Tracking is mostly an exercise of good prediction of a trackers movement, and then matching the expectation to the optical data available.
For targets, this means a 2D point matching algorithm that can handle severe outliers due to unpredictable movement. <br>
See the dedicated page on the [tracking algorithm](tracking_alg.md) for a lot more detail.