Modern versions of Windows do not support OSPF and manually adding static routes every time after a reboot is annoying. Below is a batch script you can edit and run to help make adding routes easier. This script assumes that your BGP router and Windows computer are on the same LAN.

```
@echo off
REM fill in YOUR network information
REM right click and RUN AS ADMIN

REM our entire private network address space
set networkv4=172.20.0.0
set networkv4mask=255.252.0.0
set networkv6=fd00::/8

REM our IPv4 subnet info
set subnetv4=172.20.184.240
set subnetv4mask=255.255.255.248
set gateway4=172.20.184.241

REM our IPv6 subnet info
set subnetv6=fd43:6d1:3ee2::/48
set gateway6=fd43:6d1:3ee2:1000::1

REM our address for this machine
set yournetaddr=172.20.184.242
set yournetaddr6=fd43:6d1:3ee2:1000::2/128

REM add IPs
REM if different change wlan0 to YOUR interface name
REM first line here is for my LAN. Ignore it.
netsh interface ipv4 add address "wlan0" 192.168.2.254 255.255.255.0
netsh interface ipv4 add address "wlan0" %yournetaddr% %subnetv4mask%
netsh interface ipv6 add address "wlan0" %yournetaddr6%

REM add IPv4 routes
route -4 add %subnetv4% mask %subnetv4mask% %gateway4%
route -4 add %networkv4% mask %networkv4mask% %gateway4%

REM add IPv6 routes
route -6 add %gateway6% ::
route -6 add %subnetv6% %gateway6%
REM this last route wasn't working without manually filling in the info.
REM I don't know why.. Broken line commented out.
REM route -6 add %networkv6% %gateway6%
route -6 add fd00::/8 fd43:6d1:3ee2:1000::1

echo Press enter to check your IPv4 routing table
echo Do not forget to add static routes to this computer on your BGP router!
echo Example: "root@router:~# ip route add 172.20.184.242 dev wlan0"
echo Example: "root@router:~# ip route add fd43:6d1:3ee2:1000::2/128 dev wlan0"
pause
cls
route -4 print
echo Press enter to check your IPv6 routing table
pause
cls
route -6 print
echo Press enter to try to ping gateway
pause
cls
ping %gateway4%
pause
ping %gateway6%
pause
```