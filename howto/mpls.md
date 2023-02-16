mpls label switching is faster because it's a divide and conquer search in an ordered list, compared to routing, which is a longest prefix match search

and doing just label switching, especially with multiple labels, have consequences like

you can provide vpns, be that layer2 or layer3 on the same infra, we can source-route through arbitrary paths we want, and so on....

you can control visibility / reachability by route target export / imports, so you can hide various routes from specific endpoints, then they'll become unreachable just for them, basically rendering packet filtering unnecesary

hiding service addresses (ip / mac) from the infra resulting in less resource needs: in the simplest mpls, you dont need bgp route table only where the packet enters the network

you can hide your core from traceroute by disabling ip ttl protopagation



participating networks:

nop-mnt



planned:

C4TG1RL5-famfo

Fortless
