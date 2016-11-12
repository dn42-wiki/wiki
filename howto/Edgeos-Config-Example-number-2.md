# EdgeRouterPro-8 config example with v1.9.0 

After a lot of searching and trying I [Phil/ALS7] finnaly got a working config


##Features

* IPv4/IPv6 Tunnel via OpenVPN
* dn42 DNS

##How-To

--> still work in Progress

1) you need to create all required fields in the registry --> look at howto/Getting startet page

The data i used are the following:

Own ASN: AS111111  
Own IPv4: 172.AA.AA.64/27  
Own IPv6: fdBB:BBBB:CCCC::/48  

Peer Remote Address: X.X.X.X    
Peer Remote Host: X.X.X.Y  
Peer Port: 1194  
Peer ASN: AS222222  
Peer BGP Neighbour IPv4: Z.Z.Z.Z  

2) get a peer --> ask nice @irc

3) You need following data from the peer

--tunnel options, secret key
--ASN from the peer 
--ip's 

...

###Create IPv4 OpenVPN Interface

start a ssh session to your router

copy vpn key to /config/auth/giveITaName -- Create folder if needed

    configure  
    set interface openssh vtun0  
    set interfaces openvpn vtun0 mode site-to-site  
    set interfaces openvpn vtun0 local-port 1194   
    set interfaces openvpn vtun0 remote-port 1194  
    set interfaces openvpn vtun0 local-address 172.AA.AA.64  
    set interfaces openvpn vtun0 remote-address X.X.X.X  
    set interfaces openvpn vtun0 remote-host X.X.X.Y   
    set interfaces openvpn vtun0 shared-secret-key-file /config/auth/giveITaName    

    set interfaces openvpn vtun0 openvpn-option "--comp-lzo"  //if your peer support compression  
    commit   
    save  
    exit  

Now the ipv4 tunnel should be up&running

Check it with:

    show interfaces openvpn    
    show interfaces openvpn detail  
    show openvpn status site-to-site  

### Create IPv4 BGP Session

#### Open Firewall

* You need to open the firewall to local for the tunnel Interface on port 179/tcp

#### Configure the BGP Neighbor

* You must not use AS before the as numbers !!   

With this step you create the basic bgp session    
    configure  
    set protocols bgp 111111 neighbor Z.Z.Z.Z remote-as 222222  
    set protocols bgp 111111 neighbor Z.Z.Z.Z soft-reconfiguration inbound  
    set protocols bgp 111111 neighbor update-source 172.AA.AA.64  
    commit
    save

When commit this configuration you should be able to see a BGP neighbor session start and come up. You can check this with:

    show ip bgp summary

#### Set route to blackhole

so bgp can announce the route

    set protocols static route 172.AA.AA.64/27 blackhole  
    commit  
    save  

#### Announce prefix to BGP

    set protocols bgp 111111 network 172.A.A.64/27  
    commit  
    save  
    exit  

You should now be able to see networks being advertised via

    show ip bgp neighbors Z.Z.Z.Z advertised-routes




