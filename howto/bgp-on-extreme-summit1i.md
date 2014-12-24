# DN42 peering on Extreme Summit 1i
Here i'll show how to configure DN42 peering via BGP on an old Extreme Networks [Summit 1i](http://docs.google.com/viewer?url=https://www.mtmnet.com/PDF_FILES/summit1i.pdf) routing switch. This how-to should be also applicable to any other 'i'-series switch.

## Caveats
Looks like ExtremeWare doesn't support any tunneling mechanism in contrast to ExtremeWare IPv6 or ExtremeXOS operating systems. So you need either put your switch behind the router which will do tunneling with DN42 participant or directly connect the switch to our network, if that possible.

## Snipplet
This configuration was tested on latest EW of 7.8.4.1 patch1-r4 version. But it should work on most of older releases as well.

    ## DN42 should go both in internal (for clients) and external VLANs
    create vlan svlan
    configure vlan svlan ipaddress 192.168.1.100/24
    # Adding an alias
    enable multinetting standard
    configure vlan svlan add secondary-ip 172.22.251.2/23
    ...

    enable ipforwarding

    configure vlan svlan add subvlan ext
    ...

    # It is worth to filter alien nets
    create access-list deny_int ip destination any source 192.168.1.0/24 deny ports 2-16
    ...
    ##

    # Adding route to a neighbor
    configure iproute add 172.22.151.1/32 172.22.251.1

    configure bgp soft-reconfiguration
    configure bgp AS-number 65534
    configure bgp routerid 172.22.251.2
    enable bgp

Now, if you're trying EBGP with your peer:

    # Announce some networks
    configure bgp add network 172.22.151.0/23
    configure bgp add network aa.bb.cc.dd/32

    create bgp neighbor 172.22.151.1 remote-AS-number 65535
    # Point to a proper outgoing interface, useless in case when Super VLAN is used
    #configure bgp neighbor 172.22.151.1 source-interface vlan ext

    enable bgp neighbor 172.22.151.1

Or IBGP (local router does the EBGP in following example):

    # Don't wait for an EBGP
    disable bgp synchronization

    create bgp neighbor 192.168.1.1 remote-AS-number 65535
    enable bgp neighbor 192.168.1.1

Next, you may diagnose the things doing:

    show bgp
    show bgp neighbor
    show bgp neighbor 172.22.151.1 received-routes all
    show bgp neighbor 172.22.151.1 transmitted-routes all

After that ping and traceroute are your mates. It is worth to point switch to the DNS which knows .dn42 zone:

`configure dns-client add name-server 192.168.1.1`

And use names.