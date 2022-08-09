Bird is a commonly used BGP daemon. This page provides configuration and help for using BGP communities with Bird for dn42.

Communities can be used to prioritize traffic based on different flags, in DN42 we are using communities to prioritize based on latency, bandwidth and encryption. Please note that everyone should be using community 64511.

The community is applied to the route when it is imported and exported, therefore you need to change your bird configuration, in /etc/bird/peers4 if you followed the Bird guide.

The filter helpers can be stored in a separate file, for example /etc/bird/community_filters.conf.

Below, you will see an example config for peers4 based on the original filter implementation by Jplitza.

To properly assign the right community to your peer, please reference the table below. If you are running your own network and peering internally, please also apply the communities inside your network.

## BGP community criteria
```
(64511, 1) :: latency \in (0, 2.7ms]
(64511, 2) :: latency \in (2.7ms, 7.3ms]
(64511, 3) :: latency \in (7.3ms, 20ms]
(64511, 4) :: latency \in (20ms, 55ms]
(64511, 5) :: latency \in (55ms, 148ms]
(64511, 6) :: latency \in (148ms, 403ms]
(64511, 7) :: latency \in (403ms, 1097ms]
(64511, 8) :: latency \in (1097ms, 2981ms]
(64511, 9) :: latency > 2981ms
(64511, x) :: latency \in [exp(x-1), exp(x)] ms (for x < 10)

(64511, 21) :: bw >= 0.1mbit
(64511, 22) :: bw >= 1mbit
(64511, 23) :: bw >= 10mbit
(64511, 24) :: bw >= 100mbit
(64511, 25) :: bw >= 1000mbit
(64511, 2x) :: bw >= 10^(x-2) mbit
bw = min(up,down) for asymmetric connections

(64511, 31) :: not encrypted
(64511, 32) :: encrypted with unsafe vpn solution
(64511, 33) :: encrypted with safe vpn solution (but no PFS - the usual OpenVPN p2p configuration falls in this category)
(64511, 34) :: encrypted with safe vpn solution with PFS (Perfect Forward Secrecy)

Propagation:
- - for latency pick max(received_route.latency, link_latency)
- - for encryption and bandwidth pick min between received BGP community and peer link
```
For example, if your peer is 12ms away and the link speed between you is 250Mbit/s and you are peering using OpenVPN P2P, then the community string would be (3, 24, 33).

