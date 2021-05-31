# How to setup Mikrotik with point-to-point /32 address on interfaces

## RouterOS issues

 * RouterOS doesn't have direct Point-to-Point addresses.
 * BGP doesn't resolve the next-hop for their routes using a rescursive route that uses a interface as next-hop

The long explanation about how mikrotik resolves recursive routes is documentated at [Mikrotik's page](https://wiki.mikrotik.com/wiki/Manual:IP/Route#Nexthop_lookup).

How can we workaround these issues? Simple. We setup a /32 on the Point-to-Point interface, we setup a direct route to the other peer (using the interface as next-hop for this route) and use bgp filters to change the next-hop interface.

## Legend

 * 172.24.0.1 -> Your /32 inside tunnel address
 * 172.26.2.2 -> Peer's /32 inside tunnel address.
 * gre-dn42-peer -> This is the name of the interface
 * 1.1.1.1 - peer external IP
 * 2.2.2.2 - your external IP
 * bgp-dn42-peer-in -> This is the name of the chain filter. You need to use a different chain per point to point link

## Setup

You create the GRE interface in the same way the [Mikrotik Guide](/howto/mikrotik) does.

````
/interface gre
add allow-fast-path=no comment="DN42 somepeer" local-address=2.2.2.2 name=gre-dn42-peer \
remote-address=1.1.1.1
````

Next you add the /32 address on the interface. You can install this address on a loop interface (on RouterOS this means an empty bridge) if you plan to use the same address over several GRE tunnels or other OpenVPN interfaces.

````
/ip address add address=172.24.0.1/32 interface=gre-dn42-peer
````

Next, we add the direct route as next-hop using the interface

````
/ip route add distance=1 dst-address=172.26.2.2/32 gateway=gre-dn42-peer pref-src=172.24.0.1
````

At this point, the ping with the peer should work. Also, the bgp session can be established, but the routes will not work. We need a input filter to fix the next-hop routes.

````
/routing filter add chain=bgp-dn42-peer-in protocol=bgp set-in-nexthop-direct=gre-dn42-peer
````

if you have other global input chain filters, you should add a jump in the same chain, like this:
````
/routing filter add action=jump chain=bgp-dn42-peer-in protocol=bgp jump-target=bgp-global-dn42-input
````

If you haven't created the BGP session, create it now from the [Mikrotik guide](/howto/mikrotik#how-to-connect-to-dn42-using-mikrotik-routeros_bgp). Change the peer input filter to use the chain we've just created:

````
/routing bgp peer set bgp-dn42-somename in-filter=bgp-dn42-peer-in
````

With this fix, all the routes will have set next-hop the GRE interface and there will be no need to use RouterOS' recursive route resolve.

Check the routes with:
````
/ip routes print detail where received-from=bgp-dn42-somename
````

There should an attribute like:
````
gateway=gre-dn42-peer gateway-status=gre-dn42-peer reachable
````