To quote from <https://frrouting.org/>:

"FRRouting (FRR) is a free and open source Internet routing protocol suite for Linux and Unix platforms. It implements BGP, OSPF, RIP, IS-IS, PIM, LDP, BFD, Babel, PBR, OpenFabric and VRRP, with alpha support for EIGRP and NHRP."

It features a similar configuration style to Cisco IOS.

### Installation 
Install the `frr` and `frr-pythontools` package on your favourite Linux/BSD distribution. For BGP RPKI support, also install `frr-rpki`. _Make sure you are using frr version 8.5 or greater for IPv6 link local peerings._

- More installation options: <https://docs.frrouting.org/en/latest/installation.html>
- Releases: <https://frrouting.org/release/>

## Configuration

Important cofiguration files:
- `/etc/frr/daemons`: daemons that will be started 
- `/etc/frr/vtysh.conf`: configuration for the VTY shell
- `/etc/frr/frr.conf`: configuration for the daemons
- `/etc/frr/${DAEMON}.conf`: configuration for a single daemon (deprecated)

It this guide, only BGP will be set up using the shared `/etc/frr/frr.conf`.

### Daemons

First, setup `/etc/frr/daemons`. As stated previously. this file specifies which daemons will be started.

```diff
--- /etc/frr/daemons
+++ /etc/frr/daemons
@@ -14,7 +14,7 @@
 #
 # The watchfrr, zebra and staticd daemons are always started.
 #
-bgpd=no
+bgpd=yes
 ospfd=no
 ospf6d=no
 ripd=no
```

### VTY shell

To use the VTY shell, `/etc/frr/vtysh.conf` needs to be set up. _The `hostname` and `banner motd` also need to be entered there manually to be persistant._

```
service integrated-vtysh-config
```

Unprivileged users need to be in the `frrvty` group to use `vtysh`.

The VTY shell can be used to interact with running daemons and configure them. Changes made in the VTY shell can be written to `/etc/frr/frr.conf` using the `write` command. To enter configuration mode use the `configure` command. To get information about the available commands, press `?`.

### Zebra 

Before configuring BGP, a few other things need to be set up. First, create a [prefix-list](https://docs.frrouting.org/en/latest/filter.html#ip-prefix-list) for the dn42 prefixes. That will be used to filter out non-dn42 routes to be announced to BGP. For that, open `/etc/frr/frr.conf` or `vtysh` in configuration mode and add:

```
ip prefix-list dn42 seq 1 deny 172.22.166.0/24 le 32
ip prefix-list dn42 seq 1001 permit 172.20.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1002 permit 172.21.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1003 permit 172.22.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1004 permit 172.23.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1100 permit 172.20.0.0/14 ge 21 le 29
ip prefix-list dn42 seq 2001 permit 10.100.0.0/14 le 32
ip prefix-list dn42 seq 2002 permit 10.127.0.0/16 le 32
ip prefix-list dn42 seq 2003 permit 10.0.0.0/8 ge 15 le 24
ip prefix-list dn42 seq 3001 permit 172.31.0.0/16 le 32
ip prefix-list dn42 seq 9999 deny 0.0.0.0/0 le 32
!
ipv6 prefix-list dn42v6 seq 1001 permit fd00::/8 ge 44 le 64
ipv6 prefix-list dn42v6 seq 9999 deny ::/0 le 128
```

This prefix list can be created yourself by following the instructions for Quagga in the `data/filter.txt` and `data/filter6.txt` files from the registry. 

Next create a [route-map](https://docs.frrouting.org/en/latest/routemap.html), which will be used for doing the actual filtering later.

```
route-map dn42 permit 5
 match ip address prefix-list dn42
 set src <IPv4 address of the node>
exit
!
route-map dn42v6 permit 5
 match ipv6 address prefix-list dn42v6
 set src <IPv6 address of the node>
exit
```

### BGP

With the configuration of the daemons file and Zebra done, BGP can now be configured.

```
router bgp <AS of the network>
 neighbor <IPv4 peer address> remote-as <Peer AS>
 neighbor <IPv6 peer address> remote-as <Peer AS>
 ! In case an IPv6 link local address is used to peer
 neighbor <IPv6 peer address> interface <Peer interface>
 !
 address-family ipv4 unicast
  neighbor <IPv4 peer address> activate
  neighbor <IPv4 peer address> route-map dn42 in
  neighbor <IPv4 peer address> route-map dn42 out
 exit
 !
 address-family ipv6 unicast
  neighbor <IPv6 peer address> activate
  neighbor <IPv6 peer address> route-map dn42v6 in
  neighbor <IPv6 peer address> route-map dn42v6 out
 exit
exit
```

With everything configured, the BGP session should come up. In the normal VTY shell mode the status of BGP peerings can be checked using the `show bgp summary` command.

### Complete configuration example

```
router bgp <Your AS here>
 neighbor <Peer IPv4> remote-as <Peer AS>
 neighbor <Peer IPv6> remote-as <Peer AS>
 ! In case an IPv6 link local address is used to peer
 neighbor <Peer IPv6> interface <Peer interface>
 !
 address-family ipv4 unicast
  neighbor <IPv4 peer address> activate
  neighbor <IPv4 peer address> route-map dn42 in
  neighbor <IPv4 peer address> route-map dn42 out
 exit
 !
 address-family ipv6 unicast
  neighbor <IPv6 peer address> activate
  neighbor <IPv6 peer address> route-map dn42v6 in
  neighbor <IPv6 peer address> route-map dn42v6 out
 exit
exit
!
ip prefix-list dn42 seq 1 deny 172.22.166.0/24 le 32
ip prefix-list dn42 seq 1001 permit 172.20.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1002 permit 172.21.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1003 permit 172.22.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1004 permit 172.23.0.0/24 ge 28 le 32
ip prefix-list dn42 seq 1100 permit 172.20.0.0/14 ge 21 le 29
ip prefix-list dn42 seq 2001 permit 10.100.0.0/14 le 32
ip prefix-list dn42 seq 2002 permit 10.127.0.0/16 le 32
ip prefix-list dn42 seq 2003 permit 10.0.0.0/8 ge 15 le 24
ip prefix-list dn42 seq 3001 permit 172.31.0.0/16 le 32
ip prefix-list dn42 seq 9999 deny 0.0.0.0/0 le 32
!
ipv6 prefix-list dn42v6 seq 1001 permit fd00::/8 ge 44 le 64
ipv6 prefix-list dn42v6 seq 9999 deny ::/0 le 128
!
route-map dn42 permit 5
 match ip address prefix-list dn42
 set src <IPv4 address of the node>
exit
!
route-map dn42v6 permit 5
 match ipv6 address prefix-list dn42v6
 set src <IPv6 address of the node>
exit
```

## Further reading

### General things

- FRR documentation: <https://docs.frrouting.org/en/latest>
- FRR source code: <https://github.com/frrouting/frr>

### Configuration tipps 

- Use [peer groups](https://docs.frrouting.org/en/latest/bgp.html#peer-groups) (_Strongly reccomended to limit the work neede to add new peers or change general configuration for may peers._)
- `tab` and `?` are your best friends in the VTY shell
- Use `find REGEX` in the VTY shell to find certain commands
