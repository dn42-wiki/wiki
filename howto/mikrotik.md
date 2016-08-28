# How to connect to dn42 using Mikrotik RouterOS


## Legend

 * 1.1.1.1 - peer external IP
 * 2.2.2.2 - your external IP
 * 172.20.1.116 - remote GRE IPv4 address
 * 172.20.1.117 - local GRE IPv4 address
 * fd42:c644:5222:3222::40 - remote GRE IPv6 address
 * fd42:c644:5222:3222::41 - local GRE IPv6 address
 * YOUR_AS - your AS number (numbers only)
 * PEER_AS - peer AS number (numbers only)

## RouterOS limitations

 * IPSec only supports IKEv1
 * OpenVPN only works in tcp mode
 * OpenVPN does not support LZO compression
 * You can't use /31 subnet for PtP links

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

### IPs and routes
Your peer most likely provided you with IP adresses for GRE tunnel.  
As i said before, you can't use /31 for PtP links, so we will be using two /32 with route.  
Add ip your peer provided you:

#### IPv4

```
/ip address
add address=172.20.1.117 interface=gre-dn42-peer network=172.20.1.117
```
Add route to your peer /32:

```
/ip route
add distance=1 dst-address=172.20.1.116/32 gateway=gre-dn42-peer
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
name=dn42-somepeer-ipv4 out-filter=dn42-out remote-address=172.20.1.116 remote-as=PEER_AS \
route-reflect=yes ttl=default
```
IPv6 (if needed):  

```
/routing bgp peer
add address-families=ipv6 comment="DN42: somepeer IPv6" in-filter=dn42-in \ 
instance=bgp-dn42-somename multihop=yes name=dn42-somepeer-ipv6 out-filter=dn42-out \ 
remote-address=fd42:c644:5222:3222::40 remote-as=PEER_AS route-reflect=yes ttl=default
```
### BGP Advertisements
You want to advertise your allocated network (most likely), it's very simple:  

```
/routing bgp network
add network=YOUR_ALLOCATED_SUBNET synchronize=no
```
You can repeat that with as much IPv4 and IPv6 networks which you own.