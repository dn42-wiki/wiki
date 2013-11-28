This is the config I'm running on an Ubiquiti EdgeRouter Lite (AS76197). It features:

* dn42 DNS
* "classic" OpenVPN P2P (including the common "comp-lzo" option)
* BGP
* Some traffic-shaping rules for my very slow 3mbit DSL uplink
* 2 internal: One DN42 network (172.22.117.128/25 for me and my servers as well as a NAT 192.168.42.10/24 for my parents, so that they're save from dn42 - that network is NOT announced to dn42).
* Firewall to protect my NAS server and monitoring

