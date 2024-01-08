
I am running a HA (HA OS) at home. I have a number of ESPHome devices there. My local network has addresses 192.168.0.x. 

I also have a need for a few ESPHome devices located in my summer house where I have a mobile router.
This is my solution in short:

Connected to my home router: 
Got myself a ZeroTier network with addresses 192.168.192.x.
Installed ZeroTier One add-on in my HA. 

Connected to my summer house router:
Configured the router to have addresses 192.168.8.x with x<100 for static addresses. 
Installed OS on a RPi Zero W called Landgate.
Configured Landgate to connect to the router.
Installed ZeroTier on Landgate.
Added Landgate to my ZeroTier network and got the address 192.168.192.31.
Configured Landgate to work as a ZeroTier bridge.
Configured a 'managed route' 192.168.8.x via 192.168.192.31 in my ZeroTier network.

Connected to my home router: 
Edited code for an ESPHome device connecting to my summer house router and with static address 192.168.8.15. 
Installed on the ESPHome hardware and via a plugged in cable. 
Moved this in range of the summer house router.
Added a new ESPHome device in HA settings by specifying the address 192.168.8.15.  

