This is a description on how to use Nginx Proxy Manager add-on with Home Assistant.
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
 - Nginx Proxy Manager add-on (**NPM**) is installed
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
- In addition to your Duck DNS domain, addresses like *alpha.myduckname.duckdns.org*, *beta.myduckname.duckdns.org* and *mqtt.myduckname.duckdns* will be forwarded to your router. 

## Set up NPM proxy hosts 
 
### Host 1 - HTTPS access to HA front-end

Let *https://**ha**.myduckname.duckdns.org* be the address to use to access HA front-end.
 
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

The *Forward port* makes the trick: this is an illegal port that makes all access attempts result in an error: `502 Bad Gateway`.

### Host 2 - HTTPS access to password protected pages

Let *https://**pw**.myduckname.duckdns.org**/protected/**... * be the addresses to use to access password protected pages.

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

### Host 3 - HTTPS access to open pages

Let *https://**open**.myduckname.duckdns.org**/open/**... * be the addresses to use to access unprotected pages.

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



### Host 4 - Secure Websocket access to MQTT broker 

An address like wss://mqtt.myduckname.duckdns.org:443 is used.

Note that the standard port 8884 is not used. To use port 8884, the router has to forward it to port 443.  
 
 - Details
   -  Domain Names: mqtt.myduckname.duckdns.org
   -  Scheme: http 
   -  Forward IP: 192.168.0.111
   -  Forward Port: 1884 (*Websocket port*)
   -  Websockets Support: on
   -  Access List: Publically Avaiable (*but access control is handled by MQTT broker*)
 
 - SSL
   - SSL Certificate: request a new SSL certificate, will result in mqtt.myduckname.duckdns.org
   - Force SSL: on

This set-up will work for Qdash if MQTT password is entered by the user or provided in the URL. 

In such case the application is accessed like https://ha.myduckname.duckdns.org/local/qdash/apps/myapp.html 

If you want to explicitly include MQTT credentials in the code, also access to the application must be restricted. To achieve this and still have unrestricted access to custom web pages under /config/www like https://ha.myduckname.duckdns.org/local/somepage.html, you have to add another proxy host and use some tricks:  

### Host 3 - for HTTPS access to Qdash application

 using https://qd.myduckname.duckdns.org/qdash/apps/myapp.html

- Details
   - Domain Names: qd.myduckname.duckdns.org
   - Scheme: http 
   - Forward IP: 192.168.0.111
   - Forward Port: 8124 (*trick - an illegal port, access attempts result in code 502*)
   - Websockets Support: on
   - Access List: myuserlist (*you have to add a NPM user/password list for this*)
 - Custom locations
   - (*forward addresses like qd.myduckname.duckdns.org/qdash/apps/myapp.html*)
   - Define location: /qdash/apps (*location for your apps*)
   - Scheme: http 
   - Forward IP: 192.168.0.111/local/qdash/apps
   - Forward Port: 8123  
- SSL
   - SSL Certificate: (*request a new SSL certificate, will result in qd.myduckname.duckdns.org*)
   - Force SSL: on

### Host 1 modification - block all access to ha.myduckname.duckdns.org/local/qdash/apps
- Custom locations: 
  - Define location: /local/qdash/apps *location for your apps*
  - Scheme: http 
  - Forward IP: 192.168.0.111
  - Forward Port: 8124  (*trick - an illegal port, access attempts result in code 404*)

The recommendation is to complete all steps.

If you want to have unrestricted access to pages under config/www/qdash you can change Access list no Publicly Accessible.

## Configure MQTT

Access to MQTT should be restricted. This is done by setting up a set of username/password pairs. Under *Login*, for example:

    - username: "ha"
      password: "e34fG239"
    - username: "qdash01"
      password: "zx47567hq"

You can also restrict allowed topics for each username/password pair:
Enable this option under *Customize*

    active: true
    folder: mosquitto

Create a file */share/mosquitto/acl.conf* with content: 

    acl_file /share/mosquitto/accesscontrollist
    sys_interval 10


Create a file */share/mosquitto/accesscontrollist* with content like:

    # HA - access to any topic
    user ha
    topic readwrite #

    # Qdash - access to Qdash topics
    user qdash01
    topic readwrite fromweb/#
    topic readwrite toweb/#

Under *Network*, configure your MQTT broker to listen for *MQTT over Websockets* on the standard port *1884*. 

## Access restrictions 
There are two possible types of access restrictions:
- for page access, where username and password entered by the user give access to the web pages. The page access control is implemented in NPM.
- for MQTT access, where username and password give the right to send and/or receive MQTT messages of specific topics. The MQTT access control is implemented in the MQTT broker.

A Qdash application is basically a static page that is hosed by the ordinary HA web server. The page itself contains and refers to HTML code, Javascript code and CSS code. 

In order to connect to the MQTT broker, the MQTT username and password have to be known to the Javascript code. 

In Qdash there is support for providing username and/or password in three ways:

1. they are entered by the user when starting the application
2. they are provided as URL arguments
3. they are defined in the page code

A function provides the functionality:

`mqtt.getUsernamePassword( username, password )` 

- username - a username or an empty string
- password - a password or an empty string

The function returns an object with members username and password, or empty strings if not provided.

- If argument *username* is provided, it will be returned. 
- Else, if URL argument *username* is provided, it will be returned.
- Else, the user will be prompted for the username, and it will be returned.

- If argument *password* is provided, it will be returned. 
- Else, if URL argument *password* is provided, it will be returned.
- Else, the user will be prompted for the username, and it will be returned.

Examples:

    var s;
    // result is in s.username and s.password
    s = mqtt.getUsernamePassword( "qdash01", "zx47567hq" );
    s = mqtt.getUsernamePassword( "qdash01", "" );
    s = mqtt.getUsernamePassword( "", "" );

## Multiple applications with different access restrictions

It is possible to have applications where only certain users have access. 
However, to be secure, the different users should have dedicated automations and MQTT topics.

An example:
User A - can control bedroom lights.
User B - can control all lights.

One automation (with defind MQTT topics) handles all lights. 
A's application only controls bedroom lights.
B's application controls all lights.
However, suppose A is evil. If A knows the entity ids of other lights, it is possible to control them using a MQTT tool. This is because A's and B's applications uses the same MQTT topics.

To avoid this, they should have separated automations with different MQTT topics.

    
