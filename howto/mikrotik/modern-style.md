This page is a scratchpad for a fully modernised (2026) config running on the latest version of RouterOS (7.22.1). It will use wireguard, BGP with Multihop and Extended Nexthop, and link-local addressing for address space efficiency.


Wireguard tunnel setup
====

Wireguard seems to be the most popular option for peering now, so we'll use that. The first step is to setup a Wireguard interface. You can use a single interface for all connections if you want, but I prefer to make one for each peer, as it can make problems easier to debug.

For the sake of this example we'll pretend we're peering with Kioubit, one of the most-connected and easiest to connect-with systems on DN42.

Create the interface
----

We have to do this first so we can gather the connection details, which will be used to setup the peering.

The standard port for Wireguard listeners is 51820, but you can use anything you like. An MTU of 1420 is recommended to avoid fragmentation and firewall issues.

```
/interface wireguard
add name=DN42-KIOUBIT listen-port=51820 mtu=1420
```

Without any extra parameters, it'll automatically generate a new private and public key for you. Grab the public key, you'll need that to setup your peering connection.
```
/interface/wireguard/print where name="DN42-KIOUBIT"

[furinkan@helian] > /interface/wireguard/print where name="DN42-KIOUBIT"       
 0  R name="DN42-KIOUBIT" mtu=1420 listen-port=51820 public-key="mGRGoKPl6fRffzUovFp/8AoOtHjCrWeEMN9/9NsVp2Q="
```

Exchange connection parameters
----

If connecting to a system with a self-serve automatic peering interface, now is the time to use it.

You will need:
* Your AS number
* Your public IP address that they can connect to
* Your listening port for Wireguard, eg. 51820 as above
* Your public key for your Wireguard interface
* An IPv4 address for your local end of the tunnel, this is often an address in your DN42 address allocation, but can also be any private-range address like 192.168.x.x; however this may be optional if you use...
* An IPv6 address for your local end of the tunnel. Unlike IPv4, it's possible to use a pure link-local address, like `fe80::1234`
* To know if your BGP daemon supports the Multihop and Extended Next Hop capabilities

You will receive:
* Their AS number
* Their public IP address and port to use for the wireguard connection
* Their Wireguard public key
* The IPv4 and/or IPv6 address for the remote end of the tunnel

Add the Wireguard peer
----

```
/interface wireguard peers
add name=DN42-KIOUBIT interface=<THE_INTERFACE_YOU_CREATED_EARLIER> endpoint-address=<THEIR_PUBLIC_ADDRESS> endpoint-port=<THEIR_WIREGUARD_PORT> public-key="<THEIR_WIREGUARD_PUBLIC_KEY>" allowed-address=fd00::/8,fe80::/10,172.20.0.0/14
```

At this point the tunnel should come up on its own, assuming the other end is already configured. However, it's not useful yet without an IP address attached to it.

Add an IP address to your end of the tunnel
----

Once the tunnel is established you will need an address at each end of the link for routing to work. In this Kioubit example we only need an IPv6 address, because we can route IPv4 traffic between IPv6 BGP peers.

Kioubit is using fe80::ade0 on the remote end of the tunnel. We also chose a link-local address for our end of the tunnel, it's `fe80::` plus the last four digits of our AS number. You'll notice that the remote address does not need to be adjacent at all - this is the beauty of IPv6 link-local addressing.
```
/ipv6 address
add address=fe80::2762/128 interface=DN42-KIOUBIT advertise=no
```

If you're using an IPv4 address, you can attach it to the loopback interface. This allows the address to be reused for multiple peering connections, without being "owned" by any of the wireguard interfaces.
```
/ip/address
add address=172.31.254.254 interface=lo
```

Test connectivity
----

You should be able to ping the remote end of the tunnel now. If so, packets can make it there and back, correctly routed. Notice that we're appending the interface name to the IP address. This is necessary for IPv6 link-local addresses because the system doesn't know which interface to send the pings out of.
```
[furinkan@helian] > ping fe80::ade0%DN42-KIOUBIT
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 fe80::ade0                                 56  64 133ms383us echo reply
    1 fe80::ade0                                 56  64 136ms229us echo reply
    sent=2 received=2 packet-loss=0% min-rtt=133ms383us avg-rtt=134ms806us max-rtt=136ms229us
```

