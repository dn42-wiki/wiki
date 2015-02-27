Bird is a commonly used BGP daemon.  This page provides configuration and help to run Bird for dn42.
Compared to quagga, bird supports multiple routing, which is useful, if you also plan to peer with other federated networks such as freifunk.

# Example configuration

* Replace `<AS>` with your Autonomous System Number
* Replace `<GATEWAY_IP>` with your gateway ip (the internal dn42 ip address you use on the host, where dn42 is running)
* Replace `<SUBNET>` with your registered dn42 subnet, which you allocated on [nixnodes](https://io.nixnodes.net/)
* Replace `<PEER_IP>` with the ip of your peer who is connected with you using your favorite vpn protocol (openvpn, ipsec, tinc, ...)
* Replace `<PEER_AS>` the Autonomous System Number of your peer
* Replace `<PEER_NAME>` a self chosen name for your peer

```
# /etc/bird/bird.conf
# Device status
protocol device {
  scan time 10; # recheck every 10 seconds
}

protocol static {
  # Static routes to announce your own range(s) in dn42
  route <SUBNET> reject;
};

# filter helpers
#################

include "/etc/bird/filter4.conf";

# local configuration
######################

# keeping router specific in a seperate file, 
# so this configuration can be reused on multiple routers in your network
include "/etc/bird/local4.conf";

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
  device routes;
  import none;
  export filter {
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
    if is_valid_network() then {
      accept;
    }
    reject;
  };
  route limit 10000;
  source address OWNIP;
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
```

Generate the filter list from the monotone repository

```
cd net.dn42.registry
ruby utils/bgp-filter.rb < data/filter.txt > /etc/bird/filter4.conf
```

example filter list:

```
# /etc/bird/filter4.conf
function is_valid_network() {
  return net ~ [
    172.22.0.0/15{22,28}, # dn42 main net0
    172.22.0.43/32{32,32}, # Whois Anycast
    172.22.0.53/32{32,32}, # DNS Anycast
    172.22.0.94/32{32,32}, # TOR Anycast
    192.175.48.0/24{24,32}, # AS112-prefix for reverse-dns
    10.0.0.0/8{12,28}, # freifunk/chaosvpn
    172.31.0.0/16{22,28}, # chaosvpn
    100.64.0.0/10{12,28}, # iana private range
    195.160.168.0/23{23,28}, # ctdo
    91.204.4.0/22{22,28}, # free.de via ctdo
    193.43.220.0/23{23,28}, # durchdieluft via ctdo
    83.133.178.0/23{23,28}, # muccc kapsel
    87.106.29.254/32{32,32}, # wintix (please don' announce /32)
    85.25.246.16/28{28,32}, # leon
    46.4.248.192/27{27,32}, # welterde
    94.45.224.0/19{19,28}, # ccc event network
    151.217.0.0/16{16,28}, # ccc event network 2
    195.191.196.0/23{23,29}, # ichdasich pi space
    80.244.241.224/27{27,32}, # jchome service network
    188.40.34.241/32{32,32},
    37.1.89.192/26{26,28}, # siska
    87.98.246.19/32{32,32}
  ];
}
```

```
# /etc/bird/peers4/<PEER_NAME>
protocol bgp <PEER_NAME> from dnpeers {
  neighbor <PEERING_IP> as <PEER_AS>;
};
```

# Useful bird commmands

bird can be remote controlled via the `birdc` command. Here is a list of useful bird commands:

```
$ birdc
BIRD 1.4.5 ready.
bird> reload all # reload configuration
kernel1: reloading
chelnok: reloading
hax404: reloading
static1: reload failed
bird> show protocols # this command shows your peering status
name     proto    table    state  since       info
device1  Device   master   up     07:20:25    
kernel1  Kernel   master   up     07:20:25    
chelnok  BGP      master   up     07:20:29    Established   
hax404   BGP      master   up     07:20:26    Established     
static1  Static   master   up     07:20:25
bird> show route for 172.22.141.181 # show possible routes to internal.dn42
172.22.141.0/24    via 172.23.67.1 on tobee [tobee 07:20:30] * (100) [AS64737i]
                   via 172.23.64.1 on chelnok [chelnok 07:20:29] (100) [AS64737i]
                   via 172.23.136.65 on hax404 [hax404 07:20:26] (100) [AS64737i]
```

# External Links
* more bgp commands: http://danrimal.net/doku.php?id=wiki:bgp:bird:postupy
