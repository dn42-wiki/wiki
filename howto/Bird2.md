This guide is similar to the normal [Bird](/howto/Bird) guide in that it provides you with help setting up the BIRD routing daemon, with the difference that this page is dedicated to versions 2.x.

# Arch Linux

The `extra/bird` package in the arch repositories will usually have a relatively recent version and there is (usually) no need for a manual install over the usual `# pacman -S bird`.

# Example configuration

Please note: This example configuration is made for use with IPv4 and IPv6 (Really, there is no excuse not to get started with IPv6 networking! :) )

The default config location in bird version 2.x is `/etc/bird.conf`, but this may vary depending on how your distribution compiled bird.

When copying the configuration below onto your system, you will have to enter the following values in the file header:

* Replace `<OWNAS>` with your autonomous system number, e.g. `4242421234`
* Replace `<OWNIP>` with the ip that your router is going to have, this is usually the first non-zero ip in your subnet. (E.g. x.x.x.65 in an x.x.x.64/28 network)
* Similarly, replace `<OWNIPv6>` with the first non-zero ip in your ipv6 subnet.
* Then replace `<OWNNET>` with the IPv4 subnet that was assigned to you.
* The same goes for `<OWNNETv6>`, but it takes an IPv6 subnet (Who'd have thought).
* Keep in mind that you'll have to enter both networks in the OWNNET{,v6} and OWNNETSET{,v6}, the two variables are required due to set parsing difficulties with variables.

````
################################################
#               Variable header                #
################################################

define OWNAS =  <OWNAS>;
define OWNIP =  <OWNIP>;
define OWNIPv6 = <OWNIPv6>;
define OWNNET = <OWNNET>;
define OWNNETv6 = <OWNNETv6>;
define OWNNETSET = [<OWNNET>+];
define OWNNETSETv6 = [<OWNNETv6>+];

################################################
#                 Header end                   #
################################################

router id OWNIP;

protocol device {
    scan time 10;
}

/*
 *  Utility functions
 */

function is_self_net() {
  return net ~ OWNNETSET;
}

function is_self_net_v6() {
  return net ~ OWNNETSETv6;
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

roa4 table dn42_roa;
roa6 table dn42_roa_v6;

protocol static {
    roa4 { table dn42_roa; };
    include "/etc/bird/roa_dn42.conf";
};

protocol static {
    roa6 { table dn42_roa_v6; };
    include "/etc/bird/roa_dn42_v6.conf";
};

function is_valid_network_v6() {
  return net ~ [
    fd00::/8{44,64} # ULA address space as per RFC 4193
  ];
}

protocol kernel {
    scan time 20;
    
    ipv6 {
        import none;
        export filter {
            if source = RTS_STATIC then reject;
            krt_prefsrc = OWNIPv6;
            accept;
        };
    };
};

protocol kernel {
    scan time 20;

    ipv4 {
        import none;
        export filter {
            if source = RTS_STATIC then reject;
            krt_prefsrc = OWNIP;
            accept;
        };
    };
}

protocol static {
    route OWNNET reject;

    ipv4 {
        import all;
        export none;
    };
}

protocol static {
    route OWNNETv6 reject;

    ipv6 {
        import all;
        export none;
    };
}

template bgp dnpeers {
    local as OWNAS;
    path metric 1;
    
    ipv4 {
        import filter {
          if is_valid_network() && !is_self_net() then {
            if (roa_check(dn42_roa, net, bgp_path.last) != ROA_VALID) then {
              print "[dn42] ROA check failed for ", net, " ASN ", bgp_path.last;
              reject;
            } else accept;
          } else reject;
        };

        export filter { if is_valid_network() && source ~ [RTS_STATIC, RTS_BGP] then accept; else reject; };
        import limit 1000 action block;
    };

    ipv6 {
        import filter {
          if is_valid_network_v6() && !is_self_net_v6() then {
            if (roa_check(dn42_roa_v6, net, bgp_path.last) != ROA_VALID) then {
              print "[dn42] ROA check failed for ", net, " ASN ", bgp_path.last;
              reject;
            } else accept;
          } else reject;
        };
        export filter { if is_valid_network_v6() && source ~ [RTS_STATIC, RTS_BGP] then accept; else reject; };
        import limit 1000 action block; 
    };
}


include "/etc/bird/peers/*";
````

# Route Origin Authorization

The example config above relies on ROA configuration files in `/etc/bird/roa_dn42{,_v6}.conf`. These should be automatically downloaded and updated every so often to prevent BGP highjacking, [see the bird1 page](/howto/Bird#route-origin-authorization) for more details and links to the ROA files. 

# Setting up peers

Please note: This section assumes that you've already got a tunnel to your peering partner setup.

First, make sure the /etc/bird/peers directory exists:

````
# mkdir -p /etc/bird/peers
````

Then for each peer, create a configuration file similar to this one:

`/etc/bird/peers/<NEIGHBOR_NAME>.conf`:

````
protocol bgp <NEIGHBOR_NAME> from dnpeers {
        neighbor <NEIGHBOR_IP> as <NEIGHBOR_ASN>;
}

protocol bgp <NEIGHBOR_NAME>_v6 from dnpeers {
        neighbor <NEIGHBOR_IPv6>%<NEIGHBOR_INTERFACE> as <NEIGHBOR_ASN>;
}
````

Due to the special link local addresses of IPv6, an interface has to be specified using the %<if> syntax if a link local address is used (Which is recommended)