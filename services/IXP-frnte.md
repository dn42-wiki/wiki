The IXP frnte
=============

An IXP is a collection point for Internet providers. This can be physical or virtual. In a physical IXP, several Internet providers place servers in a data center and connect them to each other.

In a virtual IXP, the servers are not "real." They are not physically connected with cables, but for example via a VPN.

In dn42 almost all connections are virtual. One builds on the Internet and creates virtual links between the single nodes. In IXP frnte, all providers have virtual machines, which are connected to each other. Due to the large number of providers in IXP, it is possible to reach them easily and with low latency. However, the large number also leads to the fact that no direct peerings are established within an IXP, instead route servers are used. This receives and coordinates all routes of the providers and sends out appropriate routes. This way, many indirect peerings can be established.

History and origin
------------------

In dn42 and in the Anonet there was the UCIS IXP for a long time. However, this is no longer actively operated.

Members of the LGP Corp have now created a new IXP in dn42. This is the IXP frnte. It is located in France near Nantes and has two separate internet connections. This article describes how to enter the IXP and set up peering with the current route server.

Join the IXP
------------

### 1\. Request the infrastructure

LGP Corp provides virtual machines free of charge to any AS operator or anyone who wants to experiment with networks. There are no costs! The VM's can be configured and linked together as desired. The VM's can be connected to each other via a VLAN. Furthermore, an internet connection is available with two ISPs, depending on your choice. The virtual machine gets a public IPv6 and if necessary IPv4 over NAT to be able to access important resources like GitHub.  
It is best to create a diagram of your network and send it to the LGP Corp.  
The LGP Corp or the responsible admin for the IXP can be reached in **IRC** on hackint.org under **toinux**. Send the diagram to them and discuss further details.  
Furthermore, all virtual machines are put into a common VLAN. This causes that one can reach all providers at the IXP without problems.

### 2\. Proxmox Login and VM Setup

After that you will receive your access data for the Proxmox portal from the LGP Corp. Under which you can set up your VM's. The portal can be reached under [**https://pve.home.lgp-corp.fr/**](https://pve.home.lgp-corp.fr/). Select "Proxmox VE authentication server" as "Realm". It also offers a VNC monitor to work directly on the server. For the setup under SSH an IPv6 connectivity to the internet is required. If you only have an IPv4, you can get an IPv6 for free from Hurricane Electric at [https://tunnelbroker.net/](https://tunnelbroker.net/).

### 3\. Configure VLAN

An internal IPv6 Range has been requested for the IXP: `fde0:93fa:7a0:2::/64` ([fde0:93fa:7a0:2::/64 on explorer](https://explorer.dn42.dev/#/inet6num/fde0:93fa:7a0:2::_64))

The following is the assignment policy:  
`fde0:93fa:7a0:2:0:<asn32|high16|hex>:<asn32|low16|hex>:1/64`  
For example, if you have the ASN 4242421080, you get the range `fde0:93fa:7a0:2:0:fcde:3558:1/64`  
It should be noted that only the last block may be changed. So you get a practical IPv6 range of `fde0:93fa:7a0:2:0:fcde:3558:/112`.  
A Ruby script to calculate the IPv6 can be found on [ixp\_frnte\_dn42\_prefix.rb on GitHub Gist](https://gist.github.com/marek22k/494cf9c4d269867f23f2c3577e1780ef).

An example configuration for Debian based Linux distributions would be:  

    iface ensXX inet6 static
        address fde0:93fa:7a0:2:0:fcde:3558:1/64

Here `ensXX` is the dn42 VLAN interface. This can be determined by comparing the MAC address of the interface with the MAC address of the dn42 VLAN in Proxmox. The MAC address can be determined on Linux with `ip l`:

    ensXX: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu
        1500 qdisc pfifo_fast state UP mode DEFAULT group
        default qlen 1000
        link/ether MAC brd ff:ff:ff:ff:ff:ff

`MAC` would be the MAC address. After that you can activate the interface with ifup or a reboot of the VM.  
Of course there are other configuration possibilities. This is only an example for Debian-based Linux distributions.

### 4\. Connect to the Route Server

There can be several Route Servers (RS) on one IXP. However, on the IXP frnte there is currently only one, which is operated by jlu5 (operator of the highdef network).
IPv6: fde0:93fa:7a0:2:0:fcde:3559:1
ASN: 4242421081

You can now enter this configuration into your routing daemon and it will connect to the RS. You should keep in mind that the RS itself does not forward any traffic, but is only responsible for the coordination. Therefore the AS path must not necessarily start with the AS of the RS.

An example configuration for bird2 would be the following:

    protocol bgp ixp_rs from dnpeers {
        neighbor fde0:93fa:7a0:2:0:fcde:3559:1 as 4242421081;
    
        enable extended messages on;
    	direct;
    	enforce first as off;
    
    	ipv4 {
    	    extended next hop;
    	};
    }

**What does this configuration do?**

First we create a new BGP session (`protocol bgp`). This is based on the dnpeers template which can be found in the standard Bird2 configuration in the [wiki](https://dn42.eu/howto/Bird2). We name this session "ixp\_rs". However, this is only an internal name and can be replaced with another one.

After that we determine with whom we want to have the session. This would be the RS. Therefore we put IPv6 address and ASN there.

Furthermore, we allow larger BGP messages. Thus, instead of 4096 bytes, a whole 65535 bytes are transmitted in one message. This is especially useful because an RS has to announce a lot of routes.

With `direct` we indicate that the RS is directly connected to our server and no routing via third parties has to be performed. In our case, the RS is connected to us via the dn42 VLAN.

The next line has the effect that the ASN of the RS does not necessarily have to be the next hop for routing. This is important because we do not route the traffic via the RS, but via the respective peers. These have an ASN that differs from the ASN of the RS.

Since the dn42 VLAN _only_ supports IPv6, any IPv4 traffic must also go over IPv6. If you do not have or do not want to use IPv4, you can ignore this part of the configuration.

Finally we save the bird2 configuration and load the new configuration with `birdc configure`.

### 5\. Check if it works

There are now a few things to check:  
Once you can see if the BGP session is esablished. In Bird you can do this with `birdc show protocols all ixp_rs`.  
Furthermore, you can display different routes (in case of bird with `birdc show route for [ip address]`) or perform a traceroute.  
One can also try to ping the IP of some at the IXP. From the latency you can also see if everything is working:  

*   Burble's pingable
    *   172.20.129.5
    *   fd42:4242:2601:ac05::1
*   Bandura's pingable:
    *   172.22.149.224
    *   fd04:234e:fc31::
