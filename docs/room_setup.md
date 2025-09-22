# Initial Room Setup
The most important thing to consider is where to put the cameras. <br>
Several factors may influence this decision:

- The volume you want to be trackable
- The cable run from the cameras to the controller
- Potential room features that facilitate mounting

For a detailed explanation of the options for mounting hardware, [look here](mounting.md). <br>
It is also recommended to get a good feel for the field of view of the cameras before you fully commit to a camera setup - see [visual feedback](#visual-feedback).

#### Tracking Volume
Generally, you'll want to mount cameras high in the corners of the room.
If you put them in the middle of a wall (also high up at the ceiling), you'll have to be very considerate about what you want to prioritise and orient the camera accordingly:
Mounting the camera upright results in a wide field of view, mounting it sideways results in a tall field of view.
The vertical field of view and orientation is a delicate balance between tracking height far into the room and tracking close to the wall the camera is mounted on.

#### Cable Runs
The cameras need to connect to the controllers for both power and a custom communication protocol.
These are NOT Ethernet or PoE (Power over Ethernet) compatible, so unless you know for sure you have direct cable runs, do not use any ethernet runs built into your house, do not use PoE hardware, and do not use ethernet switches instead of the controller!
Doing so may harm the hardware or equipment used. <br>
You need to buy the cables yourself, which can often be found locally, in the length, color, and even shape that your exact setup needs.
This way you can blend the cable to fit in as much as possible. <br>
The custom protocol used sadly prohibits merging multiple cables into one, so you may need to route multiple cables alongside each other.
Picking the camera locations carefully can minimise the complicated runs you have to do.

#### Room Features
Look out for room features that may be used to either facilitate mounting or hide the cable runs:

- Curtain tracks for both mounting (if flat) and guiding the cable runs
- Baseboards or other moldings to hide the cable runs
- Very Tall furniture to mount ontop of (though you may loose effective vertical tracking volume if not mounted as high as possible)

See [Mounting](mounting.md) for which mounting hardware you can use, and also which methods of securing you can use for the specific room you have.

### Visual Feedback
You may preview a cameras field of view in a potential mounting location by turning on image streaming.
This can be done in the context menu of the camera view, by pressing 'H' while hovering over it, and in the future via the buttons on the camera itself. <br>
To increase usefulness during setup, you may use the contrast and brightness settings in the Visualisation View, Section *Image Adjustment* to make the image more visible, and set the camera view to full-screen by pressing the *Detach* button in the top left of the camera view.