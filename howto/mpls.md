mpls label switching is faster because it's a divide and conquer search in an ordered list, compared to routing, which is a longest prefix match, which is a search in a netmask deep tree

and doing just label switching, especially with multiple labels, have consequences like

you can provide vpns, be that layer2 or layer3 on the same infra, we can source-route through arbitrary paths we want, and so on....

you can control visibility / reachability by route target export / imports, so you can hide various routes from specific endpoints, then they'll become unreachable just for them, basically rendering packet filtering unnecesary

hiding service addresses (ip / mac) from the infra resulting in less resource needs: in the simplest mpls, you dont need bgp route table only where the packet enters the network

you can hide your core from traceroute by disabling ip ttl propagation


hints:

as being layer2.5 technology, you'll need a tunnel which carry ethettype, like gre

inside the core you can do ldp, rsvp-te (strategic or auto-tunnel) or segment-routing

between two ases, you can enable ipv4/ipv6 labeled-unicast address family

to do inter-as-mpls-vpn on top of it, you can enable rr-to-rr, asbr-to-asbr or rr-to-asbr vpnv4/vpnv6/vpls/evpn peerings


participating networks:

nop-mnt



planned:

C4TG1RL5-famfo

Fortless
