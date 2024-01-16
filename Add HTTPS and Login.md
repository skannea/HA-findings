# Home Assistant and Nginx Proxy Manager 

This is a description on how to use Nginx Proxy Manager (**NPM**) add-on with Home Assistant (**HA**).
It describes how to:
- access HA front-end from the internet
- restict access to static HTML files in */config/www/* folder
- allow access to static HTML files in */config/www/* folder
- set up secure communication from the internet to the MQTT broker

## Recommended installation and set-up

A standard HA installation is used:
 - HA OS is installed
 - HA's ip address is known (in this example 192.168.0.111)
 - HA web server uses standard port 8123 
 - web pages are put under *config/www/*
 - Duck DNS add-on is installed
 - Nginx Proxy Manager add-on  is installed
 - Mosquitto add-on (MQTT broker) is installed
 
## Set up your router to forward ports 443 and 80 to NPM

NPM add-on is a server with the same ip as HA but it listens on port 443 (HTTPS) and port 80.

Router port forwarding functionality is often found under *virtual servers*.
- External port 443 shall be routed to 192.168.0.111:443 
- External port  80 shall be routed to 192.168.0.111:80 

## Get a DuckDNS domain

- At *duckdns.org* you can register a Duck DNS domain ( in this example: *myduckname.duckdns.org* ).  
- Configure Duck DNS add-on with your domain. Now the add-on will tell duckdns.org the internet ip of your router. 
- All traffic to your Duck DNS domain will be forwarded to your router. 
- If the internet ip changes, the add-on will update duckdns.org.
- In addition to your Duck DNS domain, addresses like *alpha.myduckname.duckdns.org*, *beta.myduckname.duckdns.org* and *mqtt.myduckname.duckdns.org* will be forwarded to your router. 

## Set up NPM proxy hosts 

In this example, HA is assumed to be accessed on the local network with ip `192.168.0.111:8123`.

The following folder structure is used:
- /www         (web pages placed here are accessed with `http://192.168.0.111:8123/local`)
  - /protected (for pages that require username/password)
  - /open      (for pages that  
  - /common    (for javascript and css files that may be used by all pages)
         
### Host 1 - HTTPS access to HA front-end

Let `https://ha.myduckname.duckdns.org` be the address to use to access HA front-end.
 
 - Details
   -  Domain Names: ha.myduckname.duckdns.org
   -  Scheme: http 
   -  Forward IP: 192.168.0.111
   -  Forward Port: 8123
   -  Websockets Support: on
   -  Access List: Publically Avaiable (*but access control is handled by HA*)
 - SSL
    - SSL Certificate: (*request a new SSL certificate, will result in ha.myduckname.duckdns.org*)
    - Force SSL: on

With this configuration also web pages located at /config/www (with addresses like https://ha.myduckname.duckdns.org/local/mypage.html) may be accessed. 
However, we want to determine what pages are public and what pages need a login. 
First, block all access to *ha.myduckname.duckdns.org/local* by adding the following to Host 1: 
- Custom locations
  - Define location: /local
  - Scheme: http 
  - Forward IP: 192.168.0.111
  - Forward Port: 8124  

The *Forward port* makes the trick: this is an unused port that makes all access attempts result in an error: `502 Bad Gateway`.

### Host 2 - HTTPS access to password protected pages

Let `https://pw.myduckname.duckdns.org/protected/... ` be the addresses to use to access password protected pages.

Make an Access list:
- Details
  - Name: JustMe
- Authorization
  - User name: anders
  - Password: thepassword

Make a new proxy server:
- Details
   -  Domain Names: pw.myduckname.duckdns.org
   -  Scheme: http 
   -  Forward IP: 192.168.0.111
   -  Forward Port: 8124 (*the trick*)
   -  Websockets Support: on
   -  Access List: JustMe
- Custom locations
  - Define location: /protected
  - Scheme: http 
  - Forward IP: 192.168.0.111/local/protected
  - Forward Port: 8123   
- SSL
    - SSL Certificate: (*request a new SSL certificate, will result in pw.myduckname.duckdns.org*)
    - Force SSL: on

### Host 3 - HTTPS access to unprotected pages

Let `https://open.myduckname.duckdns.org/open/...` be the addresses to use to access unprotected pages.

Make a new proxy server:
- Details
   -  Domain Names: open.myduckname.duckdns.org
   -  Scheme: http 
   -  Forward IP: 192.168.0.111
   -  Forward Port: 8124 (*the trick*)
   -  Websockets Support: on
   -  Access List: Public Accessible
- Custom locations
  - Define location: /open
  - Scheme: http 
  - Forward IP: 192.168.0.111/local/open
  - Forward Port: 8123   
- SSL
    - SSL Certificate: (*request a new SSL certificate, will result in open.myduckname.duckdns.org*)
    - Force SSL: on

### Additional protected hosts  

You may add another host for protected pages that is 
- based on another access list 
- with address `https://xxx.myduckname.duckdns.org/yyy/... ` 

You create it the same way as for Host 2:
- Make a new access list with user/password pairs.
- Assign the access list to a new proxy server with domain name xxx.myduckname.duckdns.org
- Define a location `/yyy` with forward ip to `192.168.0.111/local/yyy`   
- Make  a new SSL certificate

### Common resources for public and protected hosts  

If you want to set up javascript and css files to be used by protected as well as unprotected pages, the method described so far has to be complemented.

In host 2 as well as host 3, define another location:
- Custom locations
  - Define location: /common
  - Scheme: http 
  - Forward IP: 192.168.0.111/local/common
  - Forward Port: 8123   

In your HTML file, refer files like this:

      <head>
        ...
        <link rel="icon" href="/common/nicefavicon.ico" type="image/x-icon" />
        <link rel="stylesheet" href="/common/prettycolors.css" />
        <script src="/common/utilities.js" type="text/javascript"></script>


### Host 4 - Secure Websocket access to MQTT broker 

Let the address `wss://mqtt.myduckname.duckdns.org:443` be the address to your MQTT broker.

Note that the standard port 8884 is not used. Port 443 is used because it is already forwarded to Nginx. 
If you want to use port 8884, add this forwarding in the router.
 
 - Details
   -  Domain Names: mqtt.myduckname.duckdns.org
   -  Scheme: http 
   -  Forward IP: 192.168.0.111
   -  Forward Port: 1884 (*Standard websocket port*)
   -  Websockets Support: on
   -  Access List: Publically Avaiable (*but access control is handled by MQTT broker*)
 
 - SSL
   - SSL Certificate: request a new SSL certificate, will result in mqtt.myduckname.duckdns.org
   - Force SSL: on

