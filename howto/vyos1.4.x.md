# VyOS 1.4.x sagitta
VyOS is an open source software router.  It is feature rich and supports multiple deployment options such as physical hardware (Old PC's) or a VPC/VM.  The developers have a nightly rolling release that includes all the latest features such as Wireguard.

It can be downloaded here <https://www.vyos.io/rolling-release/>.  

## Firewall Baseline
We will configure firewall access lists for inbound connections on our peer Wireguard interfaces as well as block all inbound connections to our router with the exception of BGP.  This should be a good baseline firewall ruleset to filter inbound traffic on your network's edge.  Modifications may be needed depending on your specific goals.  If your router has an uplink back to a larger internal network (outside of DN42), an outbound firewall ruleset will need to be applied to that interface.

By default, VyOS is a **stateless** firewall.  To enable **stateful** packet inspection globally enter the following commands.
```
set firewall state-policy established action 'accept'
set firewall state-policy related action 'accept'
```

We also need to accept invalids on our network's edge.  However, this should not become common practice elsewhere.
```
set firewall state-policy invalid action 'accept'
```

The below commands create **in** and **local** baseline templates to be applied to all Wireguard interfaces that are facing peers.  In this example, **172.20.20.0/24** and **fd88:9deb:a69e::/48** are your assigned address spaces.
```
#Create Groups v4
set firewall group network-group Allowed-Transit-v4 network '10.0.0.0/8'
set firewall group network-group Allowed-Transit-v4 network '172.20.0.0/14'
set firewall group network-group Allowed-Transit-v4 network '172.31.0.0/16'

set firewall group network-group My-Assigned-Space-v4 network '172.20.20.0/24'

#Create Groups v6
set firewall group ipv6-network-group Allowed-Transit-v6 network 'fd00::/8'

set firewall group ipv6-network-group My-Assigned-Space-v6 network 'fd88:9deb:a69e::/48'


#Inbound Connections v4
set firewall name Tunnels_In_v4 default-action 'drop'
set firewall name Tunnels_In_v4 enable-default-log
set firewall name Tunnels_In_v4 rule 68 action 'drop'
set firewall name Tunnels_In_v4 rule 68 description 'Block Traffic to Operator Assigned IP Space'
set firewall name Tunnels_In_v4 rule 68 destination group network-group 'My-Assigned-Space-v4'
set firewall name Tunnels_In_v4 rule 68 log 'enable'
set firewall name Tunnels_In_v4 rule 68 action 'drop'
set firewall name Tunnels_In_v4 rule 70 action 'accept'
set firewall name Tunnels_In_v4 rule 70 description 'Allow Peer Transit'
set firewall name Tunnels_In_v4 rule 70 destination group network-group 'Allowed-Transit-v4'
set firewall name Tunnels_In_v4 rule 70 source group network-group 'Allowed-Transit-v4'
set firewall name Tunnels_In_v4 rule 70 log 'enable'
set firewall name Tunnels_In_v4 rule 99 action 'drop'
set firewall name Tunnels_In_v4 rule 99 description 'Black Hole'
set firewall name Tunnels_In_v4 rule 99 log 'enable'

#Inbound Connections v6
set firewall ipv6-name Tunnels_In_v6 default-action 'drop'
set firewall ipv6-name Tunnels_In_v6 enable-default-log
set firewall ipv6-name Tunnels_In_v6 rule 68 action 'drop'
set firewall ipv6-name Tunnels_In_v6 rule 68 description 'Block Traffic to Operator Assigned IP Space'
set firewall ipv6-name Tunnels_In_v6 rule 68 destination group network-group 'My-Assigned-Space-v6'
set firewall ipv6-name Tunnels_In_v6 rule 68 log 'enable'
set firewall ipv6-name Tunnels_In_v6 rule 70 action 'accept'
set firewall ipv6-name Tunnels_In_v6 rule 70 description 'Allow Peer Transit'
set firewall ipv6-name Tunnels_In_v6 rule 70 destination group network-group 'Allowed-Transit-v6'
set firewall ipv6-name Tunnels_In_v6 rule 70 log 'enable'
set firewall ipv6-name Tunnels_In_v6 rule 70 source group network-group 'Allowed-Transit-v6'
set firewall ipv6-name Tunnels_In_v6 rule 99 action 'drop'
set firewall ipv6-name Tunnels_In_v6 rule 99 description 'Black Hole'
set firewall ipv6-name Tunnels_In_v6 rule 99 log 'enable'

#Local Connections v4
set firewall name Tunnels_Local_v4 default-action 'drop'
set firewall name Tunnels_Local_v4 rule 50 action 'accept'
set firewall name Tunnels_Local_v4 rule 50 icmp
set firewall name Tunnels_Local_v4 rule 50 protocol 'icmp'
set firewall name Tunnels_Local_v4 rule 61 action 'accept'
set firewall name Tunnels_Local_v4 rule 61 description 'Allow BGP'
set firewall name Tunnels_Local_v4 rule 61 destination port '179'
set firewall name Tunnels_Local_v4 rule 61 protocol 'tcp'
set firewall name Tunnels_Local_v4 rule 98 action 'drop'
set firewall name Tunnels_Local_v4 rule 98 description 'Black Hole'
set firewall name Tunnels_Local_v4 rule 98 log 'enable'
set firewall name Tunnels_Local_v4 rule 98 state invalid 'enable'
set firewall name Tunnels_Local_v4 rule 99 action 'drop'
set firewall name Tunnels_Local_v4 rule 99 description 'Black Hole'
set firewall name Tunnels_Local_v4 rule 99 log 'enable'

#Local Connections v6
set firewall ipv6-name Tunnels_Local_v6 default-action 'drop'
set firewall ipv6-name Tunnels_Local_v6 rule 50 action 'accept'
set firewall ipv6-name Tunnels_Local_v6 rule 50 icmpv6
set firewall ipv6-name Tunnels_Local_v6 rule 50 protocol 'ipv6-icmp'
set firewall ipv6-name Tunnels_Local_v6 rule 61 action 'accept'
set firewall ipv6-name Tunnels_Local_v6 rule 61 description 'Allow BGP'
set firewall ipv6-name Tunnels_Local_v6 rule 61 destination port '179'
set firewall ipv6-name Tunnels_Local_v6 rule 61 protocol 'tcp'
set firewall ipv6-name Tunnels_Local_v6 rule 98 action 'drop'
set firewall ipv6-name Tunnels_Local_v6 rule 98 description 'Black Hole'
set firewall ipv6-name Tunnels_Local_v6 rule 98 log 'enable'
set firewall ipv6-name Tunnels_Local_v6 rule 98 state invalid 'enable'
set firewall ipv6-name Tunnels_Local_v6 rule 99 action 'drop'
set firewall ipv6-name Tunnels_Local_v6 rule 99 description 'Black Hole'
set firewall ipv6-name Tunnels_Local_v6 rule 99 log 'enable'
```

## Wireguard
### Setup Keys
You can choose to generate a unique keypair and use it for every wireguard peering, or you can choose to generate a different one for each new peering.
```
generate pki wireguard key-pair

#Output example:
Private key: SOoPQdMdmXE3ssp0/vwwoIMhQqvcQls+DhDjmaLw03U=
Public key: ArkXeK1c0pCWCouePcRRBCQpXfi4ZIvRFFwTxO60dxs=
```

If you choose to generate unique keypairs for peerings, you can generate and install the keypair in a single command. Note that you have to be in `configure` mode, at the top level, as shown below:
```shellsession
vyos@vyos$ configure

[edit]
vyos@vyos# run generate pki wireguard key-pair install interface wg4242424242
1 value(s) installed. Use "compare" to see the pending changes, and "commit" to apply.
Corresponding public-key to use on peer system is: 'UcqcZsJvq1MlYgo3gObjaJ8FH+N7wkfV+EH3YDAMyRE='
[edit]
vyos@vyos-home# show interfaces wireguard wg4242424242 
+private-key kHCqfe/GZ8phoNnWfkL3+joXi/qK3ZfdfAnlNuX/9FU=
```

To retrieve keys later, use the op-mode command `show interfaces wireguard wg4242424242 public-key`.

Example:
```shellsession
vyos@vyos$ show interfaces wireguard wg4242424242 public-key 
UcqcZsJvq1MlYgo3gObjaJ8FH+N7wkfV+EH3YDAMyRE=
```

### Configure First Peer's tunnel
This example assumes that your ASN is 4242421234 and your peer's ASN is 4242424242
```
set interfaces wireguard wg4242424242 description 'AS4242424242 - My First Peer'

# Common practice on DN42 is for peers to use 2+the last four digits of your peer's ASN as the port.
# You will have to let your peer know what you choose for your port, as well as your clearnet IP address.
set interfaces wireguard wg4242424242 port '24242'
set interfaces wireguard wg4242424242 private-key 'SOoPQdMdmXE3ssp0/vwwoIMhQqvcQls+DhDjmaLw03U='

# An arbitrary link-local IPv6 address (that you'll have to tell to your peer)
set interfaces wireguard wg4242424242 address 'fe80::1234/64'

# One of your DN42 IPv4 addresses (not really needed if you'll enable extended next-hop)
set interfaces wireguard wg4242424242 address '172.20.20.1/32'

# Set your peer's clearnet endpoint information. You need to use an IPv4 or IPv6 address
# (as opposed to a DNS name).
# If you have a static IP address but your peer does not,
# you can leave out this part of the configuration.
set interfaces wireguard wg4242424242 peer location1 address '192.0.2.1'
set interfaces wireguard wg4242424242 peer location1 port '21234'

# You can allow everything here and relay on your firewall
set interfaces wireguard wg4242424242 peer location1 allowed-ips '0.0.0.0/0'
set interfaces wireguard wg4242424242 peer location1 allowed-ips '::/0'
set interfaces wireguard wg4242424242 peer location1 public-key '<wireguard public key of your peer>'

# (persistent-keepalive option could be optional, but in my case I noticed that helps starting BGP session)
set interfaces wireguard wg4242424242 peer location1 persistent-keepalive '60'

# Configure firewall
set firewall interface wg4242424242 interface-group ipv6-name 'Tunnels_In_v6'
set firewall interface wg4242424242 interface-group name 'Tunnels_In_v4'
set firewall interface wg4242424242 local ipv6-name 'Tunnels_Local_v6'
set firewall interface wg4242424242 local name 'Tunnels_Local_v4'

```

## BGP
Now that we have a tunnel to our peer and theoretically can ping them, we can setup BGP.  
### Initial Router Setup
```
# Set your ASN and IP blocks
set protocols bgp system-as '4242421234'

set protocols bgp address-family ipv4-unicast network 172.20.20.0/24`  
set protocols bgp address-family ipv6-unicast network fd88:9deb:a69e::/48`

# Note that your address blocks should match your exact prefix as listed in the registry.
# if you try to advertise a subnet of your assigned block, it could get filtered by some peers.

# To keep it simple, just make your router ID match your lower IP within the DN42 registered space.
set protocols bgp parameters router-id '172.20.20.1'
```


### Neighbor Up With Peers
#### Option 1: MP-BGP (with Multi Protocol) - with Extended Next-Hop
MP-BGP peerings over IPv6 are recommended on DN42.
```
# For these examples, your peer's link-local address is fe80::4242

set protocols bgp neighbor fe80::4242 interface v6only remote-as '4242424242'
set protocols bgp neighbor fe80::4242 remote-as '4242424242'
set protocols bgp neighbor fe80::4242 interface source-interface 'wg4242424242'
set protocols bgp neighbor fe80::4242 update-source 'wg4242424242'
set protocols bgp neighbor fe80::4242 description 'FriendlyNet'

# Set the RFC 9234 role to "peer". 
set protocols bgp neighbor fe80::4242 local-role peer

set protocols bgp neighbor fe80::4242 capability extended-nexthop

set protocols bgp neighbor fe80::4242 address-family ipv4-unicast 
set protocols bgp neighbor fe80::4242 address-family ipv6-unicast 

```
#### Option 2: BGP (no Multi Protocol) - no Extended Next-Hop
```
# First, we set the ipv6 part.
set protocols bgp neighbor fe80::4242 interface remote-as '4242424242'
set protocols bgp neighbor fe80::4242 interface source-interface 'wg4242424242'
set protocols bgp neighbor fe80::4242 remote-as '4242424242'
set protocols bgp neighbor fe80::4242 address-family ipv6-unicast 
set protocols bgp neighbor fe80::4242 description 'FriendlyNet'

# For the ipv4 part we need to add first a static ipv4 route to our peer tunneled ipv4 address
set protocols static route 172.20.x.y interface wg1234

# 172.20.x.y is your peer tunneled IPv4
set protocols bgp neighbor 172.20.x.y remote-as '<your peer ASN>'
set protocols bgp neighbor 172.20.x.y address-family ipv4-unicast 
set protocols bgp neighbor 172.20.x.y description 'FriendlyNet'

# This setting may need to be adjusted depending on circumstances
set protocols bgp neighbor 172.20.x.y ebgp-multihop 20
```


You can now check your BGP summary:

```shellsession
vyos@vyos$ show ip bgp summary

IPv4 Unicast Summary (VRF default):
BGP router identifier 172.20.20.1, local AS number 4242421234 vrf-id 0
BGP table version 2782
RIB entries 1378, using 258 KiB of memory
Peers 1, using 1 MiB of memory
Peer groups 1, using 64 bytes of memory

Neighbor               V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
fe80::4242             4 4242424242      1031         6        0    0    0 00:04:20          710        1 FriendlyNet

IPv6 Unicast Summary (VRF default):
BGP router identifier 172.20.20.1, local AS number 4242421234 vrf-id 0
BGP table version 2782
RIB entries 1378, using 258 KiB of memory
Peers 1, using 1 MiB of memory
Peer groups 1, using 64 bytes of memory

Neighbor               V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
fe80::4242             4 4242424242      1031         6        0    0    0 00:04:20          710        1 FriendlyNet
```

Setting up peer-groups might help standardize multiple peerings:

```
# One peer group for all IPv6 MP-BGP link-local extended-nexthop peers
set protocols bgp peer-group dn42 address-family ipv4-unicast
set protocols bgp peer-group dn42 address-family ipv6-unicast
set protocols bgp peer-group dn42 capability extended-nexthop
set protocols bgp peer-group dn42 local-role peer

set protocols bgp neighbor fe80::4242 peer-group dn42

# If you have any non-multiprotocol peerings you'll need to set up peer-groups
# for the individual address families. This is left up to the reader.

# Delete the settings that are now redundant
delete protocols bgp neighbor fe80::4242 address-family
delete protocols bgp neighbor fe80::4242 capability
```


## RPKI/ROA Checking
### Setup RPKI Caching Server
Burble has made this super easy.  More info can be found [here](/howto/ROA-slash-RPKI) on this wiki.  Get started by running the below command on a Linux server with Docker installed (VyOS now supports containers, but doesn't yet supports commands to pass to them... so we still need another machine to run GoRTR)

```  
sudo docker run -ti -p 8082:8082 cloudflare/gortr -cache https://dn42.burble.com/roa/dn42_roa_46.json -verify=false -checktime=false -bind :8082
```

This will start a docker container that listens on the host server's IP at port 8082.  This setup is using Cloudflare's GoRTR and automatically reaching out and downloading a custom JSON file generated by Burble just for the DN42 network.  

### Point VyOS Router at RPKI Caching Server
```
set protocols rpki cache <ip address of your GoRTR instance> port '8082'
set protocols rpki cache <ip address of your GoRTR instance> preference '1'   
```

You can check the connection with `show rpki cache-connection` and the received prefix-table with `show rpki prefix-table`.  

### Create Route Map
```
set policy route-map DN42-ROA rule 10 action 'permit'
set policy route-map DN42-ROA rule 10 match rpki 'valid'
set policy route-map DN42-ROA rule 20 action 'permit'
set policy route-map DN42-ROA rule 20 match rpki 'notfound'
set policy route-map DN42-ROA rule 30 action 'deny'
set policy route-map DN42-ROA rule 30 match rpki 'invalid'
```
This example allows all routes in unless they are marked invalid or in other words possibly been a victim of BGP hijacking.
You can also consider to "deny" the "notfound" prefixes, for better control.

You can also consider to combine within the same route-map the RPKI and one or more a prefix lists containing your internal network prefixes, as described later (The example "No RPKI/ROA and Internal Network Falls Into DN42 Range").

### Assign Route Map to Neighbor
```
set protocols bgp neighbor fe80::1234 address-family ipv4-unicast route-map export 'DN42-ROA'
set protocols bgp neighbor fe80::1234 address-family ipv4-unicast route-map import 'DN42-ROA'
set protocols bgp neighbor fe80::1234 address-family ipv6-unicast route-map export 'DN42-ROA'
set protocols bgp neighbor fe80::1234 address-family ipv6-unicast route-map import 'DN42-ROA' 
```
_Remember to do that for all your new peerings!_

## Example Route Map
### No RPKI/ROA and Internal Network Falls Into DN42 Range
```
##Build prefix list to match personal internal network
set policy prefix-list BlockIPConflicts description 'Prevent Conflicting Routes'
set policy prefix-list BlockIPConflicts rule 10 action 'permit'
set policy prefix-list BlockIPConflicts rule 10 description 'Internal IP Space'
set policy prefix-list BlockIPConflicts rule 10 le '32'
set policy prefix-list BlockIPConflicts rule 10 prefix '10.10.0.0/16'


##Build prefix list to match personal internal network
set policy prefix-list6 BlockIPConflicts-v6 description 'Prevent Conflicting Routes'
set policy prefix-list6 BlockIPConflicts-v6 rule 10 action 'permit'
set policy prefix-list6 BlockIPConflicts-v6 rule 10 description 'Internal IP Space'
set policy prefix-list6 BlockIPConflicts-v6 rule 10 le '128'
set policy prefix-list6 BlockIPConflicts-v6 rule 10 prefix 'fd42:4242:1111::/48'



##Build prefix list to match DN42's IPv4 network
set policy prefix-list DN42-Network rule 10 action 'permit'
set policy prefix-list DN42-Network rule 10 le '32'
set policy prefix-list DN42-Network rule 10 prefix '172.20.0.0/14'
set policy prefix-list DN42-Network rule 20 action 'permit'
set policy prefix-list DN42-Network rule 20 le '32'
set policy prefix-list DN42-Network rule 20 prefix '10.0.0.0/8'


##Build prefix list to match DN42's IPv6 network
set policy prefix-list6 DN42-Network-v6 rule 10 action 'permit'
set policy prefix-list6 DN42-Network-v6 rule 10 le '128'
set policy prefix-list6 DN42-Network-v6 rule 10 prefix 'fd00::/8'




##Block prefixes within internal network range, then allow everything else within DN42, then block everything else.
set policy route-map Default-Peering rule 10 action 'deny'
set policy route-map Default-Peering rule 10 description 'Prevent IP Conflicts'
set policy route-map Default-Peering rule 10 match ip address prefix-list 'BlockIPConflicts'
set policy route-map Default-Peering rule 11 action 'deny'
set policy route-map Default-Peering rule 11 description 'Prevent IP Conflicts'
set policy route-map Default-Peering rule 11 match ip address prefix-list6 'BlockIPConflicts-v6'
set policy route-map Default-Peering rule 20 action 'permit'
set policy route-map Default-Peering rule 20 description 'Allow DN42-Network'
set policy route-map Default-Peering rule 20 match ip address prefix-list 'DN42-Network-Network'
set policy route-map Default-Peering rule 21 action 'permit'
set policy route-map Default-Peering rule 21 description 'Allow DN42-Network'
set policy route-map Default-Peering rule 21 match ip address prefix-list6 'DN42-Network-Network-v6'
set policy route-map Default-Peering rule 99 action 'deny'


##Apply the route-map on import/export

set protocols bgp neighbor x.x.x.x address-family ipv4-unicast route-map export 'Default-Peering'
set protocols bgp neighbor x.x.x.x address-family ipv4-unicast route-map import 'Default-Peering'
set protocols bgp neighbor x.x.x.x address-family ipv6-unicast route-map export 'Default-Peering'
set protocols bgp neighbor x.x.x.x address-family ipv6-unicast route-map import 'Default-Peering' 
```

## Credits
This How-To has to be considered a work-in-progress by **Matwolf** with parts co-authored by **bri**

It's based on the original VyOS How-To made by **Owens Research**: [How-To/VyOS](/howto/vyos).

The commands in this page have been adapted to be compatible with the new version of VyOS 1.4.x (sagitta) and to include configurations for IPv6 (MP-BGP over link-local and extended next-hop).

If you have any questions or suggestions please reach out.

## See also
[WireGuard](https://docs.vyos.io/en/latest/configuration/interfaces/wireguard.html) and [BGP](https://docs.vyos.io/en/latest/configuration/protocols/bgp.html) in the official VyOS documentation.