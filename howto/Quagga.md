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
In this interface the following commands can be used:

The following text use this placeholders:

- `<AS>` your Autonomous System Number (only the digits)
- `<GATEWAY_IP>` your gateway ip (the internal dn42 ip address you use on the host, where dn42 is running)
- `<SUBNET>` your registered dn42 subnet, which you allocated on [nixnodes](https://io.nixnodes.net/)
- `<PEER_IP>` dn42 ip of your peer who is connected with you using your favorite vpn/tunnel protocol (openvpn, ipsec, tinc, ...)
- `<INTERFACE>` Interface which is used to connect to the peer, in case of openvpn it is the tun device
- `<PEER_AS>` Autonomous System Number of your peer (only the digits)

## Configure a new ipv6 peering

In your interactive vtysh session type the following:

```
vtysh> configure terminal
vtysh> router bgp <AS>
vtysh> neighbor <PEER_IP> remote-as <PEER_AS>
vtysh> neighbor <PEER_IP> peer-group dn
vtysh> neighbor <PEER_IP> interface <INTERFACE>
vtysh> no neighbor <PEER_IP> activate
vtysh> exit
vtysh> address-family ipv6
vtysh> neighbor <PEER_IP> activate
vtysh> neighbor <PEER_IP> soft-reconfiguration inbound
vtysh> exit
```

## Configure a new ipv4 peering

```
vtysh> configure terminal
vtysh> router bgp <AS>
vtysh> neighbor <PEER_IP> remote-as <PEER_AS>
vtysh> neighbor <PEER_IP> peer-group dn
vtysh> neighbor <PEER_IP> interface <INTERFACE>
vtysh> exit
```

# show bpg session status

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