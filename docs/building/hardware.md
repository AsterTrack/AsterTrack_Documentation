# Hardware

Currently, the only way to get a hold of the hardware is through the first batch of dev kits that is currently being planned. <br>
If you are interested, please contact [devkits@astertrack.dev](mailto:devkits@astertrack.dev).

## Development Philosophy
This project is open source, and will also be open hardware in the future.
However, contrary to most open source projects, we decided to develop this in relative silence until it was in a state we were comfortable sharing for people to use. <br>
The reasoning is two-fold: <br>
First, the hardware requires significant upfront investment to build.
Not only does it use multiple expensive components like Raspberry Pis and the OV9281 camera module, it also relies on custom PCBs, with no viable option to only use off-the-shelf development boards to reduce the complexity. <br>
Second, we repeatedly failed to accurately estimate a timeline early on (almost exclusively due to software development).
We were not comfortable with people investing this much money and time into a project based on promises alone, if we could not give even loose guarantees on when it would live up to the potential we envisioned for it. <br>
Years have passed, more than was ever intended, and there is still a lot of potential for improvements in both hardware and software even now.
But at least we can be sure that early adopters and developers who decide to spend money on the first batches do so knowing what they will receive.
The base multi-camera experience is solid, with usability mostly determined by the features required for a particular use case - and we intend to discuss with every early adopter to ensure their requirements are currently met.

## DIY Version
While the core functionality of the hardware is deceptively easy to build from scratch (see [first prototype hardware](https://www.seneral.dev/astertrack.html#prototype-hardware)), the [block diagram of the system](../details/architecture.md#hardware) hints at the complexities that result from making the system actually stable and useful in real world use.
As such, the custom PCBs are considered a necessity, with their essential improvements to system stability and usability. <br>
That means, our approach to DIY will always require ordering the PCBs, either from us or from industry-standard PCBA partners.
Additionally, we are currently using a custom camera module which includes a filter switcher.
While we plan to make those available individually in the future, more likely we would want to create a separate PCB version intended to be used with off-the-shelf modules like those from [innomaker](https://www.inno-maker.com/product/cam-mipi9281raw-v2/), and a different case to go along with that form factor.
We are aiming to work on this more DIY-compatible version of the PCBs in the future after the current design is finalised, to allow for iteration on the hardware design without huge capital expenses. <br>
And there is potential for iteration that we are excited about: Various different camera sensors like the SC132GS, purpose-built versions, perhaps even other processing solutions using FPGAs - there is a lot of room to improve still.

## Driving Design Decisions
AsterTrack has been designed around the custom hardware from the very beginning and it is essential for a performant and user-friendly multi-camera experience.
Some of those defining decisions will be explained here at a high level.
You can find some more technical details in the [hardware architecture page](../details/architecture.md#hardware).

##### On-Board Processing
One of the core decisions is that the frames need to be processed on-board the AsterTrack Cameras for a scalable, low-latency system that is not constrained by data-throughput and host system capabilities.
Otherwise, you end up with a system that needs to use tricks to achieve a usable framerate on most consumer systems, might require purchasing of additional hardware to ensure available data throughput, has a harsh upper limit for the number of cameras supported, and takes away processing resources from accompanying software like VR applications. <br>
This is why the decision was made to build a [custom high-efficiency frame processing pipeline](../details/architecture.md#qpu-processing-camera) on the Raspberry Pis GPU - it is one of the few cheap SBCs that allow for such low-level access to its hardware to be able to optimise for one use case.
That means we are currently locked into requiring a Raspberry Pi with the Video Core IV - unless the whole processing pipeline is replaced.
This only makes sense when considering the use of FPGAs in the future.

##### Wired Communication
Another important design decision was to focus on wired cameras, despite their obvious usability concerns, as consumer wireless systems are too unreliable for such a low-latency and high-bandwidth application.
We expect multiple cameras to send blob and image data to a single system, with the synchronisation essentially guaranteeing collisions - the worst case for any wireless protocol. <br>
Instead, we are using the Raspberry Pis built-in UART and use a custom controller to pass the data to the host system via USB, which the CH32V307 chip allowed us to do. The alternative, using Ethernet and PoE switches, does have some advantages, but would have increased costs and negatively impacted the user experience. <br>
Wireless operation is planned despite all this, but it will never be reliable for all users, and can not be relied upon as the sole method of communication.
Instead, it is reserved as an option for cameras that are difficult to hardwire.

## Required Hardware
You need at least 3 AsterTrack Cameras and one AsterTrack Controller for a minimal setup. You can add more cameras and even controllers at any point, with each controller supporting up to 8 cameras. <br>
Future features like IMU integration may require wireless dongles and IMU hardware itself, and future wireless operation may require a similar wireless dongle as a sync beacon. <br>
Dev Kits may require you to assemble the camera, perhaps even source your own [Raspberry Pi Zero 2s](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w), or 3D printed case.
The details of this are still being worked out and up for individual discussion. <br>
You will need mounting hardware for each camera - while we will likely always provide our recommended mounting hardware, a full list of your options can be found in the dedicated [mounting documentation](../setup/mounting_hardware.md), and tips for room setup in the [room setup documentation](../setup/camera_setup.md). <br>
Additionally, you will need a USB PD power source or another power supply that can output between 12V to 24V to power all cameras via the Controller.
Until further testing has been done, we recommend reserving about 5W for each camera.
You also have the option to power each camera individually with a standard 5V power supply (Power Delivery not required) with the cameras USB-C port, intended to be used for wireless operation. <br>
Finally, you'll need a USB 2.0 Data cable for each controller, and a CAT 5e cable with straight wiring for each camera.
These CAT 5e cables are best sourced on your own, they can often be found locally for cheap in a variety of lengths, colors and shapes, depending on your exact setup and requirements.