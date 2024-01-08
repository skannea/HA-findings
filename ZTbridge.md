## How to access remote ESPHome devices using ZeroTier VPN
Main source is https://zerotier.atlassian.net/wiki/spaces/SD/pages/224395274/Route+between+ZeroTier+and+Physical+Networks

I am running a HA (HA OS) at home. I have a number of ESPHome devices there. My local network has SSID *republik* and has addresses 192.168.0.x. 

I also have a need for a few ESPHome devices located in my summer house where I have a mobile router. 
It has SSID *quartet* and handles addresses 192.168.8.x. Static addresses have x below 100.

My goal was to be able to manage and access all my ESPHome devices from my HA.

When connected to *republik* with my PC: 
- Got myself a ZeroTier network with addresses 192.168.192.x and a network id *a09acf02331f7899*
- Installed ZeroTier One on my PC which got address 192.168.192.14
- Installed ZeroTier One add-on in my HA which got address 192.168.192.77
- Tested to log on from PC to HA using 192.168.192.77:8123 
See the add-on documentation for how to do this.
The address to my *ZeroTier network page* is https://my.zerotier.com/network/a09acf02331f7899

Note that you can log on to HA using 192.168.192.77:8123 from any local network where there is an internet connection. 

When connected to *quartet*:
- I put an 16 GB SD card into the PC
- downloaded and ran *Raspberry Pi Imager*
- selected RASPBERRY PI ZERO,  RASPBERRY PI OS (LEGACY, 32-BIT) LITE
- selected my SD card and clicked NEXT
- edited:
  - Host name: *landgate*.local
  - User name: *anders* and a password
  - SSID: *quartet* and password
  - On the SERVICES tab I enabled SSH with password authentification
- made the SD card

- I started Landgate:
  - put the SD card into my RPi Zero W
  - powered on

- I found out the address assigned to Landgate:
  - connected my PC to *quartet*
  - logged into the router
  - found landgate in the client list
  - noted the ip address (192.168.8.131)
    
- I logged in to Landgate:
  - started puTTY on my PC
  - selected SSH, port 22 and 192.168.8.131
  - opened the session and answered yes I trust this connection
  - got a login prompt
  - entered *anders* and password

- I installed ZeroTier on *landgate*:
  - ran command `curl -s https://install.zerotier.com | sudo bash`
  - waited a couple of minutes
  - noted the ZeroTier address *3aea0e1e4a*.
  - connected to my ZeroTier network using `sudo zerotier-cli join a09acf02331f7899`
  - on my ZeroTier network page I found the new ZeroTier address *3aea0e1e4a* and checked the box to authenticate it
  - here I also got the address for *landgate*: 192.168.192.31
  - ran command `sudo zerotier-cli listnetworks` to get device name *ztuku5a6vk*: `...OK PRIVATE ztuku5a6vk 192.168.192.31/24`

- I configured *landgate* to work as a ZeroTier bridge
  - uncommented line `net.ipv4.ip_forward=1` using the *nano* editor and command `sudo nano /etc/sysctl.conf` (exit with ctrl-X)
  - ran command `sudo sysctl -w net.ipv4.ip_forward=1` 
  - introduced two variables with command `PHY_IFACE=wlan0; ZT_IFACE=ztuku5a6vk`
  - ran command `sudo iptables -t nat -A POSTROUTING -o $PHY_IFACE -j MASQUERADE`
  - ran command `sudo iptables -A FORWARD -i $PHY_IFACE -o $ZT_IFACE -m state --state RELATED,ESTABLISHED -j ACCEPT`
  - ran command `sudo iptables -A FORWARD -i $ZT_IFACE -o $PHY_IFACE -j ACCEPT`
  - ran command `sudo apt install iptables-persistent` (takes a couple of minutes)
  - ran command `sudo bash -c iptables-save > x` (wrote to local file x to avoid access problems) 
  - ran command `sudo mv x /etc/iptables/rules.v4`

- I connected my PC back to to *republik* 
- on my ZeroTier network page, under Advanced, configured a *managed route* 192.168.8.0/23 via 192.168.192.31 

Now all connections to ip addresses 192.168.8.x on my local network at home will be forwarded to 192.168.8.x on the local network at my summer house.
 
- I set up an ESPHome device at my summer house:
  - in HA, edited code for an ESPHome device connecting to *quartet* and with static address 192.168.8.15 
  - installed on the ESPHome hardware via a plugged in cable to my PC. 
  - moved this in range of *quartet* and powered on
  - added a new ESPHome device in HA settings by specifying the address 192.168.8.15  