If you're using IPv4 inside the tunnel, it'll look something like this:
```
[furinkan@helian] > ping 172.20.53.105%DN42-KIOUBIT
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 172.20.53.105                              56  64 133ms544us
    1 172.20.53.105                              56  64 131ms672us
    2 172.20.53.105                              56  64 133ms521us
    sent=3 received=3 packet-loss=0% min-rtt=131ms672us avg-rtt=132ms912us max-rtt=133ms544us
```

Notice that we specified the egress interface again, you don't normally do this with IPv4 addresses. The router doesn't know how to get to the destination because it doesn't belong to a known subnet - it doesn't have a route!

We'll add a static route to fix this:
```
/ip route
add dst-address=172.20.53.105/32 gateway=DN42-KIOUBIT

[furinkan@helian] > ping 172.20.53.105
  SEQ HOST                                     SIZE TTL TIME       STATUS
    0 172.20.53.105                              56  64 131ms152us
    1 172.20.53.105                              56  64 131ms128us
    sent=2 received=2 packet-loss=0% min-rtt=131ms128us avg-rtt=131ms140us max-rtt=131ms152us
```

That's better. Now we're ready for some BGP peering.


Setup BGP
====

We'll create an instance for DN42, some basic route filters for security, a template to hold common DN42 peering settings, then finally the actual peering connection.

Instance
----

An important part of the routing protocols is your router's ID, a 32-bit value usually written like an IPv4 address. It's common convention to use your router's IP address as the router ID, so we'll do that here too. I've picked the highest IP address in our IPv4 allocation as the router's IP, which we'll also use as the router's ID.

This gives a text label to our router ID:
```
/routing id
add id=172.22.124.62 name=my-DN42-router select-dynamic-id=""
```

Create the BGP instance, which will identify itself with your AS number:
```
/routing bgp instance
add as=424242<YOUR_ASN> name=DN42 router-id=my-DN42-router
```

Route filters
----

Filters are necessary to prevent other people from hijacking our routing table. A malicious peer could send routes that override your default route to public internet services like Google, government services, your online banking, etc.

These rules are reasonably tight, you can tighten or relax them as desired. [Interconnected networks'](/Interconnections) IPv4 ranges are included as well, if you don't want then you can ignore that rule.
```
/routing filter rule
add chain=dn42-in comment="reject prefixes clashing with home network" disabled=no rule="if (dst in 192.168.0.0/16 && dst-len >= 16) { reject }"
add chain=dn42-in comment="reject v4 auto-addressing prefixes" rule="if (dst in 169.254.0.0/16 && dst-len >= 16) { reject }"
add chain=dn42-in comment="accept DN42 v4 prefixes" disabled=no rule="if (dst in 172.20.0.0/14) { accept }"
add chain=dn42-in comment="accept DN42 interconnected network v4 prefixes" disabled=no rule="if (dst in 10.0.0.0/8) { accept }"
add chain=dn42-in comment="accept DN42 and interconnected v6 prefixes" disabled=no rule="if (dst in fd00::/8) { accept }"
```

```
/routing filter rule
add chain=dn42-out comment="don't advertise our home network" rule="if (dst in 192.168.0.0/16 && dst-len >= 16) { reject }"
add chain=dn42-out comment="don't advertise v4 auto-addressing prefixes" rule="if (dst in 169.254.0.0/16 && dst-len >= 16) { reject }"
add chain=dn42-out comment="tag my prefixes with community" disabled=no rule="if ( dst in 172.20.0.0/14 && bgp-communities-empty ) { append bgp-communities DN42-communities; }"
add chain=dn42-out comment="advertise DN42 v4 prefixes" disabled=no rule="if (dst in 172.20.0.0/14) { accept }"
add chain=dn42-out comment="advertise DN42 interconnected network v4 prefixes" disabled=no rule="if (dst in 10.0.0.0/8) { accept }"
add chain=dn42-out comment="tag my IPv6 prefixes with community" disabled=no rule="if ( dst in fd00::/8 && bgp-communities-empty ) { append bgp-communities DN42-communities; }"
add chain=dn42-out comment="advertise DN42 and interconnected v6 prefixes" disabled=no rule="if (dst in fd00::/8) { accept }"
```

Connection template
----

WIP

Create a template with the common BGP settings used for DN42:
```
/routing bgp template
set DN42-thighhighs afi=ip,ipv6 as=424242<YOUR_ASN> input.filter=dn42-in multihop=yes name=DN42-thighhighs output.filter-chain=dn42-out .redistribute=connected,bgp routing-table=main
```
