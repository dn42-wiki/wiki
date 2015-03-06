Bird is a commonly used BGP daemon.  This page provides configuration and help to run Bird for dn42.
Compared to quagga, bird supports multiple routing, which is useful, if you also plan to peer with other federated networks such as freifunk.

# Example configuration

* Replace `<AS>` with your Autonomous System Number
* Replace `<GATEWAY_IP>` with your gateway ip (the internal dn42 ip address you use on the host, where dn42 is running)
* Replace `<SUBNET>` with your registered dn42 subnet, which you allocated on [nixnodes](https://io.nixnodes.net/)
* Replace `<PEER_IP>` with the ip of your peer who is connected with you using your favorite vpn protocol (openvpn, ipsec, tinc, ...)
* Replace `<PEER_AS>` the Autonomous System Number of your peer
* Replace `<PEER_NAME>` a self chosen name for your peer

### IPV4

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
$ cd net.dn42.registry
$ ruby utils/bgp-filter.rb < data/filter.txt > /etc/bird/filter4.conf

or

$ curl -sk https://sour.is/git/dn42/registry/plain/data/filter.txt | \
  awk 'BEGIN {printf "function is_valid_network() {\n return net ~ [\n" } \
       /^[0-9]/ && $2 ~ /permit/ {printf " %s{%s,%s},\n", $3, $4, $5};' | \
  sed "$ s/,$/\n ];\n}/" > /etc/bird/filter4.conf

```

example filter list:

```
# /etc/bird/filter4.conf
function is_valid_network() {
  return net ~ [
    172.22.0.0/15{22,28}, # dn42 main net0
    172.22.0.0/23{23,32}, # dn42 Anycast
    172.22.0.43/32{32,32}, # Whois Anycast
    172.22.0.53/32{32,32}, # DNS Anycast
    172.22.0.94/32{32,32}, # TOR Anycast
    172.23.0.0/24{24,32}, # dn42 Anycast range
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

### IPV6

```
#/etc/bird/bird6.conf
protocol device {
  scan time 10;
}

# filter helpers
#################

include "/etc/bird/filter6.conf";

# local configuration
######################

include "bird/local6.conf";

# Kernel routing tables
########################

protocol kernel {
  scan time 20;
  device routes;
  import none;
  export filter {
    krt_prefsrc = OWNIP;
    accept;
  };
}

# static routes
################

protocol static {
  route <SUBNET> reject;
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
    if is_valid_network() then {
      accept;
    }
    reject;
  };
  route limit 10000;
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
```

Generate the filter list from the monotone repository

```
$ cd net.dn42.registry
$ ruby utils/bgp-filter.rb < data/filter6.txt > /etc/bird/filter6.conf

or

$ curl -sk https://sour.is/git/dn42/registry/plain/data/filter6.txt | \
  awk 'BEGIN {printf "function is_valid_network() {\n return net ~ [\n" } \
       /^[0-9]/ && $2 ~ /permit/ {printf " %s{%s,%s},\n", $3, $4, $5};' | \
  sed "$ s/,$/\n ];\n}/" > /etc/bird/filter6.conf

```

example filter list:

```
/etc/bird/filter6.conf
function is_valid_network() {
  return net ~ [
    fc00::/8{48,64}, # ULA (undefined)
    fd00::/8{48,64}, # ULA (defined)
    2001:67c:20c1::/48{48,48}, # E-UTP IPv6
    2001:bf7::/32{32,128}, # Freifunk (Foerderverein Freie Netzwerke) IPv6 Range
    2001:67c:20a1::/48{48,48}, # CCC Event Network
    2001:0470:006c:01d5::/64{64,64}, # Registered IANA
    2001:0470:006d:0655::/64{64,64},
    2001:0470:1f09:172d::/64{64,64},
    2001:0470:1f0b:0592::/64{64,64},
    2001:0470:1f0b:0bca::/64{64,64},
    2001:0470:1f0b:1af5::/64{64,64},
    2001:0470:1f10:0275::/64{64,64},
    2001:0470:1f12:0004::/64{64,64},
    2001:0470:5084::/48{48,64},
    2001:0470:51c6::/48{48,64},
    2001:0470:73d3::/48{48,64},
    2001:0470:7972::/48{48,64},
    2001:0470:9949::/48{48,64},
    2001:0470:99fc::/48{48,64},
    2001:0470:9af8::/48{48,64},
    2001:0470:9ce6::/55{55,64},
    2001:0470:9f43::/48{48,64},
    2001:0470:caab::/48{48,64},
    2001:0470:cd99::/48{48,64},
    2001:0470:d4df::/48{48,64},
    2001:0470:d889:0010::/64{64,64},
    2001:0470:e3f0:000a::/64{64,64},
    2001:067c:21ec::/48{48,64},
    2001:06f8:1019:0000::/64{64,64},
    2001:06f8:118b::/48{48,64},
    2001:06f8:1194::/48{48,64},
    2001:06f8:121a::/48{48,64},
    2001:06f8:1c1b::/48{48,64},
    2001:06f8:1d14::/48{48,64},
    2001:06f8:1d26::/48{48,64},
    2001:06f8:1d53::/48{48,64},
    2001:07f0:3003::/48{48,64},
    2001:08d8:0081:05c8::/63{63,64},
    2001:08d8:0081:05ca::/64{64,64},
    2001:15c0:1000:0100::/64{64,64},
    2001:1b60:1000:0001::/64{64,64},
    2001:41d0:0001:b6bb::/64{64,64},
    2001:41d0:0001:cd42::/64{64,64},
    2001:4dd0:fcff::/48{48,64},
    2001:4dd0:fdd3::/48{48,64},
    2001:4dd0:ff00:8710::/64{64,64},
    2604:8800:0179:4200::/56{56,64},
    2801:0000:80:8000::/50{50,64},
    2a00:1328:e101:0200::/56{56,64},
    2a00:1828:2000:0289::/64{64,64},
    2a00:1828:a013:d242::/64{64,64},
    2a00:5540:0387::/48{48,64},
    2a01:0198:022c::/48{48,64},
    2a01:0198:035a:fd13::/64{64,64},
    2a01:0198:0485::/48{48,64},
    2a01:04f8:0121:4fff::/64{64,64},
    2a01:04f8:0140:1ffd::/64{64,64},
    2a01:04f8:0d13:17c0::/64{64,64},
    2a02:0a00:e010:3c00::/56{56,64},
    2a02:0ee0:0002:0051::/64{64,64},
    2a03:2260::/30{30,64}
  ];
}
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
bird> show route filtered
172.23.245.1/32    via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76175i]
172.22.247.128/32  via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76175i]
172.22.227.1/32    via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76115i]
172.23.196.75/32   via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76115i]
172.22.41.241/32   via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS76115i]
172.22.249.4/30    via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS4242420002i]
172.22.255.133/32  via 172.23.64.1 on chelnok [chelnok 21:26:18] * (100) [AS64654i]
...
```

# External Links
* more bgp commands: http://danrimal.net/doku.php?id=wiki:bgp:bird:postupy
