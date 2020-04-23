#WIFI CONFIG COMMON COMMANDS

* Use Wireless Card
** lsusb, list all usbs
** ifconfig, list all interfaces enabled
** ifconfig [interface] up, enable wireless card
** iwconfig, list all interfaces, check their wireless is supported
** iwlist [interface] scan, list all available wifi networks, (use root, otherwise no result)

* Config Wireless Card
wpa_passphrase SSID PASSWD
* wpa_supplicant -B -i [SSID] -c [CONFIG_PATH], -B means Background,
-i means interface,
-c means config file

** set ip, ifconfig [interface] ip
** set gateway, 