Two utilities which measure round trip time and calculate community values automatically are provided, written in  [ruby](https://github.com/Mic92/bird-dn42/blob/master/bgp-community.rb) and [C](https://github.com/nixnodes/bird/blob/master/misc/dn42-comgen.c).

```
$ ruby bgp-community.rb --help
USAGE: bgp-community.rb host mbit_speed unencrypted|unsafe|encrypted|pfs
    -6, --ipv6                       Assume ipv6 for ping
$ ruby bgp-community.rb 212.129.13.123 300 encrypted
  # 15 ms, 300 mbit/s, encrypted tunnel (updated: 2016-02-11)
  import where dn42_import_filter(3,24,33);
  export where dn42_export_filter(3,24,33);
$ ruby bgp-community.rb -6 dn42-2.higgsboson.tk 1000 pfs
  # 11 ms, 1000 mbit/s, pfs tunnel (updated: 2016-02-11)
  import where dn42_import_filter(3,25,34);
  export where dn42_export_filter(3,25,34);
```

### Route Origin
There are two type of route origin: `region` and `country`

#### Region
The range `41-70` is assigned to the region property.
The communities for route origin region were first defined in [December 2015](https://lists.nox.tf/pipermail/dn42/2015-December/001259.html) and further extended in [May 2022](https://groups.io/g/dn42/topic/91226190):

```
(64511, 41) :: Europe
(64511, 42) :: North America-E
(64511, 43) :: North America-C
(64511, 44) :: North America-W
(64511, 45) :: Central America
(64511, 46) :: South America-E
(64511, 47) :: South America-W
(64511, 48) :: Africa-N (above Sahara)
(64511, 49) :: Africa-S (below Sahara)
(64511, 50) :: Asia-S (IN,PK,BD)
(64511, 51) :: Asia-SE (TH,SG,PH,ID,MY)
(64511, 52) :: Asia-E (JP,CN,KR)
(64511, 53) :: Pacific&Oceania (AU,NZ,FJ)
(64511, 54) :: Antarctica
(64511, 55) :: Asia-N (RU)
(64511, 56) :: Asia-W (IR,TR,UAE)
(64511, 57) :: Central Asia (AF,UZ,KZ)
```

#### Country
The range `1000-1999` is assigned to the country property. Here we use [ISO-3166-1 numeric](https://github.com/lukes/ISO-3166-Countries-with-Regional-Codes/blob/master/all/all.csv) country codes, adding `1000` to each value to get the country origin community:

```
(64511, 1124) :: Canada
(64511, 1156) :: China
(64511, 1158) :: Taiwan
(64511, 1250) :: France
(64511, 1276) :: Germany
(64511, 1344) :: Hong Kong
(64511, 1392) :: Japan
(64511, 1528) :: Netherlands
(64511, 1578) :: Norway
(64511, 1643) :: Russian Federation
(64511, 1702) :: Singapore
(64511, 1756) :: Switzerland
(64511, 1826) :: United Kingdom
(64511, 1840) :: United States of America
# etc. Please follow the ISO-3166-1 Numeric standard
# https://github.com/lukes/ISO-3166-Countries-with-Regional-Codes/blob/master/all/all.csv
```

You need to add following lines to your config(s):
- `define DN42_REGION = $VALUE_FROM_ABOVE` to your node's config (where OWNAS and OWNIP are set)
- `if source = RTS_STATIC then bgp_community.add((64511, DN42_REGION));`
just above `update_flags` in `dn42_export_filter` function
- Unlike the other community values, **the DN42_REGION community should only be set on routes originating from your network!** (This is what the `source = RTS_STATIC` check does).
  - Otherwise, if you export routes across multiple regions within your network, you may be sending incorrect origin information to other peers.


## Example configurations
```
# /etc/bird/peers4/tombii.conf
protocol bgp tombii from dnpeers {
  neighbor 172.23.102.x as 4242420321;
  import where dn42_import_filter(3,24,33);
  export where dn42_export_filter(3,24,33);
};
```
```
#/etc/bird/community_filters.conf
function update_latency(int link_latency) {
  bgp_community.add((64511, link_latency));
       if (64511, 9) ~ bgp_community then { bgp_community.delete([(64511, 1..8)]); return 9; }
  else if (64511, 8) ~ bgp_community then { bgp_community.delete([(64511, 1..7)]); return 8; }
  else if (64511, 7) ~ bgp_community then { bgp_community.delete([(64511, 1..6)]); return 7; }
  else if (64511, 6) ~ bgp_community then { bgp_community.delete([(64511, 1..5)]); return 6; }
  else if (64511, 5) ~ bgp_community then { bgp_community.delete([(64511, 1..4)]); return 5; }
  else if (64511, 4) ~ bgp_community then { bgp_community.delete([(64511, 1..3)]); return 4; }
  else if (64511, 3) ~ bgp_community then { bgp_community.delete([(64511, 1..2)]); return 3; }
  else if (64511, 2) ~ bgp_community then { bgp_community.delete([(64511, 1..1)]); return 2; }
  else return 1;
}

function update_bandwidth(int link_bandwidth) {
  bgp_community.add((64511, link_bandwidth));
       if (64511, 21) ~ bgp_community then { bgp_community.delete([(64511, 22..29)]); return 21; }
  else if (64511, 22) ~ bgp_community then { bgp_community.delete([(64511, 23..29)]); return 22; }
  else if (64511, 23) ~ bgp_community then { bgp_community.delete([(64511, 24..29)]); return 23; }
  else if (64511, 24) ~ bgp_community then { bgp_community.delete([(64511, 25..29)]); return 24; }
  else if (64511, 25) ~ bgp_community then { bgp_community.delete([(64511, 26..29)]); return 25; }
  else if (64511, 26) ~ bgp_community then { bgp_community.delete([(64511, 27..29)]); return 26; }
  else if (64511, 27) ~ bgp_community then { bgp_community.delete([(64511, 28..29)]); return 27; }
  else if (64511, 28) ~ bgp_community then { bgp_community.delete([(64511, 29..29)]); return 28; }
  else return 29;
}

function update_crypto(int link_crypto) {
  bgp_community.add((64511, link_crypto));
       if (64511, 31) ~ bgp_community then { bgp_community.delete([(64511, 32..34)]); return 31; }
  else if (64511, 32) ~ bgp_community then { bgp_community.delete([(64511, 33..34)]); return 32; }
  else if (64511, 33) ~ bgp_community then { bgp_community.delete([(64511, 34..34)]); return 33; }
  else return 34;
}

function update_flags(int link_latency; int link_bandwidth; int link_crypto)
int dn42_latency;
int dn42_bandwidth;
int dn42_crypto;
{
  dn42_latency = update_latency(link_latency);
  dn42_bandwidth = update_bandwidth(link_bandwidth) - 20;
  dn42_crypto = update_crypto(link_crypto) - 30;
  # replace 4 with your calculated bandwidth value
  if dn42_bandwidth > 4 then dn42_bandwidth = 4;
  return true;
}

# Combines filter from local4.conf/local6.conf and filter4.conf/filter6.conf,
# which means, these must included before this file

function dn42_import_filter(int link_latency; int link_bandwidth; int link_crypto) {
  if is_valid_network() && !is_self_net() then {
    update_flags(link_latency, link_bandwidth, link_crypto);
    accept;
  }
  reject;
}

function dn42_export_filter(int link_latency; int link_bandwidth; int link_crypto) {
  if is_valid_network() then {
    update_flags(link_latency, link_bandwidth, link_crypto);
    accept;
  }
  reject;
}

```
Please remember to include /etc/bird/community_filters.conf in your bird.conf/birdc6.conf
```

# local configuration
######################
include "bird/local4.conf";

# filter helpers
#################

include "/etc/bird/filter4.conf";
include "/etc/bird/community_filters.conf";
```


***
