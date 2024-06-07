# CHMT Pick-and-place machine configs.

This repo is for documenting configs for CHMT pick and place machines.

The original author has a CHMT48VB machine, however these configs can serve as the basis for the CHMT36VA/VB and CHMT48VA as well. The VB variants have touch screen and integrated camera switchers, the VA machines have a camera switcher board.

## Modifications

* Touch screen computer removed.
* Up and down cameras removed.
* Fitted a 720p 25FPS USB "Mini Bullet Camera" for the down camera, color, 26mm diameter body. With replacable lens. ~8-12mm lens (get both, see which you prefer).
* Fitted a 720p 60FPS USB "Global shutter camera", black and white output. Standard board camera, adjustable zoom/focus lens.
* Drag chanins replaced with semi-closed (i.e. openable) ones.
* Pump and blower motors mounted on foam for some sound and vibration isolation.
* Down-light ring PCB designed, to be connected to 24V+OT2 pins when assembled. Controllable by SmoothieWare.
* Smoothieware-CHMT installed on the MCU on the main control board.  See https://github.com/hydra/Smoothieware-CHMT/releases 
 * See 'config.default' in this repo for the smoothieware-chmt config.
* OpenPnP 2 (test version)
* RS422 to UART interface board using Silicon CP2105, connected to the RS232 and RS422 interfaces, exposes 2 COM ports.
* Minisforums UM773 Lite Mini PC (AMD 7735HS, 8 core, 16 thread, 32GB DDR5, 512GB SSD). See https://store.minisforum.com/products/minisforum-um773-lite
 * Windows 11
 * One camera on a USB3.2 port. (Root hub 1)
 * Other camera on a USB2.0 port. (Root hub 2)
 * 2nd USB 2 port connected to a passive USB Hub, Keyboard + Mouse + STLink V3 Set.
 * HDMI monitor.


## Setup

* Using hole-punch to make 5mm OpenPnP test objects.
* Primary and secondary homing fiducials permenantly mounted in front of the discard tray.
  * Secondary one printed on card, then stuck to a 2mm + 1mm double sided black foam tape.  Drag pin clears it.
* Park position rear center, to allow for feeder access on both sides and the front of the machine and easy nozzle change/inspection.

### Drag feeders

Drag feeders are a fiddly, OpenPnP doesn't have proper support for the ones on the CHMT machine built-in, and requires much coordination and setup, the general principle of operation is as follows:

The idea is the dragpin goes to the hole location, extends, then the head uses the drag pin to pull the tape just a bit more than 4mm, then moves the head back a bit so the dragpin can release easily. while the drag pin is moving from the start location to the mid location, it rotates the peeler by 4.00, then while the dragpin is moving from the mid location to the end location it rotates the peeler by another 4.00. the movement and rotation happens in parallel.  the dragpin then releases at the end.
After a feed operation, if you position the camera over the drag pin location it should be exactly in the center of the hole.

* add dragpin actuator, with axis interlock.
* ensure machine co-ordination 'Before actuation' and 'After actuation' are enabled.
* add dragpin gcode driver commands for boolean operations.
* determine dragpin offset, use the nozzle offset wizard, but actuate the dragpin into blue-tack/glay instead of operating the nozzle, once the offset is calculated apply it to the drag pin actuator.
* add 2 axis, C and D, for the left/right cover tape peelers, call them LPEEL and RPEEL.
* ensure gcode driver has config for acceleration/move/read/reset operations for the C/D axis (there's some machine.xml files out there that don't)
* add 2 more actuators, LFEED and RFEED.
  * Set them to be 'profile' actuators.
  * on the profiles tab, add default boolean on and off profiles so that the default 'on' profile turns the drag pin on, and default 'off' turns the drag pin off.
  * Set the axis to X=X, Y=Y, Z=cam counter clockwise, Rotation=LPEEL/RPEEL.
  * No axis interlock.
  * Set the X/Y/Z offset to the same offset as the dragpin.
* add a ReferencePushPullFeeder
  * ensure the camera can see two holes above the pick position. one hole to the left of the metal feeder cover plate, and one hole to the right of the metal cover feeder plate, i.e. the hole that the dragpin will use.
  * set the pick position.
  * disable 'check on job start'. the OCR stuff cannot be easily used as there's no-where to attach part labels near the pick position.
    * Note: it should be possible to make a 3D printed part which attaches to the cover tape bar, which has pockets for feeder labels, at which point the OCR region could be defined is being to the left of the pick position, but this will only work for the left hand feeder back as the camera cannot be moved to the right of the right hand feeder bank.
  * on the push-pull motion, uncheck all the tickboxes for movement (up/down/arrows) except for the last 2 PUSH (down-arrows) for the Mid 3 location and the End location.
  * set the start location to the dragpin hole.
  * set mid 3 location to 4mm + 0.1mm to the right of the dragpin hole.
  * set end location to 4mm - 0.2mm to the right of the dragpin hole.
  * set the feed actuator to LFEED/RFEED, do not set an auxiliary actuator.
  * set the rotation to 4.00 for the mid 3 location and end location. ensure additive rotation is enabled.

Note: The above assumes 4mm component and hole pitch. Adjust peeler rotation and end positions appropriately when using different pitches. 

## Issues and solutions

### Drag pin missing drag hole

Solution 1: incorrectly configured drag pin offset.
 * verify the offset is correct by using the blu-tack/putty method.
 * update the x/y offset on the dragpin and LFEED/RFEED actuators.

### Drag pin missing drag hole only when feeding

The drag pin goes in the tape hole when the machine is stationary, but not during a feed operation.

Solution 1: it was found that when using the ReferenceAdvanceMotionPlanner without 'After actuation' was needed on the dragpin actuator.
  * without it enabled the machine would have already started moving the head to drag before the drag pin had even actuated.
  * adding delay G4 delay commands, e.g. `G4 P200` to the DRAGPIN GCode drivers `ACTUATE_BOOLEAN_COMMAND` made no difference (!)

### Drag pin hole not in the right position for the next feed operation

Hole is to the right of the drag pin location. 

Solution 1: peeler grub screws too tight and the cover tape is pulling the tape after the pin has been removed.
* loosen the grub screw.

Solution 2: adjust the end position left slightly.
