# EdgeOS  

This document describes some possibilities for connecting to dn42 using an Ubiquiti EdgeRouter:

* IPv4/IPv6 tunnel via:
    * OpenVPN - support built into EdgeOS already - covered below
    * IPsec/IKEv2 - support built into EdgeOS already - not covered here
    * QuickTun - see [vyatta-quicktun package](https://github.com/neilalexander/vyatta-quicktun) - not covered here
* Route exchange using BGP
* DNS resolution for the .dn42 TLD

## First Steps

1. Create the required objects in the Registry - see [[Getting started]] 
2. Find a peer - ask nicely in [[IRC]]!
3. Get the following details:
    * Tunnel configuration (OpenVPN, IPsec, QuickTun)
    * AS numbers 

### Tunnel Configuration

### OpenVPN 

Using the below as examples:

    Own ASN: AS111111  
    Own IPv4 Space: 172.AA.AA.64/27  
    Own IPv6 Space: fdBB:BBBB:CCCC::/48  
    Own IPv4 If-Address: 172.AA.AA.65  
    Own IPv6 If-Address: fdBB:BBBB:CCCC::1   

    Peer OpenVPN Remote Address: 172.X.X.X  //that's the peers OpenVPN IF IP  
    Peer OpenVPN Remote Host: X.X.X.Y  //that's the peers clearnet IP  
    Peer OpenVPN IP for you: fdAA::BBB/64  
    Peer OpenVPN IP: fdAA::CC  
    Peer OpenVPN Port: 1194  
    Peer OpenVPN encryption: aes256  
    Peer ASN: AS222222  
    Peer BGP Neighbour IPv4: Z.Z.Z.Z  
    Peer BGP Neighbour IPv6: fdAA::CC  

#### Copy OpenVPN key to the EdgeRouter  

Copy the VPN key to `/config/auth/SomeSharedKey.key`:
 
    sudo cat > /config/auth/SomeSharedKey.key

Paste the key in the terminal window, hit return once and kill `cat` with CTRL+C. Then type `exit`.

####  Create IPv4 OpenVPN Interface

Create the OpenVPN virtual interface, i.e. using `vtun0`:

    configure  
    set interfaces openvpn vtun0  
    set interfaces openvpn vtun0 mode site-to-site  
    set interfaces openvpn vtun0 local-port 1194   
    set interfaces openvpn vtun0 remote-port 1194  
    set interfaces openvpn vtun0 local-address 172.AA.AA.65  
    set interfaces openvpn vtun0 remote-address 172.X.X.X  
    set interfaces openvpn vtun0 remote-host X.X.X.Y   
    set interfaces openvpn vtun0 shared-secret-key-file /config/auth/SomeSharedKey.key    
    set interfaces openvpn vtun0 encryption aes256  

    set interfaces openvpn vtun0 openvpn-option "--comp-lzo"  //if your peer support compression  

    commit   
    save  
    exit  

The OpenVPN tunnel should now be up and running.

Check it with:  

    show interfaces openvpn    
    show interfaces openvpn detail  
    show openvpn status site-to-site  

### Create BGP Session

#### Open Firewall

You need to open the firewall to local for the tunnel Interface on port 179/tcp

#### Configure the BGP Neighbor

When entering AS numbers, do not include the "AS" prefix, i.e. enter AS111111 as just 111111.

Build the BGP session with your peer:

    configure  
    set protocols bgp 111111 neighbor Z.Z.Z.Z remote-as 222222  
    set protocols bgp 111111 neighbor Z.Z.Z.Z soft-reconfiguration inbound  
    set protocols bgp 111111 neighbor Z.Z.Z.Z update-source 172.AA.AA.65  
    commit
    save

Check that the BGP session has come up:

    show ip bgp summary  

#### Create Blackhole Route

so bgp can announce the route  

    set protocols static route 172.AA.AA.64/27 blackhole  
    commit  
    save  

#### Announce Route to BGP
  
    set protocols bgp 111111 network 172.A.A.64/27  
    commit  
    save  
    exit  

You should now be able to see networks being advertised to your peer: 

    show ip bgp neighbors Z.Z.Z.Z advertised-routes  

### Set DNS Forwarding

Try to ping `172.23.0.53` (anycast DNS resolver). If you get a response then you are good to continue. 

Add the DNS forwarder: 

    configure
    set service dns forwarding options server=/23.172.in-addr.arpa/172.23.0.53  
    set service dns forwarding options server=/22.172.in-addr.arpa/172.23.0.53  
    set service dns forwarding options server=/dn42/172.23.0.53  
    commit
    save
    exit

### Create NAT rule

    set service nat rule 5013 outbound-interface vtun0
    set service nat rule 5013 type masquerade
    set service nat rule 5013 description "Masquerade for dn42"

You should now be able to access .dn42 domains. 
