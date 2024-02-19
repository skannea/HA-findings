# HA-findings
My own solutions for Home Assistant, ESPHome, etc.

[How to connect remote ESPHome devices to HA using ZeroTier](ZTbridge.md) 

I am running a HA (HA OS) at home. I have a number of ESPHome devices there. 
I also have a need for a few ESPHome devices located in my summer house.
My goal is to be able to manage and access all my ESPHome devices from my HA.


[Web pages and MQTT security](Web%20pages%20and%20MQTT%20security.md)

under construction ...

[Home Assistant and Nginx Proxy Manager](Add%20HTTPS%20and%20Login.md)

This is a description on how to use Nginx Proxy Manager (NPM) add-on with Home Assistant (HA). It describes how to:
- access HA front-end from the internet
- restict access to static HTML files in /config/www/ folder
- allow access to static HTML files in /config/www/ folder
- set up secure communication from the internet to the MQTT broker

How to make your own HA add-on 

Also for us running HA OS there is a simple way to add servers (and other programs) in the HA environment: Home Assistant Add-ons. 

There is a [tutorial](https://developers.home-assistant.io/docs/add-ons/tutorial/) for that.
My own example is the **Creds server**. It is 
- an ordinary static page web server 
- used primarly for web applications communicating with MQTT 
- intended to be used with Nginx
- a server that can provide MQTT credentials
- a server that can provide the auth credentials that were used at login

