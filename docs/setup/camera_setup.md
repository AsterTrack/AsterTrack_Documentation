# Camera Setup

## Setup Considerations

The positioning of the cameras is critical in determining the quality and volume of tracking. The lenses play an important role as well, but [changing lenses for specific use cases](#custom-lenses) is not expected to be necessary for the majority of users. <br>
Several factors may influence the ideal position for your cameras:

- The desired tracking volume and tracking quality
- The cable runs from the cameras to the controller
- Any room features that facilitate or hinder mounting

The recommended way to mount the cameras is to use the default adhesive ball joint mount, as it provides maximum flexibility and ease of use while not requiring any screws. For a detailed explanation of your options for mounting cameras, [look here](mounting_hardware.md). <br>
It is also recommended to get a good feel for the field of view of the cameras before you fully commit to a camera location - see [visual feedback](#visual-feedback).

#### Tracking Volume
Generally, you'll want to mount cameras high up and in the corners of your tracking space.
If you put them on the edges (e.g. in the middle of a wall right under the ceiling), you'll have to be very considerate about what you want to prioritise and orient the camera accordingly: <br>
Mounting the camera upright results in a wide field of view, mounting it sideways results in a tall field of view - essentially the opposite of the camera body.
The vertical field of view and orientation is a delicate balance between tracking height far into the room and tracking close to the wall the camera is mounted on. <br>
If you have cameras in the corners, they should likely be upright for a wide field of view for greatest coverage, for any additional cameras mounted at the edges, they might benefit from a sideways mount for a tall field of view if you want tracking close to the wall.
But in the end, every tracking space is unique and you may want to experiment to get the most of out of the cameras.

#### Cable Runs
For wired use, the cameras need to connect to the controllers for both power and communication.
The hardware is NOT Ethernet or PoE (Power over Ethernet) compatible, so unless you know for sure you have direct cable runs, do not use any ethernet cables built into your house, do not use PoE hardware, and do not use ethernet switches instead of the AsterTrack controller!
Doing so may damage the hardware or equipment used. <br>
This also means you can not bundle the cable runs together into one cable, and you can not chain the cameras together - every camera needs a direct connection to the controller with a CAT 5e cable. <br>
CAT 5e cables can often be found locally, in the exact length, color, and even shape that your setup needs, for a relatively low price.
This way you can select the cable to blend in with your environment as much as possible if that is a concern.
High quality flat and extra small cables exist for a higher price. <br>
Picking the camera locations carefully can minimise the complicated runs you have to do.
But if a cable run is too prohibitive for whatever reason, you may consider [wireless operation](../operation/wireless.md#wireless-camera-operation) for one or more affected cameras, once that is supported, though you will still have to power the camera from a nearby power outlet.

#### Room Features
Look out for room features that may be used to either facilitate mounting or hide the cable runs:

- Curtain tracks for both guiding the cable runs and, if flat, mounting
- Baseboards or other moldings to hide the cable runs
- Very tall furniture to mount ontop of - though you may loose effective vertical tracking volume if not mounted as high as possible
- Anything molded into your wall or ceiling, like vents, made of a material suited for adhesive mounting

You may want to explore other [mounting options](mounting_hardware.md), including different mounting mechanism and methods of securing, to fit the needs of your tracking space or room.

## Visual Feedback
You may preview a cameras field of view in a potential mounting location by turning on image streaming.
This can be done in the context menu of the camera view, by pressing 'H' while hovering over it, and in the future via the buttons on the camera itself. <br>
To increase usefulness during setup, you may use the contrast and brightness settings in the "Visualisation" panel, Section "Image Adjustment" to make the image more visible, and set the camera view to full-screen by pressing the detach button in the top left of the camera view and sizing it to maximise the area.

## Custom Lenses
The default lenses right now have a high field of view, with significant distortions that need to be compensated for, to improve ease of setup in VR-related use cases over absolute tracking quality. <br>
Professional tracking systems often use narrower field of views with less distortions, but that often pairs with larger tracking spaces requiring more cameras. <br>
The current default lenses have an effective field of view of (89°h, 51°v, 108°d), but due to about 128 horizontal pixels being unused by processing, you will get about (78°h, 51°v, 99°d) tracking field of view. <br>
The lenses can be relatively easily exchanged for custom setups, the default ones are manually fixated with a fixing ring and light glue which can be easily removed. <br>
The main problem will be lens compatibility due to total height and focal length, and achieving a good focus. <br>
After installation, you will need to [calibrate the new lenses](camera_calib.md#custom-lenses) by following the steps and optimising with a lot of samples.

#### Lens Compatibility
The camera has a filter switcher, and it does limit the kinds of lenses you can use. Some complex lenses that compensate for distortion may have a focal length smaller than the height of the filter switcher, and thus can not be focused without removing the filter switcher. If you must use such a lens, you may need to order one with an IR-pass filter around 850nm attached or carefully attach one yourself, and replace the filter switcher assembly with another suitable M12 lens mount. <br>
Additionally, the total lens height once focused needs to roughly line up with the IR LED board height, lest the IR light leaks into the lens and washes out the image, the lens occludes portions of the useable LED light, or the IR LED board occludes part of the image.
Generally, lenses with wildly higher or lower field of view than the default may not line up.
To address this, you may need to reprint the case with a custom height (within limits) - we currently have no case design that allows for adjusting the height, though the rest of the internals have been designed to allow for that in the future. <br>
Lenses with a wider field of view may also have a wider lens head that may exceed the size of the throat of the IR LED Board.

#### Focusing the Lens
For practical reasons (the IR LED board hindering access to the lens) it is recommended to focus the lens without relying on the retroreflection from these IR LEDs. <br>
This can be achieved by using an IR LED, or a visible light LED with the IR-pass filter disabled. You may need to occlude a larger LED using a piece of cardboard with a pinhole puncture to ensure a point-like quality - this will greatly aid in achieving a proper focus. <br>
After installing the lens with a fixing ring like the one provided by default, you may start it up, turn on [image streaming in the interface](#visual-feedback), switch filter if needed, and maximise the view. <br>
After achieving a rough focus, you may zoom in on the point, and find the exact focus - you should be able to see the [pixel instability of the OV9281 sensor](../operation/issues.md#ov9281-pixel-instability) at maximum focus.
You will want to focus past that, so that the pixel instability at the maximum range you want to track becomes insignificant. A slight blur is not a concern, and especially at close range, where markers appear larger, might even be desired for a cleaner image. <br>
Finally, if you are not familiar with a fixing ring, you need to bring the ring to the estimated position first, so that you can tighten the lens against it. Doing so will disturb your intended focus point, so you may need to iteratively find the right place for the fixing ring so that the lens achieves a good focus when tightened against the ring.