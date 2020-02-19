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

Also, you can't use a /32 on the GRE/PtP links. Even if you add a local route to your peer, BGP can't resolve the installed routes using "a nexthop interface". Please use any /30 on the GRE link, either from your assigned DN42 pool address or use a private address like 192.168. Please don't choose from 172.16.0.0/12 or 10.0.0.0/8 because they may overlap with DN42 or ChaosVPN.

## Tunnel

### IPSec
First, let's add IPSec peer and encryption policy.  
Peer most likely provided you with encryption details.  
If not, ask him about it.
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
Pretty straightforward here

```
/interface gre
add allow-fast-path=no comment="DN42 somepeer" local-address=2.2.2.2 name=gre-dn42-peer \
remote-address=1.1.1.1
```

### IPs inside the GRE tunnel
Your peer most likely provided you with IP adresses for GRE tunnel.  
As I said before, you can't use /31 for PtP links, so we will be using /30.
BGP can't resolve routes if you use a /32 in the GRE link.

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
add address=fdc8:c633:5319:3300::41/127 advertise=no interface=gre-dn42-moos
```

If you configured everything correctly, you should be able to ping 

## BGP

### Filters
It's a good idea to setup filters for BGP instances, both IN (accept advertises) and OUT (send advertises)  
In this example, we will be filtering IN: 192.168.0.0/16 and 169.254.0.0/16  
OUT: 192.168.0.0/16 and 169.254.0.0/16, you really don't want to advertise this networks.  
This filter will not only catch /8 or /16 networks, but smaller networks inside this subnets as well.  

```
/routing filter
add action=discard address-family=ip chain=dn42-in prefix=192.168.0.0/16 prefix-length=16-32 protocol=bgp
add action=discard address-family=ip chain=dn42-in prefix=169.254.0.0/16 prefix-length=16-32 protocol=bgp
add action=discard address-family=ip chain=dn42-out prefix=192.168.0.0/16 prefix-length=16-32 protocol=bgp
add action=discard address-family=ip chain=dn42-out prefix=169.254.0.0/16 prefix-length=16-32 protocol=bgp
```

Now, if you want only DN42 connection, you can filter IN 10.0.0.0/8 (ChaosVPN / freifunk networks):

```
/routing filter
add action=discard address-family=ip chain=dn42-in prefix=10.0.0.0/8 prefix-length=8-32 protocol=bgp
```

### BGP
Now, for actual BGP configuration.

```
/routing bgp instance
set default disabled=yes
add as=YOUR_AS client-to-client-reflection=no name=bgp-dn42-somename out-filter=dn42-in \
router-id=1.1.1.1
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

Also, as a note, Mikrotik doesn't deal well with BGP running over link-local addresses (the address starting with fe80). You need to use a fd42:: address in your BGP session, otherwise, BGP will not install any received route.

### BGP Advertisements
You want to advertise your allocated network (most likely), it's very simple:  

```
/routing bgp network
add network=YOUR_ALLOCATED_SUBNET synchronize=no
```
You can repeat that with as much IPv4 and IPv6 networks which you own.

## Split DNS
Separate dns requests for dn42 tld from your default dns traffic with L7 filter in Mikrotik.
Change network and LAN GW to mach your network configuration.

```
/ip firewall layer7-protocol
add name=DN42-DNS regexp="\\x04dn42.\\x01"
/ip firewall nat
add action=src-nat chain=srcnat comment="NAT to DN42 DNS" dst-address=172.23.0.53 dst-port=53 protocol=udp src-address=192.168.0.0/24 to-addresses=192.168.0.1
add action=dst-nat chain=dstnat dst-address-type=local dst-port=53 layer7-protocol=DN42-DNS protocol=udp src-address=192.168.0.0/24 to-addresses=172.23.0.53 to-ports=53

```