Bird is a commonly used BGP daemon.  This page provides configuration and help to run Bird for dn42.
Compared to quagga, bird supports multiple routing tables, which is useful, if you also plan to peer with other federated networks such as freifunk. In the following a working configuration for dn42 is shown. If you 
want to learn the practical details behind routing protocols in bird, see the following [guide](https://github.com/knorrie/network-examples)

# Debian
In the Debian release cycle the bird packages may become outdated at times, if that is the case you should use the official bird package repository maintained by the developers of nic.cz.

This is not necessary for Debian Stretch, which currently ships the most recent version (1.6.3) in this repositories.

```sh
wget -O - http://bird.network.cz/debian/apt.key | apt-key add -
apt-get install lsb-release
echo "deb http://bird.network.cz/debian/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/bird.list
apt-get update
apt-get install bird
```

# Example configuration

Note: This file covers the configuration of Bird 1.x. For an example configuration of Bird 2.x see [howto/Bird2](/howto/Bird2)

* Replace `<AS>` with your Autonomous System Number (only the digits)
* Replace `<GATEWAY_IP>` with your gateway ip (the internal dn42 ip address you use on the host, where dn42 is running)
* Replace `<SUBNET>` with your registered dn42 subnet
* Replace `<PEER_IP>` with the ip of your peer who is connected with you using your favorite vpn protocol (openvpn, ipsec, tinc, ...)
* Replace `<PEER_AS>` the Autonomous System Number of your peer (only the digits)
* Replace `<PEER_NAME>` a self chosen name for your peer

### IPv6

```
#/etc/bird/bird6.conf
protocol device {
  scan time 10;
}

# local configuration
######################

include "/etc/bird/local6.conf";

# filter helpers
#################

##include "/etc/bird/filter6.conf";

# Kernel routing tables
########################


/*
    krt_prefsrc defines the source address for outgoing connections.
    On Linux, this causes the "src" attribute of a route to be set.
    
    Without this option outgoing connections would use the peering IP which
    would cause packet loss if some peering disconnects but the interface
    is still available. (The route would still exist and thus route through
    the TUN/TAP interface but the VPN daemon would simply drop the packet.)
*/
protocol kernel {
  scan time 20;
  import none;
  export filter {
    if source = RTS_STATIC then reject;
    krt_prefsrc = OWNIP;
    accept;
  };
}

# static routes
################

protocol static {
  route <SUBNET> reject;
  import all;
  export none;
}

template bgp dnpeers {
  local as OWNAS;
  path metric 1;
  import keep filtered;
  import filter {
    if is_valid_network() && !is_self_net() then {
      accept;
    }
    reject;
  };
  export filter {
    if is_valid_network() && source ~ [RTS_STATIC, RTS_BGP] then {
      accept;
    }
    reject;
  };
  import limit 1000 action block;
}

include "/etc/bird/peers6/*";
```

```
# /etc/bird/local6.conf
# should be a unique identifier, use same id as for ipv4
router id <GATEWAY_IP>;

define OWNAS =  <AS>;
define OWNIP = <GATEWAY_IP>;

function is_self_net() {
  return net ~ [<SUBNET>+];
}

function is_valid_network() {
  return net ~ [
    fd00::/8{44,64} # ULA address space as per RFC 4193
  ];
}
```

```
# /etc/bird/peers6/<PEER_NAME>
protocol bgp <PEER_NAME> from dnpeers {
  neighbor <PEERING_IP> as <PEER_AS>;
  # if you use link-local ipv6 addresses for peering using the following
  # neighbor <PEERING_IP> % '<INTERFACE_NAME>' as <PEER_AS>;
};
```

### IPv4

```
# /etc/bird/bird.conf
# Device status
protocol device {
  scan time 10; # recheck every 10 seconds
}

protocol static {
  # Static routes to announce your own range(s) in dn42
  route <SUBNET> reject;
  import all;
  export none;
};

# local configuration
######################

# keeping router specific in a seperate file, 
# so this configuration can be reused on multiple routers in your network
include "/etc/bird/local4.conf";

# filter helpers
#################

##include "/etc/bird/filter4.conf";

# Kernel routing tables
########################

/*
    krt_prefsrc defines the source address for outgoing connections.
    On Linux, this causes the "src" attribute of a route to be set.
    
    Without this option outgoing connections would use the peering IP which
    would cause packet loss if some peering disconnects but the interface
    is still available. (The route would still exist and thus route through
    the TUN/TAP interface but the VPN daemon would simply drop the packet.)
*/
protocol kernel {
  scan time 20;
  import none;
  export filter {    
    if source = RTS_STATIC then reject;
    krt_prefsrc = OWNIP;
    accept;
  };
};
# DN42
#######

template bgp dnpeers {
  local as OWNAS;
  # metric is the number of hops between us and the peer
  path metric 1;
  # this lines allows debugging filter rules
  # filtered routes can be looked up in birdc using the "show route filtered" command
  import keep filtered;
  import filter {
    # accept every subnet, except our own advertised subnet
    # filtering is important, because some guys try to advertise routes like 0.0.0.0
    if is_valid_network() && !is_self_net() then {
      accept;
    }
    reject;
  };
  export filter {
    # here we export the whole net
    if is_valid_network() && source ~ [RTS_STATIC, RTS_BGP] then {
      accept;
    }
    reject;
  };
  import limit 1000 action block;
  #source address OWNIP;
};

include "/etc/bird/peers4/*";
```

```
#/etc/bird/local4.conf
# should be a unique identifier, <GATEWAY_IP> is what most people use.
router id <GATEWAY_IP>;

define OWNAS =  <AS>;
define OWNIP = <GATEWAY_IP>;

function is_self_net() {
  return net ~ [<SUBNET>+];
}

function is_valid_network() {
  return net ~ [
    172.20.0.0/14{21,29}, # dn42
    172.20.0.0/24{28,32}, # dn42 Anycast
    172.21.0.0/24{28,32}, # dn42 Anycast
    172.22.0.0/24{28,32}, # dn42 Anycast
    172.23.0.0/24{28,32}, # dn42 Anycast
    172.31.0.0/16+,       # ChaosVPN
    10.100.0.0/14+,       # ChaosVPN
    10.127.0.0/16{16,32}, # neonetwork
    10.0.0.0/8{15,24}     # Freifunk.net
  ];
}
```

```
# /etc/bird/peers4/<PEER_NAME>
protocol bgp <PEER_NAME> from dnpeers {
  neighbor <PEERING_IP> as <PEER_AS>;
};
```

# Bird communities

Communities can be used to prioritize traffic based on different flags, in DN42 we are using communities to prioritize based on latency, bandwidth and encryption. It is really easy to get started with communities and we encourage all of you to get the basic configuration done and to mark your peerings with the correct flags for improved routing.
More information can be found [here](/howto/Bird-communities). 

# Route Origin Authorization

Route Origin Authorizations should be used in BIRD to authenticate prefix announcements. These check the originating AS and validate that they are allowed to advertise a prefix. 

## ROA Tables

The ROA table can be generated from the registry directly or you can use the following pre-built ROA tables for BIRD (generated every 30 mins from the registry):

ROA files generated by [dn42regsrv](https://git.dn42.dev/burble/dn42regsrv) are available from burble.dn42:

|URL|&nbsp;IPv4/IPv6&nbsp;|Description|                                                                                      
|---|---|---|                                                                                                                
|[https://dn42.burble.com/roa/dn42_roa_46.json](https://dn42.burble.com/roa/dn42_roa_46.json) &nbsp; | &nbsp;Both&nbsp; | JSON format for use with RPKI |                                                                                          
|[https://dn42.burble.com/roa/dn42_roa_bird1_46.conf](https://dn42.burble.com/roa/dn42_roa_bird1_46.conf) &nbsp; | &nbsp;Both&nbsp; | Bird1 format |                                                                          
|[https://dn42.burble.com/roa/dn42_roa_bird1_4.conf](https://dn42.burble.com/roa/dn42_roa_bird1_4.conf) &nbsp; | &nbsp;IPv4 Only&nbsp; | Bird1 format |                                                                       
|[https://dn42.burble.com/roa/dn42_roa_bird1_6.conf](https://dn42.burble.com/roa/dn42_roa_bird1_6.conf) &nbsp; | &nbsp;IPv6 Only&nbsp; | Bird1 format |                                                                       
|[https://dn42.burble.com/roa/dn42_roa_bird2_46.conf](https://dn42.burble.com/roa/dn42_roa_bird2_46.conf) &nbsp; | &nbsp;Both&nbsp; | Bird2 format |                                                                          
|[https://dn42.burble.com/roa/dn42_roa_bird2_4.conf](https://dn42.burble.com/roa/dn42_roa_bird2_4.conf) &nbsp; | &nbsp;IPv4 Only&nbsp; | Bird2 format |                                                                      
|[https://dn42.burble.com/roa/dn42_roa_bird2_6.conf](https://dn42.burble.com/roa/dn42_roa_bird2_6.conf) &nbsp; | &nbsp;IPv6 Only&nbsp; | Bird2 format |                                                                      

### Updating ROA tables

You can add cron entries to periodically update the tables:

```
*/15 * * * * curl -sfSLR {-o,-z}/var/lib/bird/bird6_roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird1_6.conf && chronic birdc6 configure
*/15 * * * * curl -sfSLR {-o,-z}/var/lib/bird/bird_roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird1_4.conf && chronic birdc configure
```

Debian version:

```
*/15 * * * * curl -sfSLR -o/var/lib/bird/bird6_roa_dn42.conf -z/var/lib/bird/bird6_roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird1_6.conf && /usr/sbin/birdc6 configure
*/15 * * * * curl -sfSLR -o/var/lib/bird/bird_roa_dn42.conf -z/var/lib/bird/bird_roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird1_4.conf && /usr/sbin/birdc configure
```

then create the directory to make sure curls can save the files:

```
mkdir -p /var/lib/bird/
```

# Use RPKI ROA for bird2
* Download  gortr

`https://github.com/cloudflare/gortr/releases`

* Running gortr
`./gortr -verify=false -checktime=false -cache=https://dn42.burble.com/roa/dn42_roa_46.json`

* Add this your bird configure file,other ROA protocol must remove


```
protocol rpki rpki_dn42{
  roa4 { table dn42_roa; };
  roa6 { table dn42_roa_v6; };

  remote "<your rpki server ip or domain>" port 8282;

  retry keep 90;
  refresh keep 900;
  expire keep 172800;
}
```

## Filter configuration

In your import filter add the following to reject invalid routes:

```
if (roa_check(dn42_roa, net, bgp_path.last) != ROA_VALID) then {
   print "[dn42] ROA check failed for ", net, " ASN ", bgp_path.last;
   reject;
}
```

Also, define your ROA table with:

```
roa table dn42_roa {
    include "/var/lib/bird/bird_roa_dn42.conf";
};
```


**NOTE**: Make sure you setup ROA checks for both bird and bird6 (for IPv6).

# Useful bird commmands

bird can be remote controlled via the `birdc` command. Here is a list of useful bird commands:

```
$ birdc
BIRD 1.4.5 ready.
bird> configure # reload configuration
Reading configuration from /etc/bird.conf
Reconfigured
bird> show  ? # Completions work either by pressing tab or pressing '?'
show bfd ...                                   Show information about BFD protocol
show interfaces                                Show network interfaces
show memory                                    Show memory usage
show ospf ...                                  Show information about OSPF protocol
show protocols [<protocol> | "<pattern>"]      Show routing protocols
show roa ...                                   Show ROA table
show route ...                                 Show routing table
show static [<name>]                           Show details of static protocol
show status                                    Show router status
show symbols ...                               Show all known symbolic names
bird> show protocols # this command shows your peering status
name     proto    table    state  since       info
device1  Device   master   up     07:20:25    
kernel1  Kernel   master   up     07:20:25    
chelnok  BGP      master   up     07:20:29    Established   
hax404   BGP      master   up     07:20:26    Established     
static1  Static   master   up     07:20:25
bird> show protocols all chelnok # show verbose peering status for peering with chelnok
bird> show route for 172.22.141.181 # show possible routes to internal.dn42
172.22.141.0/24    via 172.23.67.1 on tobee [tobee 07:20:30] * (100) [AS64737i]
                   via 172.23.64.1 on chelnok [chelnok 07:20:29] (100) [AS64737i]
                   via 172.23.136.65 on hax404 [hax404 07:20:26] (100) [AS64737i]
bird> show route filtered # shows routed filtered out by rules
172.23.245.1/32    via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76175i]
172.22.247.128/32  via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76175i]
172.22.227.1/32    via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76115i]
172.23.196.75/32   via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76115i]
172.22.41.241/32   via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76115i]
172.22.249.4/30    via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS4242420002i]
172.22.255.133/32  via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS64654i]
bird> show route protocol <somepeer> # shows the route they export to you
bird> show route export <somepeer> # shows the route you export to someone
...
```

# External Links
* detailed bird configuration from Mic92: https://github.com/Mic92/bird-dn42
* more bird commands: https://bird.network.cz/?get_doc&v=20&f=bird-4.html