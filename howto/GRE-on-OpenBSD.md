# Point-to-Point Layer 3 GRE tunnel interface
This guide describes how to establish an unencrypted and unauthenticated IPv6-over-IPv6 tunnel on [OpenBSD](https://openbsd.org), see [gre(4) EXAMPLES](http://man.openbsd.org/gre.4#Point-to-Point_Layer_3_GRE_tunnel_interfaces_(gre)_example) for similar setups.


# Configuration
Let *A* be the local OpenBSD host and *D* the remote peer, assume public DNS names and IPv6 reachability.

Let `fd42::` and `fd42::1` be the IPs of *A* and *D* respectively where both are allocated as `/127` subnet from one of the peer's DN42 prefix.

## pseudo interface
Populate [`/etc/hostname.gre0`](https://man.openbsd.org/hostname.if.5) with:
```
tunnel A.example.com D.example.net
inet6 fd42::/127
```
This will resolve FQDNs at parse time, set *A*'s and *D*'s IPs as source and destination tunnel address and set *A*'s assigned IP as point-to-point address on the interface.

Replace hostnames in the `tunnel` line with literal IPs if DNS is not available (at system boot).

Reboot or run [`sh /etc/netstart gre0`](https://man.openbsd.org/netstart.8) to bring up the tunnel.

## miscellaneous
Populate `/etc/sysctl.conf` with:
```
net.inet.gre.allow=1
```
Reboot or run `sysctl net.inet.gre.allow=1` to allow GRE packet processing.

-
At this point, `gre0` will be administratively *UP*:
```
$ ifconfig gre0
gre0: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1476
        index 22 priority 0 llprio 6
        encap: vnetid none txprio payload rxprio packet
        groups: gre
        tunnel: inet6 2001:db8::a --> 2001:db9::d ttl 64 nodf ecn
        inet6 fe80::221:28ff:fef9:c1d8%gre0 -->  prefixlen 64 scopeid 0x16
        inet6 fd42:: -->  prefixlen 127
```

All traffic destined to `fd42::1/127` will be encapsulated and routed to *D*:
```
$ route show
[...]
Internet6:
Destination                        Gateway                        Flags   Refs      Use   Mtu  Prio Iface
fd42::/127                         fd42::                         UCn        1        0     -     4 gre0
fd42::                             fd42::                         UHl        0        0     -     1 gre0
fd42::1                            link#0                         UHc        0     3180     -     3 gre0
fe80::%gre0/64                     fe80::221:28ff:fef9:c1d8%gre0  Un         0        0     -     4 gre0
fe80::221:28ff:fef9:c1d8%gre0      fe80::221:28ff:fef9:c1d8%gre0  UHl        0        0     -     1 gre0
ff01::%gre0/32                     fe80::221:28ff:fef9:c1d8%gre0  Um         0        1     -     4 gre0
ff02::%gre0/32                     fe80::221:28ff:fef9:c1d8%gre0  Um         0        1     -     4 gre0
[...]
```
```
$ route -n get fd42::1
   route to: fd42::1
destination: fd42::1
       mask: ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff
  interface: gre0
 if address: fd42::
   priority: 3 ()
      flags: <UP,HOST,DONE,CLONED>
     use       mtu    expire
    3181         0         0 
```

# Security
GRE may be protected with IPsec to encrypt and authenticate traffic, [OpenIKED](http://www.openiked.org/) can be used to establish an IKEv2 session between *A* and *D*.
