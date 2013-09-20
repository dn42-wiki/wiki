# DN42 peering on Extreme Summit 5i
Here i'll show how to configure DN42 peering via BGP on an old Extreme Networks [Summit 5i](http://docs.google.com/viewer?url=andovercg.com/datasheets/summit-5i-switches.pdf) routing switch. This how-to should be also applicable to any other 'i'-series switch. Other configuration info may be found [here](https://bitbucket.org/dukzcry/hobbies/src).

## Caveats
Looks like ExtremeWare doesn't support any tunneling mechanism in contrast to ExtremeWare IPv6 or ExtremeXOS operating systems. So you need either put your switch behind the router which will do tunneling with DN42 participant or directly connect the switch to our network, if that possible.

## Snipplet
This configuration was tested on latest EW of 7.8.4.1 patch1-r4 version. But it should work on most of older releases as well.

    enable ipforwarding

    # Adding an alias
    enable multinetting standard
    configure vlan ext add secondary-ip 172.23.150.2/24

    # Adding route to a neighbor
    configure iproute add 172.23.11.1/32 172.23.150.1

    configure bgp soft-reconfiguration
    configure bgp AS-number 64528
    configure bgp routerid 172.23.150.2
    enable bgp

Now, if you're trying EBGP with your peer:

    # Announce some networks
    configure bgp add network 172.23.150.0/25
    configure bgp add network 77.37.212.15/32

    create bgp neighbor 172.23.11.1 remote-AS-number 64526
    # Point to a proper outgoing interface
    configure bgp neighbor 172.23.11.1 source-interface vlan ext

    enable bgp neighbor 172.23.11.1

Or IBGP (local router does the EBGP in following example):

# Don't wait for an EBGP
    disable bgp synchronization

    create bgp neighbor 192.168.1.1 remote-AS-number 64528
    enable bgp neighbor 192.168.1.1

Next, you may diagnose the things doing:

    show bgp
    show bgp neighbor
    show bgp neighbor 172.23.11.1 received-routes all
    show bgp neighbor 172.23.11.1 transmitted-routes all

After that ping and traceroute are your mates. It is worth to point switch to the DNS which knows .dn42 zone:

`configure dns-client add name-server 192.168.1.1`

And use names.
## TO-DO
Check 'Switch acts like router in multinetting case' setup.