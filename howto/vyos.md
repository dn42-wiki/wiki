#VyOS
VyOS is an open source software router.  It is feature rich and supports multiple deployment options such as physical hardware (Old PC's) or a VPC/VM.  The developers have a nightly rolling release that includes all the latest features such as Wireguard.

It can be downloaded here https://www.vyos.io/rolling-release/.  

## Firewall Baseline
We will configure firewall access lists for inbound connections on our peer Wireguard interfaces as well as block all inbound connections to our router with the exception of BGP.  This should be a good baseline firewall ruleset to filter inbound traffic on your network’s edge.  Modifications may be needed depending on your specific goals.  If your router has an uplink back to a larger internal network (outside of DN42), an outbound firewall ruleset will need to be applied to that interface. The examples here only cover **IPv4**, but the same concepts can be applied to **IPv6** rulesets.

By default, VyOS is a **stateless** firewall.  To enable **stateful** packet inspection globally enter the following commands.
````
set firewall state-policy established action 'accept'
set firewall state-policy related action 'accept'
````

We also need to accept invalids on our network’s edge.  However, this should not become common practice elsewhere.
````
set firewall state-policy invalid action 'accept'
````

The below commands create **in** and **local** baseline templates to be applied to all Wireguard interfaces that are facing peers.  In this example, **172.20.20.0/24** is your assigned address space.
````
#Create Groups
set firewall group network-group Allowed-Transit-v4 network '10.0.0.0/8'
set firewall group network-group Allowed-Transit-v4 network '172.20.0.0/14'

#Inbound Connections
set firewall name Tunnels_In_v4 default-action 'drop'
set firewall name Tunnels_In_v4 enable-default-log
set firewall name Tunnels_In_v4 rule 68 action 'drop'
set firewall name Tunnels_In_v4 rule 68 description 'Block Traffic to Operator Assigned IP Space'
set firewall name Tunnels_In_v4 rule 68 source address '172.20.20.0/24'
set firewall name Tunnels_In_v4 rule 68 log 'enable'
set firewall name Tunnels_In_v4 rule 68 action 'drop'
set firewall name Tunnels_In_v4 rule 69 description 'Block Traffic to Operator Assigned IP Space'
set firewall name Tunnels_In_v4 rule 69 destination address '172.20.20.0/24'
set firewall name Tunnels_In_v4 rule 69 log 'enable'
set firewall name Tunnels_In_v4 rule 70 action 'accept'
set firewall name Tunnels_In_v4 rule 70 description 'Allow Peer Transit'
set firewall name Tunnels_In_v4 rule 70 destination group network-group 'Allowed-Transit-v4'
set firewall name Tunnels_In_v4 rule 70 source group network-group 'Allowed-Transit-v4'
set firewall name Tunnels_In_v4 rule 70 log 'enable'
set firewall name Tunnels_In_v4 rule 99 action 'drop'
set firewall name Tunnels_In_v4 rule 99 description 'Black Hole'
set firewall name Tunnels_In_v4 rule 99 log 'enable'

#Local Connections
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
````

## Wireguard
### Setup Keys
````
generate wireguard default-keypair
show wireguard keypairs pubkey default
````
_Grab your public key and save it for later.  This will be shared with peers._  
### Configure First Peer
````
#Your DN42 Address
set interfaces wireguard wg92 address '172.20.20.1/32'

#Apply Description and Firewall
set interfaces wireguard wg92 description 'First Peer Example'
set interfaces wireguard wg92 firewall in name 'Tunnels_In_v4'
set interfaces wireguard wg92 firewall local name 'Tunnels_Local_v4'

#Peer Endpoint Address (Clearnet)
set interfaces wireguard wg92 peer location1 address '123.243.141.39'

#Best to allow everything here - This is why we have a firewall
set interfaces wireguard wg92 peer location1 allowed-ips '0.0.0.0/0'

#First Peer's Endpoint Port and Public Key
set interfaces wireguard wg92 peer location1 port '12345'
set interfaces wireguard wg92 peer location1 pubkey '=kjhfkasdhjdsfasdefghjklfghjkl/'

#Port Your Endpoint Listens On
set interfaces wireguard wg92 port '12345'

#Set static interface route to first peers /32 DN42 IPv4 on their tunnel endpoint
set protocols static interface-route 172.20.50.1/32 next-hop-interface wg92
````




##BGP
Now that we have a tunnel to our peer and theoretically can ping them, we can setup BGP.  
###Initial Router Setup  
`set protocols bgp 424242XXXX address-family ipv4-unicast network 172.x.x.x\x`  
_Insert your ASN and your assigned network block.  Note that this should match your exact prefix as listed in the registry; if you try to advertise a subnet of your assigned block it could get filtered by some peers._  
`set protocols bgp 424242XXX parameters router-id 172.x.x.x`  
_To keep it simple just make your router ID match your lower IP within the DN42 registered space._  
###Neighbor Up With Peers
`set protocols bgp 424242XXXX neighbor 172.x.x.x address-family ipv4-unicast`  
_This is likely the same IP as the one used in your static route earlier when creating the Wireguard tunnel._  
`set protocols bgp 424242XXXX neighbor 172.x.x.x ebgp-multihop 20`   
_This setting may need to be adjusted depending on circumstances_  
`set protocols bgp 424242XXXX neighbor 172.x.x.x remote-as 424242XXXX`  
_Your peers ASN_  
  
`show ip bgp summary`

##RPKI/ROA Checking
###Setup RPKI Caching Server
Burble has made this super easy.  More info can be found [here](https://wiki.dn42/howto/ROA-slash-RPKI) on this wiki.  Get started by running the below command on a Linux server with Docker installed.     

````  
sudo docker run -ti -p 8082:8082 cloudflare/gortr -cache https://dn42.burble.com/roa/dn42_roa_46.json -verify=false -checktime=false -bind :8082
````
  
This will start a docker container that listens on the host server's IP at port 8082.  This setup is using Cloudflare's GoRTR and automatically reaching out and downloading a custom JSON file generated by Burble just for the DN42 network.  

###Point VyOS Router at RPKI Caching Server
`set protocols rpki cache GoRTR address x.x.x.x`   
   
`set protocols rpki cache GoRTR port 8082`  
  
You can check the connection with `show rpki cache-connection` and the received prefix-table with `show rpki prefix-table`.  

###Create Route Map
````
set policy route-map DN42-ROA rule 10 action 'permit'
set policy route-map DN42-ROA rule 10 match rpki 'valid'
set policy route-map DN42-ROA rule 20 action 'permit'
set policy route-map DN42-ROA rule 20 match rpki 'notfound'
set policy route-map DN42-ROA rule 30 action 'deny'
set policy route-map DN42-ROA rule 30 match rpki 'invalid'
````
This example allows all routes in unless they are marked invalid or in other words possibly been a victim of BGP hijacking.  
###Assign Route Map to Neighbor
````
set protocols bgp 424242XXXX neighbor x.x.x.x address-family ipv4-unicast route-map import DN42-ROA  
set protocols bgp 424242XXXX neighbor x.x.x.x address-family ipv4-unicast route-map export DN42-ROA   
````
 
## Example Route Map
### No RPKI/ROA and Internal Network Falls Into DN42 Range
````
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

set protocols bgp 4242421099 neighbor x.x.x.x address-family ipv4-unicast route-map export 'Default-Peering'
set protocols bgp 4242421099 neighbor x.x.x.x address-family ipv4-unicast route-map import 'Default-Peering'
set protocols bgp 4242421099 neighbor x.x.x.x address-family ipv6-unicast route-map export 'Default-Peering'
set protocols bgp 4242421099 neighbor x.x.x.x address-family ipv6-unicast route-map import 'Default-Peering'
````


This page is a work-in-progress by Owens Research.  If you have any suggestions or questions please reach out.