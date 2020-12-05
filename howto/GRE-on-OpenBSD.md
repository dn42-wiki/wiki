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


# Security
GRE may be protected with IPsec to encrypt and authenticate traffic, [OpenIKED](http://www.openiked.org/) can be used to establish an IKEv2 session between *A* and *B*.