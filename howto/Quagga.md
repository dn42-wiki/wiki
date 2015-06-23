# Quagga

Quagga is probably one of the oldest software router around.  It still works, of course, even though it has an unattractive configuration syntax (unfortunately inspired by Cisco's IOS) and has some small issues with IPv6.  But since it's so old, you will find a lot of configuration examples around.

## Source address selection

Use this in your `zebra.conf`:

    route-map RM_SET_SRC permit 10
      set src 172.22.XX.XX
    ip protocol bgp route-map RM_SET_SRC

Unfortunately, this is not possible with IPv6...

## Important bgp commands
To connect to bgpd use:

    $ vtysh

Which provides an interactive interface.
In this interface '?' can be used to list the available commands or subcommands.

## Configure Quagga
a minimal config would look like this:

    vtysh> configure terminal
    vtysh(config)> router bgp <your-asn>
    vtysh(config-router)> neighbor <neighbor-ip> remote-as <neighbor-asn>
    vtysh(config-router)> neighbor <neighbor-ip> interface <interface>
    vtysh(config-router)> exit
    vtysh(config)> exit

### IPv6
for IPv6 do something like

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
    
### peer groups, prefix lists and such
If you want to use 'prefix-list' to filter some of the prefixes quagga is receiving, you can use a 'peer-group' instead of apply the prefix list to every neighbor. 

Define a peer group:

    vtysh(config-router)> neighbor <peer-group-name> peer-group

Apply to a neighbor:

    vtysh(config-router)> neighbor <neighbor-ip> peer-group <name>

Apply a prefix list for incoming prefixes to your peer group:

    vtysh(config-router)> neighbor <peer-group-name> prefix-list <prefix-list-name> in


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