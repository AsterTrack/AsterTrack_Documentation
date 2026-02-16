# Hardware

Currently, the only way to get a hold of the hardware is through the first batch of dev kits that is currently being planned. <br>
If you are interested, please contact [devkits@astertrack.dev](mailto:devkits@astertrack.dev).

## Development History and DIY
There are good reasons that despite always being intended to be open source and open hardware, we have not distributed hardware design files and asked people to build it before.
The hardware is and always was a significant expenditure, and as long as the software to use it was not ready, there was only ever the promise of future potential. <br>
We felt it was not appropriate to try to get developers or early adopters onboard with this promise alone when the timeline for getting the project into a genuinely useable state envisioned by us was so uncertain.
While releasing and publicising it earlier as a DIY project might have sped up development, knowing that an imperfect multi-camera system held nothing but frustration for its users, and that this fact is hard to properly convey in the same sentence as the potential it held, we decided to get it into a useable state first. <br>
Years have passed, more than was ever intended, and there is still a lot of potential for improvements in both hardware and software even now.
But at least we can be sure that early adopters and developers who decide to spend money on the first batches do so without solely relying on promises of future potential - the base multi-camera experience is solid, with only the feature set required determining its current usability.

## Going forward with DIY
While the core functionality of the hardware is deceptively easy to recreate (see [first prototype hardware](https://www.seneral.dev/astertrack.html)), the current [block diagram of the system](../details/architecture.md) hints at the complexities that result from making the system actually stable and useful for real world use.
As such, the custom PCBs are considered a necessity, with their improvements to system stability, complex power management, and many usability features, even when cutting some additional features like wired/wireless use or the filter switcher. <br>
That means, our approach to DIY will always require ordering the PCBs, either from us or from industry-standard PCBA partners.
Sadly, this is in practice quite unattainable at a reasonable price for individuals, with just the PCBA order costing as much as a small camera system. <br>
Additionally, we are currently using a custom camera module which includes a filter switcher. While it may be possible for us to sell those individually in the future, more likely we would want to redesign the PCB to instead use off-the-shelf modules like those from [innomaker](https://www.inno-maker.com/product/cam-mipi9281raw-v2/), and a different case to go along with that larger form factor.
We are aiming to work on this more DIY-compatible version of the PCBs in the future after the current design is finalised, to allow for iteration by the community without huge capital expenses. <br>
And there is potential for iteration that we are excited to see: Various different camera sensors like the SC132GS, purpose-built versions, perhaps even other processing solutions like FPGAs - there is a lot of room to improve still.
For anyone looking to DIY such a system, please take a look at the following section to find out how we arrived at certain critical design decisions.

## Driving Design Decisions
AsterTrack has been designed around the custom hardware from the very beginning and it is essential for a performant and user-friendly multi-camera experience.
Some of those defining decisions will be explained here at a high level.
You can find some more technical details in the [hardware architecture page](../details/architecture.md#hardware).

##### On-Board Processing
One of the core decisions is that the frames need to be processed on-board the AsterTrack Cameras for a scalable, low-latency system that is not constrained by data-throughput and host system capabilities.
Otherwise, you end up with a system that needs to use tricks to achieve a useable framerate on most consumer systems, might require purchasing of additional hardware to ensure available data throughput, has a harsh upper limit for the number of cameras supported, and takes away processing resources from accompanying software like VR applications. <br>
This is why the decision was made to build a [custom high-efficiency frame processing pipeline](../details/architecture.md#qpu-processing-camera) on the Raspberry Pis GPU - it is one of the few cheap SBCs that allow for such low-level access to its hardware to be able to optimise for one use case.
That means we are currently locked into requiring a Raspberry Pi with the Video Core IV - unless the whole processing pipeline is replaced.
This only makes sense when considering the use of FPGAs.

##### Wired Communication
Another important design decision was to focus on wired cameras, despite their obvious usability concerns, as consumer wireless systems are too unreliable for such low-latency and medium-bandwidth requirements of sending blob and image data to a single system from many cameras at once.
Instead, we are using the Raspberry Pis built-in UART and use a custom controller to pass the data to the host system via USB, which the CH32V307 chip allowed us to do. The alternative, using Ethernet and PoE switches, does have some advantages, but would have increased costs and significantly lowered the usability for end users. <br>
Wireless operation is planned, but it will never be reliable for all users, and can not be relied upon as the sole method of communication. and is instead reserved as an option. Due to delays in software development, we were already able to accommodate for this in our first dev kits.

## Required Hardware
You need at least 3 AsterTrack Cameras and one AsterTrack Controller for a minimal setup. You can add more cameras and even controllers at any point, with each controller supporting up to 8 cameras. <br>
Future features like IMU may require wireless dongles and IMU hardware itself, and future wireless operation may require a similar wireless dongle as a sync beacon. <br>
Dev Kits may require you to assemble the camera, perhaps even source your own [Raspberry Pi Zero 2s](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w), or 3D printed case.
The details of this are still being worked out and up for individual discussion. <br>
You will need mounting hardware for each camera - while we will likely always provide our recommended mounting hardware, a full list of your options can be found in the dedicated [mounting documentation](../setup/mounting_hardware.md), and tips for room setup in the [room setup documentation](../setup/camera_setup.md). <br>
Additionally, you will need a USB PD power source or another power supply that can output between 12V to 24V to power all cameras via the Controller.
10W should be sufficient for a 3 camera setup, each camera needs about 3W, but please calculate with 5W to be safe.
You also have the option to power each camera individually with a standard 5V power supply (Power Delivery not required) with the cameras USB-C port, intended to be used for wireless operation. <br>
Finally, you'll need a USB Data and Power cable for each controller, and a CAT 5e cable with straight wiring for each camera.
These CAT 5e cables are best sourced on your own, they can often be found locally for cheap in a variety of lengths, colors and shapes, depending on your exact setup and requirements.