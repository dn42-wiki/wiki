# Quagga

Quagga is probably one of the oldest software router around.  It still works, of course, even though it has an unattractive configuration syntax (unfortunately inspired by  [Cisco's IOS](https://dn42.net/howto/IPsecWithPublicKeys/CiscoIOSExample)) and has some small issues with IPv6.  But since it's so old, you will find a lot of configuration examples around.

## Source address selection

Use this in your `zebra.conf`:

```conf
route-map RM_SET_SRC permit 10
    set src 172.22.XX.XX
ip protocol bgp route-map RM_SET_SRC
```

Unfortunately, this is not possible with IPv6...

## Important bgp commands
To connect to bgpd use:

```sh
$ vtysh
```

Which provides an interactive interface.
In this interface '?' can be used to list the available commands or subcommands.

## Configure Quagga
a minimal config would look like this:

```sh
vtysh> configure terminal
vtysh(config)> router bgp <your-asn>
vtysh(config-router)> neighbor <neighbor-ip> remote-as <neighbor-asn>
vtysh(config-router)> neighbor <neighbor-ip> interface <interface>
vtysh(config-router)> exit
vtysh(config)> exit
```

### IPv6
for IPv6 do something like

```sh
vtysh> configure terminal
vtysh(config)> router bgp <your-asn>
vtysh(config-router)> neighbor <neighbor-ip> remote-as <neighbor-asn>
vtysh(config-router)> neighbor <neighbor-ip> interface <interface>
vtysh(config-router)> no neighbor <neighbor-ip> activate
vtysh(config-router)> address-family ipv6
vtysh(config-router-af)> neighbor <neighbor-ip> activate
vtysh(config-router-af)> exit
vtysh(config-router)> exit
vtysh(config)> exit
```

### peer groups, prefix lists and such
If you want to use 'prefix-list' to filter some of the prefixes quagga is receiving, you can use a 'peer-group' instead of apply the prefix list to every neighbor. 

Define a peer group:

```sh
vtysh(config-router)> neighbor <peer-group-name> peer-group
```

Apply to a neighbor:

```sh
vtysh(config-router)> neighbor <neighbor-ip> peer-group <name>
```

Apply a prefix list for incoming prefixes to your peer group:

```sh
vtysh(config-router)> neighbor <peer-group-name> prefix-list <prefix-list-name> in
```

#### Example filter list

```sh
ip prefix-list vpn-in description BGP IPv4 import filter
!old network:
ip prefix-list vpn-in seq 5 permit 172.22.0.0/15 ge 22 le 28
!new dn42 allocation:
ip prefix-list vpn-in seq 10 permit 172.20.0.0/16 ge 22 le 28

! Anycast /32s for Whois and DNS:
ip prefix-list vpn-in seq 11 permit 172.22.0.43/32
ip prefix-list vpn-in seq 12 permit 172.22.0.53/32

ip prefix-list vpn-in seq 18 permit 192.175.48.0/24
ip prefix-list vpn-in seq 20 deny 10.10.10.0/24
ip prefix-list vpn-in seq 21 permit 10.0.0.0/8
ip prefix-list vpn-in seq 30 permit 172.31.0.0/16
ip prefix-list vpn-in seq 39 permit 100.64.0.0/10
ip prefix-list vpn-in seq 40 permit 195.160.168.0/23
ip prefix-list vpn-in seq 41 permit 91.204.4.0/22
ip prefix-list vpn-in seq 43 permit 193.43.220.0/23
ip prefix-list vpn-in seq 46 permit 83.133.178.0/23
ip prefix-list vpn-in seq 47 permit 87.106.29.254/32
ip prefix-list vpn-in seq 50 permit 85.25.246.16/28
ip prefix-list vpn-in seq 51 permit 46.4.248.192/27
ip prefix-list vpn-in seq 60 permit 94.45.224.0/19
ip prefix-list vpn-in seq 70 permit 195.191.196.0/23
ip prefix-list vpn-in seq 80 permit 80.244.241.224/27
ip prefix-list vpn-in seq 90 permit 46.19.90.48/28
ip prefix-list vpn-in seq 91 permit 46.19.90.96/28
ip prefix-list vpn-in seq 110 permit 188.40.34.241/32
ip prefix-list vpn-in seq 130 permit 37.1.89.192/26
ip prefix-list vpn-in seq 140 permit 178.33.32.123/32
ip prefix-list vpn-in seq 150 permit 87.98.246.19/32
ip prefix-list vpn-in seq 1000 deny 0.0.0.0/0

ipv6 prefix-list vpn-in seq 10 permit fd00::/8 ge 9
ipv6 prefix-list vpn-in seq 15 deny any
```

#### Example filter list script
```sh
#!/bin/bash

vtysh -c 'conf t' -c "no ip prefix-list dn42"; #drop old prefix list

while read pl
do
   vtysh -c 'conf t' -c "$pl"; #insert prefix list row by row
done < <(curl -s https://ca.dn42.us/reg/filter.txt | grep -e  ^[0-9] | awk '{ print "ip prefix-list dn42 seq " $1 " " $2 " " $3 " ge " $4 " le " $5}' | sed "s_/\([0-9]\+\) ge \1_/\1_g;s_/\([0-9]\+\) le \1_/\1_g");
vtysh -c "wr" #write new prefix list

```

## show bpg session status

in this example:
* an active bgp session exists with peer 64713.
* no (vpn) connection at all exists with peer 64692
* a (vpn) connection with 4242421375 exists, but no bgp session

```
vtysh> show ip bgp summary 
BGP router identifier 172.22.100.254, local AS number 64698
RIB entries 938, using 103 KiB of memory
Peers 11, using 49 KiB of memory
Peer groups 1, using 32 bytes of memory

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.22.92.247   4 64692       0       0        0    0    0 never    Connect
...
172.22.113.2    4 64713    2206     865        0    0    0 01:23:11      322
....
172.23.64.1     4 4242421375  0       0        0    0    0 never    Active
fe80::deca:fbad 4 64699     902     694        0    0    0 01:23:57      486
```
