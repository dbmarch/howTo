# Setting up the PI:

Instructions to setup the Raspberry PI to host a MQTT broker and setup a wireless AP with DHCP server and NAT.  NAT is only needed if you want to be able to plug your Raspberry PI into a LAN and route traffic between the 2 networks.

Load Raspian
 Update it:
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get install build-essential 

Configure raspi-config to enable the I2C, camera, ssh, etc
    sudo raspi-config

check your interfaces:
  ifconfig

  lsusb



## 1) Install Dexter Libraries: (IF YOU ARE USING A GROVE PI SHIELD)

These are only necessary if the applications you are using are based upon the Dexter Industries sensors.
With these libraries and the shield, it is very simple to access all types of sensors using Python.

Install Grove PI Libraries from Dexter: (PYTHON)

Setting up the PI:

    sudo curl -kL dexterindustries.com/update_grovepi | bash

    sudo reboot


## 2) Install MQTT Client Python libraries:

```
git clone https://github.com/eclipse/paho.mqtt.python
cd paho.mqtt.python
sudo python setup.py install```

Useful links:

a) https://pypi.python.org/pypi/paho-mqtt

b) https://www.eclipse.org/paho/downloads.php

## Now add some python libraries
    sudo pip install netifaces

    sudo apt-get install python-mosquitto


## 3) MOSQUITTO BROKER & CLIENTS

to view the packages available to you:
    sudo apt-cache search mosquitto

to install broker and clients:
  
    sudo apt-get install mosquitto mosquitto-clients

    sudo mosquitto -v -c /etc/mosquitto/mosquitto.conf


## 7) Install Mosquitto Broker

[Setting up MQTT on PI](https://learn.adafruit.com/diy-esp8266-home-security-with-lua-and-mqtt/configuring-mqtt-on-the-raspberry-pi)

----
To startup the mosquitto server:

In foreground in separate window:

    sudo mosquitto -v -c /etc/mosquitto/mosquitto.conf

As daemon:

    sudo mosquitto -v -d -c /etc/mosquitto/mosquitto.conf

to kill background mosquitto :


    sudo kill $(ps aux |awk '/mosquitto/ {print $2}')


## 8) Setting up a WiFi AP

Tutorial:

[Adafruit Wifi AP Setup](https://cdn-learn.adafruit.com/downloads/pdf/setting-up-a-raspberry-pi-as-a-wifi-access-point.pdf)

```
sudo apt-get install hostapd isc-dhcp-server
sudo apt-get install iptables-persistent
```

setup dhcp:

```
sudo nano /etc/dhcp/dhcpd.conf
```

comment out the 2 lines with the option domain name and domain name server.

>uncomment "authoritative"

add the subnet configuration at the bottom.


```
subnet 192.168.10.0 netmask 255.255.255.0 {
   range 192.168.10.10 192.168.10.50;
   option broadcast-address 192.168.10.255;
   option routers 192.168.10.1;
   default-lease-time 600;
   max-lease-time 7200;
   option domain-name "local";
   option domain-name-servers 8.8.8.8,8.8.4.4;
   }

sudo nano /etc/default/isc-dhcp-server
```

and scroll down to 

>INTERFACES="" 

and update it to say 

>INTERFACES="wlan0"

make sure the interface is down:

```
sudo ifdown wlan0
```

create a file in /etc/hostapd 
call it hostapd.conf:

```
interface=wlan0
driver=nl80211
ssid=PI_AP
wpa_passphrase=raspberry
country_code=US
hw_mode=g
channel=5
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
wpa_group_rekey=86400
ieee80211n=1
wme_enabled=1
```


at the bottom of the network interfaces file: 
( in /etc/network/interfaces)


>auto eth0
>iface eth0 inet dhcp
>
>allow-hotplug wlan0
>iface wlan0 inet static
>     address 192.168.10.1
>     netmask 255.255.255.0

in /etc/defaults/hostapd

>DAEMON_CONF="/etc/hostapd/hostapd.conf"


in /etc/init.d/hostapd

>DAEMON_CONF=/etc/hostapd/hostapd.conf


### Configure NAT
Run 

``` 
sudo nano /etc/sysctl.conf
```


Scroll to the bottom and add

>net.ipv4.ip_forward=1


```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

reboot and try it out ( the tutorial on adafruit is thorough and it is easy to mess something up.)

---


## 9) Useful blog on using Paho mosquitto.

[Paho Blog](http://www.steves-internet-guide.com/client-objects-python-mqtt/)

## 10) Download FileZilla to upload files to PI:
https://wiki.filezilla-project.org/Client_Installation


## 11) Install python tools if you don't have pip
----
    sudo apt-get install python-pip python-dev 

## 12) Markdown

[viewer](http://markdownlivepreview.com/)
[cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
