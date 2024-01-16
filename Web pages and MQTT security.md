# Web pages and MQTT security

## MQTT client access rights
A web page may have code for a MQTT client. 
When a MQTT client connects to a MQTT broker it uses a username/password pair. 
In the broker configuration, the username is assigned to a set of MQTT topics.
For each topic the access rights tell if the client may publish and/or subscribe to the topic.

## Web page access rights
A web page may be protected or unprotected. Access to a protected web page requires a user login. 
The user has to enter a username/password pair.
The username/password pair may be remembered by the user's browser. 

## A dilemma
The code of an unprotected web page must never contain the MQTT password. This can either be provided in the URL or by prompting the user. The URL solution is practical but might be considered as a risk.
The prompt solution is secure but the user has to enter the password every time the page is loaded. To avoid this, the page may store and retrieve the password in the browser. However, this storage is not secured in any way.

The code of a protected web page on the other hand, is only available to trusted users. The MQTT password is in the code and may be revealed by the trusted user. This might be considered as a risk. 

So, there is a trade off between user friendlyness and security aspects.

## Other solutions
The so called *Basic Authentication* that is used by Nginx is intended for web servers, but it also supports Web sockets. That is what MQTT communication uses. 

Dedicated web servers like Apache may provide more secure solutions. However, in that case a solution with MQTT is not the first choice.

The HA add-on for ZeroTier VPN give very simple and secure solutions. In such cases it is possible to skip Nginx and there is no need for port forwarding in the router.  


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

    

