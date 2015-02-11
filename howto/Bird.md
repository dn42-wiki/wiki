Bird is a commonly used BGP daemon.  This page provides configuration and help to run Bird for dn42.

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

function is_freifunk() {
  return net ~ [ 10.0.0.0/8+ ];
}

function is_dn42()     {
  return net ~ [
    37.1.89.160/29+,      # siska
    46.4.248.192/27+,     # welterde
    46.19.90.48/28+,      # planet cyborg
    46.19.90.96/28+,      # planet cyborg
    80.244.241.224/27+,   # jchome service network
    85.25.246.16/28+,     # Leon Weber
    87.106.29.254/32,     # wintix
    91.204.4.0/22+,       # free.de via ctdo
    94.45.224.0/19+,      # ccc event network
    172.22.0.53/32,       # dns
    172.22.0.0/15{15,30}, # official subnet for dn42
    172.23.0.0/16{15,30}, # official subnet for dn42
    178.33.32.123/32,     # Martin89
    178.63.170.40/32,     # jomat
    188.40.34.241/32,     # jomat
    192.175.48.0/24+,     # AS112-prefix for reverse-dns
    193.43.220.0/23+,     # durchdieluft via ctdo
    195.16.84.40/29+,     # siska
    195.160.168.0/23+,    # ctdo
    195.191.196.0/23+     # ichdasich pi-space
  ];
}

function is_chaosvpn() {
  return net ~ [
    10.4.0.0/16+,        # Allocated for ChaosVPN. Ready for distribution, currently not used
    10.32.0.0/16+,       # Allocated for ChaosVPN. Ready for distribution, currently not used
    10.42.16.0/20+,      # legacy
    10.100.0.0/14+,      # us hackerspaces range
    10.104.0.0/14+,      # Warzone, currently not used
    172.31.0.0/16+,      # In use by European hackerspaces
    83.133.178.0/23+,    # kapsel - CCC Munich
    172.26.0.0/15+,      # KBU Freifunk
    176.9.52.58/32+,     # haegar_vlad
    178.33.2.240/28+,    # o_g
    193.103.159.0/24+,   # haegar_vlad
    193.103.160.0/23+,   # haegar_vlad
    212.12.50.208/29+,   # ccchh
    213.238.61.128/26+   # mc.fly
  ];
}

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
    if (is_dn42() || is_freifunk() || is_chaosvpn()) && !is_self_net() then {
      accept;
    }
    reject;
  };
  export filter {
    # here we export the hole net
    if is_dn42() || is_freifunk() || is_chaosvpn() then {
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
router id 172.23.75.1;

define OWNAS =  <AS>;
define OWNIP = <GATEWAY_IP>;

function is_self_net() {
  return net ~ [<SUBNET>+];
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
