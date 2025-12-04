# Installation notes
This page is applicable to bird versions 2.x
## Arch Linux

The `extra/bird` package in the arch repositories will usually have a relatively recent version and there is (usually) no need for a manual install over the usual `# pacman -S bird`.

## Bird2 Version <2.0.8 / Debian

Please note, that Bird2 versions before 2.0.8 don't support IPv6 extended nexthops for IPv4 destinations (<https://bird.network.cz/pipermail/bird-users/2020-April/014412.html>).
Additionally Bird2 before 2.0.8 cannot automatically update filtered bgp routes when a used RPKI source changes.

Debian 11 Bullseye delivers Bird 2.0.7. But you can use the Debian Bullseye backport-repository which provides version 2.0.8 (see <https://backports.debian.org/Instructions/> for adding backports repository and install packages from the repository).

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

```conf
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

function is_self_net() -> bool {
  return net ~ OWNNETSET;
}

function is_self_net_v6() -> bool {
  return net ~ OWNNETSETv6;
}

function is_valid_network() -> bool {
  return net ~ [
    172.20.0.0/14{21,29}, # dn42
    172.20.0.0/24{28,32}, # dn42 Anycast
    172.21.0.0/24{28,32}, # dn42 Anycast
    172.22.0.0/24{28,32}, # dn42 Anycast
    172.23.0.0/24{28,32}, # dn42 Anycast
    172.31.0.0/16+,       # ChaosVPN
    10.100.0.0/14+,       # ChaosVPN
    10.127.0.0/16+,       # neonetwork
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

function is_valid_network_v6() -> bool {
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
              # Reject when unknown or invalid according to ROA
              print "[dn42] ROA check failed for ", net, " ASN ", bgp_path.last;
              reject;
            } else accept;
          } else reject;
        };

        export filter { if is_valid_network() && source ~ [RTS_STATIC, RTS_BGP] then accept; else reject; };
        import limit 9000 action block;
    };

    ipv6 {   
        import filter {
          if is_valid_network_v6() && !is_self_net_v6() then {
            if (roa_check(dn42_roa_v6, net, bgp_path.last) != ROA_VALID) then {
              # Reject when unknown or invalid according to ROA
              print "[dn42] ROA check failed for ", net, " ASN ", bgp_path.last;
              reject;
            } else accept;
          } else reject;
        };
        export filter { if is_valid_network_v6() && source ~ [RTS_STATIC, RTS_BGP] then accept; else reject; };
        import limit 9000 action block; 
    };
}


include "/etc/bird/peers/*";
```

# Setting up peers

Please note: This section assumes that you've already got a tunnel to your peering partner set up.

First, make sure the /etc/bird/peers directory exists:

```sh
# mkdir -p /etc/bird/peers
```

Each peer can use different methods to peer. Most usually this is either two seperate sessions,
one for ipv4 and one for ipv6, or  Multi protocol BGP with Extended Next Hop, as detailed below. 

`/etc/bird/peers/<NEIGHBOR>.conf`:
For the case with seperate BGP sessions
```conf
protocol bgp <NEIGHBOR_NAME> from dnpeers {
        neighbor <NEIGHBOR_IP> as <NEIGHBOR_ASN>;
}

protocol bgp <NEIGHBOR_NAME>_v6 from dnpeers {
        neighbor <NEIGHBOR_IPv6>%<NEIGHBOR_INTERFACE> as <NEIGHBOR_ASN>;
        # Or:
        # neighbor <NEIGHBOR_IPv6> as <NEIGHBOR_ASN>;
        # interface <NEIGHBOR_INTERFACE>;****
}
```
And for the case of MP-BGP over IPV6 with ENH
This work without a configured IPV4 in the link interface
```conf
protocol bgp <NEIGHBOR_NAME> from dnpeers {
    enable extended messages on;
    neighbor <NEIGHBOR_IPv6>%<NEIGHBOR_INTERFACE> as <NEIGHBOR_ASN>;
        # Or:
        # neighbor <NEIGHBOR_IPv6> as <NEIGHBOR_ASN>;
        # interface <NEIGHBOR_INTERFACE>;****
     ipv4 {
        extended next hop on;
    };
};
```

Due to the special link local addresses of IPv6, an interface has to be specified using the `%<if>` or the `interface <if>;` syntax if a link local address is used (Which is recommended)

# Getting routes installed to the kernel automatically

If you do not do this, your bird logs *will* be full of `Netlink: Invalid argument` errors and you will not be able to access the DN42 network.

You will need to create a loopback dummy interface with your DN42 router IP, for both IPv4 and IPv6

Create the dummy interface with these commands:

```
ip link add dn42-dummy type dummy
ip link set dev dn42-dummy up
```

Then, give it your router IP addresses:

```
ip addr add dev dn42-dummy <router ip>/<subnet>
```

Do this for both IPv4 and IPv6.

## Note on multiprotocol BGP and extended next hops
This configuration example shows the required configuration without using multiprotocol BGP and extended next hops.
These two options are helpful if one desires to optimize their peering by reducing the session count per peer to 1 (in the case of multiprotocol BGP) and remove the need to have IPv4 tunnel IP addresses (in the case of Extended next hops over IPv6)

# BGP communities

Communities can be used to prioritize traffic based on different flags, in DN42 we are using communities to prioritize based on latency, bandwidth and encryption. It is really easy to get started with communities and we encourage all of you to get the basic configuration done and to mark your peerings with the correct flags for improved routing.
More information can be found [here](/howto/BGP-communities). 

# Route Origin Authorization

Route Origin Authorizations should be used in BIRD to authenticate prefix announcements. These check the originating AS and validate that they are allowed to advertise a prefix. 

## RPKI / RTR for ROA

To use an RTR server for ROA information, replace this config in your bird2 configuration file:

```conf
protocol static {
    roa4 { table dn42_roa; };
    include "/etc/bird/roa_dn42.conf";
};

protocol static {
    roa6 { table dn42_roa_v6; };
    include "/etc/bird/roa_dn42_v6.conf";
};
```

... with this one (by changing address and port so it points to your RTR server)

```conf
protocol rpki roa_dn42 {
        roa4 { table dn42_roa; };
        roa6 { table dn42_roa_v6; };
        remote 10.1.3.3;
        port 323;
        refresh 600;
        retry 300;
        expire 7200;
}
```
To reflect changes in the ROA table without a manual reload, **ADD** "import table" switch for both channels in your DN42 BGP template:

```conf
template bgp dnpeers {
  ipv4 {
    ...existing configuration
    import table;
  };
  ipv6 {
    ...existing configuration
    import table;
  };
}
```

## ROA Tables

The ROA table can be generated from the registry directly or you can use the following pre-built ROA tables for BIRD:

ROA files generated by [dn42regsrv](https://git.burble.com/burble.dn42/dn42regsrv) are available from burble.dn42:

|URL|&nbsp;IPv4/IPv6&nbsp;|Description|
|---|---|---|
| <https://dn42.burble.com/roa/dn42_roa_46.json> &nbsp; | &nbsp;Both&nbsp; | JSON format for use with RPKI |
| <https://dn42.burble.com/roa/dn42_roa_bird1_46.conf> &nbsp; | &nbsp;Both&nbsp; | Bird1 format |
| <https://dn42.burble.com/roa/dn42_roa_bird1_4.conf> &nbsp; | &nbsp;IPv4 Only&nbsp; | Bird1 format |
| <https://dn42.burble.com/roa/dn42_roa_bird1_6.conf> &nbsp; | &nbsp;IPv6 Only&nbsp; | Bird1 format |
| <https://dn42.burble.com/roa/dn42_roa_bird2_46.conf> &nbsp; | &nbsp;Both&nbsp; | Bird2 format |
| <https://dn42.burble.com/roa/dn42_roa_bird2_4.conf> &nbsp; | &nbsp;IPv4 Only&nbsp; | Bird2 format |
| <https://dn42.burble.com/roa/dn42_roa_bird2_6.conf> &nbsp; | &nbsp;IPv6 Only&nbsp; | Bird2 format |

ROA files generated by [roa_wizard](https://github.com/Kioubit/dn42_registry_wizard) are available from kioubit.dn42:

|URL|&nbsp;IPv4/IPv6&nbsp;|Description|                                                                                      
|---|---|---|
| <https://kioubit-roa.dn42.dev/?type=v4> &nbsp; | &nbsp;IPv4 Only&nbsp; | Bird2 format |
| <https://kioubit-roa.dn42.dev/?type=v6> &nbsp; | &nbsp;IPv6 Only&nbsp; | Bird2 format |
| <https://kioubit-roa.dn42.dev/?type=json> &nbsp; | &nbsp;Both&nbsp; | JSON format for use with RPKI |

### Updating ROA tables

You can add cron entries to periodically update the tables:

```conf
*/15 * * * * curl -sfSLR {-o,-z}/etc/bird/roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird2_4.conf && birdc configure > /dev/null
*/15 * * * * curl -sfSLR {-o,-z}/etc/bird/roa_dn42_v6.conf https://dn42.burble.com/roa/dn42_roa_bird2_6.conf && birdc configure > /dev/null
```

Note that this is for the user crontab (`crontab -e`). For global crontabs (`/etc/crontab`, `/etc/cron.d/`, ...), you need to add an user:

```conf
*/15 * * * * bird curl -sfSLR {-o,-z}/etc/bird/roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird2_4.conf && birdc configure > /dev/null
*/15 * * * * bird curl -sfSLR {-o,-z}/etc/bird/roa_dn42_v6.conf https://dn42.burble.com/roa/dn42_roa_bird2_6.conf && birdc configure > /dev/null
```

Or use a systemd timer: (check the commands before copy-pasting)

```conf
# /etc/systemd/system/dn42-roa.service
[Unit]
Description=Update DN42 ROA

[Service]
Type=oneshot
ExecStart=curl -sfSLR -o /etc/bird/roa_dn42.conf -z /etc/bird/roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird2_4.conf
ExecStart=curl -sfSLR -o /etc/bird/roa_dn42_v6.conf -z /etc/bird/roa_dn42_v6.conf https://dn42.burble.com/roa/dn42_roa_bird2_6.conf
ExecStart=birdc configure
```

```conf
# /etc/systemd/system/dn42-roa.timer
[Unit]
Description=Update DN42 ROA periodically

[Timer]
OnBootSec=2m
OnUnitActiveSec=15m
AccuracySec=1m

[Install]
WantedBy=timers.target
```

then enable and start the timer with `systemctl enable --now dn42-roa.timer`.

More advanced script with error checking:
```sh
#!/bin/bash
roa4URL=""
roa6URL=""

roa4FILE="/etc/bird/roa/roa_dn42.conf"
roa6FILE="/etc/bird/roa/roa_dn42_v6.conf"

cp "${roa4FILE}" "${roa4FILE}.old"
cp "${roa6FILE}" "${roa6FILE}.old"

if curl -f -o "${roa4FILE}.new" "${roa4URL}" ;then
    mv "${roa4FILE}.new" "${roa4FILE}"
fi

if curl -f -o "${roa6FILE}.new" "${roa6URL}" ;then
    mv "${roa6FILE}.new" "${roa6FILE}"
fi

if birdc configure ; then
    rm "${roa4FILE}.old"
    rm "${roa6FILE}.old"
else
    mv "${roa4FILE}.old" "${roa4FILE}"
    mv "${roa6FILE}.old" "${roa6FILE}"
fi
```


### Use RPKI ROA in bird2

* Download stayrtr

<https://github.com/bgp/stayrtr>

* Run stayrtr.

```sh
./stayrtr -verify=false -checktime=false -cache=https://dn42.burble.com/roa/dn42_roa_46.json
```


* Run with docker

```sh
docker pull rpki/stayrtr
```

```sh
docker run --name dn42rpki -p 8282:8282 --restart=always -d rpki/stayrtr -verify=false -checktime=false -cache=https://dn42.burble.com/roa/dn42_roa_46.json
```

* Add this to your bird configure file (and remove other conflicting ROA setup, if present):

```conf
protocol rpki rpki_dn42{
  roa4 { table dn42_roa; };
  roa6 { table dn42_roa_v6; };

  remote "<your rpki server ip or domain>" port 8282;

  retry keep 90;
  refresh keep 900;
  expire keep 172800;
}
```
### Use BFD in bird2
BFD is an additional protocol to detect failures in the switching plane between peers,
it is used widely by cleanet peerings and some networks in DN42 already have enabled it globally.
To do a basic configuration you need to add 1 line to your bird.conf and enable it per peer or globally by defining it in the
template. 
It is currently recommended that you only enable it for each peer that supports it and has it enabled.
It acts as a detection mechanism that is very fast and can detect for example if your tunnel link is down.

Add this above the template for dnpeers.
```conf
protocol bfd {};
```
And below is an example for a MP-BGP over IPV6 with ENH peering 
`/etc/bird/peers/<NEIGHBOR>.conf`
Note bfd graceful; only activates when both sides have bfd configured and does not cause issues in peerings without BFD
```conf
protocol bgp <NEIGHBOR_NAME> from dnpeers {
    enable extended messages on;
    bfd graceful;
    bfd {
        interval 10s;
    };
    neighbor <NEIGHBOR_IPv6>%<NEIGHBOR_INTERFACE> as <NEIGHBOR_ASN>;
        # Or:
        # neighbor <NEIGHBOR_IPv6> as <NEIGHBOR_ASN>;
        # interface <NEIGHBOR_INTERFACE>;****
     ipv4 {
        extended next hop on;
    };
};
```

**Warning:** with high frequency connectivity issues and/or/with ping over ~150ms and/or/with significant packet loss
it can cause significant issues with route flapping since BFD uses UDP packets to reduce overhead.
It should not be used in extremely long distance peers with the default settings and use of it on 
lossy networks like but not only, Satellite, Wireless Mesh Networks should be avoided.
Regardless, use of BFD in high quality fiber based networks with low ping is optimal.

Additional documentation about the BFD protocol is available at [the BIRD2 documentation](https://bird.network.cz/?get_doc&v=20&f=bird-6.html#ss6.3) .