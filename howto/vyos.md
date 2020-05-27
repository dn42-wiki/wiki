#VyOS
VyOS is an open source router.  The developers have a nightly rolling release that includes all the latest features such as Wireguard.  It can be downloaded here https://www.vyos.io/rolling-release/.  


_1.3-rolling-202004300117 is a known good release to work with Wireguard and DN42._


##Quick Start
###Quick to-do-list from router deployment to receiving DN42 routes
1. Establish internet connectivity.
2. Setup Wireguard.
3. Setup BGP.
4. `show ip route`


##Wireguard
1. First we need to setup keys.  `generate wireguard default-keypair`
2. Grab your public key and save it for later.  This will be shared with peers.  `show wireguard keypairs pubkey default`
3. Next we need to configure our peer.  They should have provided their endpoint public IP, port, and public wireguard key.
`set interfaces wireguard wg01 address '172.x.x.x/32'` - this is a single address within your DN42 registered address space
`set interfaces wireguard wg01 peer OtherGuy1 allowed-ips '0.0.0.0/0''` - it's just easier to filter traffic with the firewall
`set interfaces wireguard wg01 peer OtherGuy1 address 'x.x.x.x'` - this is the public IP of your peers endpoint
`set interfaces wireguard wg01 OtherGuy1 port '12345'` - the configured port on your peers endpoint
`set interfaces wireguard wg01 peer OtherGuy1 pubkey 'XMrlPykaxhdAAiSjhtPlvi30NVkvLQliQuKP7AI7CyI='` - your peers public wireguard key
`set interfaces wireguard wg01 port '12345'` - the port your wireguard endpoint will "listen" on
4. In case you are wondering how you are going to route packets anywhere with a /32, the next command explains it all. 
`set protocols static interface-route 172.x.x.x/32 next-hop-interface wg01` - this is a single provided address by your peer that is assigned to them in the registry


####Coming Soon

Setup BGP

Neighbor Up With Peer via BGP

RPKI/ROA Checking

Firewall Would be a good idea…










