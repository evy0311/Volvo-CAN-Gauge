# Volvo-CAN-Gauge
Reverse engineering the Volvo VIDA protocol to gather diagnostic information not available via OBD2 on Volvo cars. Modified by evy0311 and started by Alfaa123.

**This is a Fork of Volvo-CAN-Gauge by Alfaa123 on Github. I am modifying it to suit my needs and hardware.**

# Hardware:

- 2006 Volvo S40 2.4i
- SparkFun CANBus Shield (Buy it Here: https://www.sparkfun.com/products/13262)
- SparkFun 5v Serial Enabled 16x2 LCD (Buy it Here: https://www.sparkfun.com/products/9395)
- Arduino UNO (Buy it Here: https://www.sparkfun.com/products/11021)

# Included Libraries:

- MCP_CAN_lib: CAN shield library with modifications to work with the SparkFun shield as well as Alfaa123's setup

# What's Different:

Alfaa123 was designing his code around his C30 T5, which uses a Bosch ECU. THe 2.4i motor uses a Denso ECU. In my testing, the parameters that Alfaa123 was using to get data such as coolant temperature were not the same on my vehicle. This was mostly expected, but for this reason modifications will need to be made.

CANBus hardware is also different, as I am using a completely different setup and need to modify all code to work with my shield and Arduino. 

I am using the default MCP_CAN_lib library instead of the one provided by Alfaa123. However, I have taken some of his modifications and carried them over to my version to allow them to work with my shield.

Also, I believe I have different goals/ideas in mind than Alfaa123 for where I want to take my version of this project, and forking off of his original repo seemed to be the best way to do so. I have been trying to work with the CANBus on this car forever and have had tremendous difficulties, and only recently have I been successful as have others (such as Alfaa123). 

# Basic Functional Description:

The Volvo VIDA protocol is a basic message/response protocol not very different from ISO 15765-4. However, unlike ISO 15765-4, VIDA can also write controller firmware, activate actuators and run diagnostic tests.

In this project, we are only concerned with recieving data that would be otherwise unavailable with ISO 15765-4. Boost pressure, for example, is not available via ISO 15765-4 on Volvo cars. (a partial list of discovered codes from the VIDA database files is in Codes.txt)

We also use data from some broadcasted CAN frames that are used elsewhere in the car (for dashboard brightness, ignition status and headlights). Ideally, we would capture ALL of our information from broadcast frames as that involves a lot less overhead and traffic on the bus. However, some of the information that we need never gets broadcast, so we sometimes have no choice.

The code uses a psudo multi-tasking approach where the message recieve loop is always running if one of the other loops isn't currently running. This allows us to update/check broadcast frames in the background for brightness changes, ignition status changes and button presses and update the global variables accordingly.

In the display loop, we can show boost pressure, coolant temperature and intake temperature. Using the OLEDs massive resolution, we can also graph each of those values for a short amount of time to track dynamic changes.

For right now, the only button we track is the cruise control cancel button. Holding the button for more than 2 seconds changes the currently displayed page. Perhaps in the future, a menu system could be implemented.

# Notes (from Alfaa123):

- Compatibility with other vehicles is unknown at this point. Most likely, any other cars using the Volvo Bosch ME9 implementation will work with no modifications (although I have heard some reports that the broadcast IDs change from vehicle to vehicle, so that may need to be tweaked) **NOTE: As noted above, this was expected and has been seemingly proven after testing with my vehicle**
- Volvo uses extended IDs for their CAN frames. I am not 100% sure why they do this yet.
- Volvos of this vintage have 2 CAN networks, a High Speed bus at 500kbps and a Low Speed bus at 125kbps. The high speed bus is connected to the ECU, steering modules, braking and other modules. The low speed bus is connected to the radio, door modules, instrument cluster and other associated accessories. The CEM (Central Electronics Module) acts as a gateway between the high and low speed busses. This project connects directly to the high speed bus.
- Older vehicles have a diagnostic relay that needs to be activated via K-line in order to access the CAN buses via the OBD2 port. Because mine does not require this, I don't have much information about it.
- The Arduino Mega is not fast enough to recieve all the traffic on the bus. I strongly suspect that we miss a lot of traffic with this implementation, espically because we don't use inturrupt based message handling.

# My Notes:

- The Arduino Uno is also not fast enough to recieve all of the traffic on the bus. Hopefully in the near future we can find a solution for this, although for now it isnt seeming to be a problem.
- All networks in the car connect to the CEM which acts asa gateway. This means that not only do CAN signals go there, but so do LIN signals. Would this make it possible to send CAN messages to manipulate the LIN bus (and potentially control the radio, climate control, etc)? 
-The wiring diagram linked below in "Other Resources" is extremely helpful in determining where the CANBus connects in this car as well as what pins different modules use for it in their harness(es). This will likely be helpful in the future should we choose to spin up modules on a bench and want to manipulate them outside of the car.

# Other Resources:
- http://hackingvolvo.blogspot.com/
- https://github.com/hackingvolvo
- http://opengarages.org/handbook/ebook/
- https://www.theeshadow.com/files/S40MY2005/S40MY2005-tp3978202.pdf
