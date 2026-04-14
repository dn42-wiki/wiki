# How to connect to dn42 using Mikrotik RouterOS


## Legend

 * 1.1.1.1 - peer external IP
 * 2.2.2.2 - your external IP
 * A private /30 range for the GRE endpoints: 192.168.200.128/30
 * 192.168.200.129 - remote GRE IPv4 address
 * 192.168.200.130 - local GRE IPv4 address
 * fd42:c644:5222:3222::40 - remote GRE IPv6 address
 * fd42:c644:5222:3222::41 - local GRE IPv6 address
 * YOUR_AS - your AS number (numbers only)
 * PEER_AS - peer AS number (numbers only)

## RouterOS limitations

 * IPSec only supports IKEv1
 * OpenVPN only works in tcp mode
 * OpenVPN does not support LZO compression
 * You can't use /31 subnet for Point-to-Point (PtP) links

Mikrotik/RouterOS doesn't handle /32 addresses very well on Point-to-Point links (like GRE). There is a [separate howto](/howto/mikrotik/ptp32) to explain how to setup /32 addresses on a GRE tunnel (or even an OpenVPN tunnel). What is the easy way? Just use any /30 subnet on the GRE Link, either from your assigned DN42 pool of addresses, or use a private address like 192.168.x.x. Please don't choose from 172.16.0.0/12 or 10.0.0.0/8 because they may overlap with DN42 or ChaosVPN.

RouterOS v7.2 has some nasty bugs when using PTP configuration or IPv6 link local addresses as NEXTHOP. It won't work (confirmed for v7.2 by their support staff).

## Tunnel

### IPSec
First, let's add IPSec peer and encryption policy.
Your peer most likely provided you with encryption details. If not, ask them about it.
Here we're gonna use aes256-sha256-modp1536

```
/ip ipsec peer
add address=1.1.1.1 comment=gre-dn42-peer dh-group=modp1536 \
enc-algorithm=aes-256 hash-algorithm=sha256 local-address=2.2.2.2 secret=PASSWORD
```

```
/ip ipsec policy
add comment=gre-dn42-peer dst-address=1.1.1.1/32 proposal=dn42 protocol=gre \
sa-dst-address=1.1.1.1 sa-src-address=2.2.2.2 src-address=2.2.2.2/32
```

### GRE
Pretty straightforward here:
```
/interface gre
add allow-fast-path=no comment="DN42 somepeer" local-address=2.2.2.2 name=gre-dn42-peer \
remote-address=1.1.1.1
```

### IPs inside the GRE tunnel
Your peer most likely provided you with IP adresses for the GRE tunnel.
As mentioned before, you can't use /31 for PtP links, so for the "easy way" we will be using a /30.
If you want to avoid wasting a whole /30 for your peering, please check the [point-to-point configuration for RouterOS](/howto/mikrotik/ptp32)

Add the IP your peer provided you:

#### IPv4

```
/ip address
add address=192.168.200.130/30 interface=gre-dn42-peer network=192.168.200.128
```

#### IPv6

Here we can use /127, so it's simple:
```
/ipv6 address
add address=fdc8:c633:5319:3300::41/127 advertise=no interface=gre-dn42-peer
```

If you configured everything correctly you should be able to ping the remote end of the tunnel. In this specific example that would be 192.168.200.129 (IPv4) or fdc8:c633:5319:3300::40 (IPv6).

## BGP

### Filters

Both BGP and routing filters were redone from the ground up on RoS 7.x
The official migration guide can be found [here](https://help.mikrotik.com/docs/display/ROS/Routing)

It's a good idea to setup filters for BGP instances, both IN (accept advertisements) and OUT (send advertisements)
In this example, we will be filtering:

* IN: 192.168.0.0/16 and 169.254.0.0/16, because we don't want other people's routes interfering with out network
* OUT: 192.168.0.0/16 and 169.254.0.0/16, because you shouldn't be advertising your private (non-DN42) network

This filter will not only catch /8 or /16 networks, but smaller networks inside this subnets as well.

#### RoS 6.x

```
/routing filter
add action=discard address-family=ip chain=dn42-in prefix=192.168.0.0/16 prefix-length=16-32 protocol=bgp
add action=discard address-family=ip chain=dn42-in prefix=169.254.0.0/16 prefix-length=16-32 protocol=bgp
add action=discard address-family=ip chain=dn42-out prefix=192.168.0.0/16 prefix-length=16-32 protocol=bgp
add action=discard address-family=ip chain=dn42-out prefix=169.254.0.0/16 prefix-length=16-32 protocol=bgp
```

If you want only DN42 connectivity, you can filter IN 10.0.0.0/8 (ChaosVPN / freifunk networks):
```
/routing filter
add action=discard address-family=ip chain=dn42-in prefix=10.0.0.0/8 prefix-length=8-32 protocol=bgp
```

#### RoS 7.x

RoS 7 now defaults to default-reject, meaning if you reach the end of the chain without matching any rules, the route will be rejected.

As such, you need to either explicitly accept all the prefixes that you want to keep, or place a final accept at the end of the chain, after rejecting undesired prefixes.

In this example, we will use the second method.
```
/routing filter rule
add chain=dn42-in rule="if (dst in 192.168.0.0/16 && dst-len > 16) { reject }"
add chain=dn42-in rule="if (dst in 169.254.0.0/1 && dst-len > 16) { reject }"
add chain=dn42-in rule="accept"
add chain=dn42-out rule="if (dst in 192.168.0.0/16 && dst-len > 16) { reject }"
add chain=dn42-out rule="if (dst in 169.254.0.0/1 && dst-len > 16) { reject }"
add chain=dn42-out rule="accept"
```

If you want only DN42 connectivity, you can also filter IN 10.0.0.0/8 (ChaosVPN / freifunk networks):

```
/routing filter
add chain=dn42-in rule="if (dst in 10.0.0.0 && dst-len > 8) { reject }"
```

### BGP
Now, for actual BGP configuration.

#### RoS v6
```
/routing bgp instance
set default disabled=yes
add as=YOUR_AS client-to-client-reflection=no name=bgp-dn42-somename out-filter=dn42-in router-id=1.1.1.1
```
Let's add some peers. Right now we have just one, but we still need two connections - to IPv4 and IPv6

IPv4:
```
/routing bgp peer
add comment="DN42: somepeer IPv4" in-filter=dn42-in instance=bgp-dn42-somename multihop=yes \
name=dn42-somepeer-ipv4 out-filter=dn42-out remote-address=192.168.200.129 remote-as=PEER_AS \
route-reflect=yes ttl=default
```
IPv6 (if needed):

```
/routing bgp peer
add address-families=ipv6 comment="DN42: somepeer IPv6" in-filter=dn42-in \
instance=bgp-dn42-somename multihop=yes name=dn42-somepeer-ipv6 out-filter=dn42-out \
remote-address=fd42:c644:5222:3222::40 remote-as=PEER_AS route-reflect=yes ttl=default
```

Also, as a note, Mikrotik RoS 6.x doesn't deal well with BGP running over link-local addresses (the address starting with fe80). You need to use a fd42:: address in your BGP session, otherwise, BGP will not install any received route.

#### BGP Advertisements
You want to advertise your allocated network (most likely), it's very simple:

```
/routing bgp network
add network=YOUR_ALLOCATED_SUBNET synchronize=no
```
You can repeat that with as much IPv4 and IPv6 networks which you own.

#### RoS 7.x

First difference from v6.x: There is no "network" menu. We advertise our networks now by adding them to the firewall address-list and referencing in the BGP configuration. Also, we can only advertise networks that are part of our static routes. Of course, we can still propagate routes received from others peers.

Adding a network list:
```
IPv4
/ip firewall address-list
add address=YOUR_ALLOCATED_SUBNET list=DN42_allocated_v4

IPv6
/ipv6 firewall address-list
add address=YOUR_ALLOCATED_SUBNET list=DN42_allocated_v6
```

Adding a static route to your full allocated network:
```
/ipv6 route
add blackhole disabled=no distance=1 dst-address=YOUR_ALLOCATED_SUBNET
```

Let's create a template for DN42. It isn't strictly necessary, but it makes our life easier when adding more peers in future.
```
/routing bgp template
add afi=ipv4 as=YOUR_AS_NUMBER name=DN42_template_v4 output.network=DN42_allocated_v4 router-id=1.1.1.1
add afi=ipv6 as=YOUR_AS_NUMBER name=DN42_template_v6 output.network=DN42_allocated_v6 router-id=1.1.1.1
```

Now is time to add one peer:

Another difference from RoS v6.x is that v7.x can use link-local adresses (validated with RoS 7.14.3, 7.18.1, 7.18.2 and 7.19rc2). The trick is to add "%INTERFACE" after the address, where "INTERFACE" is the name of the interface the link-local is allocated to - or the interface used to get to that remote link-local. So, if You want to listen on fe80::1 on the "myPeer" interface, the address would be "fe80::1%myPeer".

RoS 7.17 and newer can set the link local address.


```
IPv4 peer
/routing bgp connection
add address-families=ipv4 disabled=no input.filter=dn42-in \
local.address=ADDRESS_YOUR_PEER_USE_TO_CONNECT_ON_YOU .role=ebgp \
multihop=yes name=PEER_NAME output.filter-chain=dn42-out \
.network=DN42_allocated_v4 remote.address=YOUR_PEER_REMOTE_ADDRESS \
.as=PEER_AS_NUMBER routing-table=main templates=DN42_template_v4

IPv6 peer
/routing bgp connection
add address-families=ipv6 disabled=no input.filter=dn42-in \
local.address=ADDRESS_YOUR_PEER_USE_TO_CONNECT_ON_YOU .role=ebgp \
multihop=yes name=PEER_NAME output.filter-chain=dn42-out \
.network=DN42_allocated_v6 remote.address=YOUR_PEER_REMOTE_ADDRESS \
.as=PEER_AS_NUMBER routing-table=main templates=DN42_template_v6
```


## Split DNS

You can separate DNS requests for the .dn42 TLD from your default DNS traffic. This allows regular (non-DN42) lookups to work as normal, while .dn42 queries are handled on the DN42 network.

Adjust the network and LAN GW in these examples to match your own network configuration.

### RoS 6.47 and later

Newer versions of RouterOS can redirect DNS queries according to special rules. We add a "static" DNS mapping that forwards matching queries to a specific DNS server:
```
/ip dns static
add comment=DN42 forward-to=172.23.0.53 regexp=".*\\.dn42" type=FWD
```

### RoS earlier than 6.47

DNS redirection can be achieved with a Layer 7 (L7) filter in RouterOS. We define a new L7 protocol by matching the body of the DNS query, then use NAT to redirect those queries to a DN42 DNS server.

In this example we assume that your LAN hosts use the 192.168.0.0/24 subnet, and your gateway is 192.168.0.1

```
/ip firewall layer7-protocol
add name=DN42-DNS regexp="\\x04dn42.\\x01"

/ip firewall nat
add action=src-nat chain=srcnat comment="NAT to DN42 DNS" dst-address=172.23.0.53 dst-port=53 protocol=udp src-address=192.168.0.0/24 to-addresses=192.168.0.1
add action=dst-nat chain=dstnat dst-address-type=local dst-port=53 layer7-protocol=DN42-DNS protocol=udp src-address=192.168.0.0/24 to-addresses=172.23.0.53 to-ports=53
```

## Specifying BGP Communities (v7)

See the [BGP communities](/howto/BGP-communities) page to understand what this means.

In this example we are applying community numbers 5 (peer link latency of 55-148ms) and 41 (prefixes originate from Europe) to our advertisements.

```
/routing/filter/community-list
add list=dn42 communities=64511:41
add list=dn42 communities=64511:5

/routing/filter/rule/
add chain=dn42-out rule="append bgp-communities dn42"
```
