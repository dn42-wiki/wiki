# Global Route Collector

The Global Route Collector (GRC) provides a real time view of routing and peering across DN42 and can be used to generate maps, stats or just query how routes are being propagated across the network. 

Technically the GRC is a [bird](https://bird.network.cz/) instance that anyone can peer with, it imports all routes whilst exporting none and provides a number of interfaces for querying the route data.

Data from the GRC is used to generate some of the DN42 Maps (see the [Internal Services](/internal/Internal-Services) page).

## Peering with the collector

The collector uses the dynamic peering capability in Bird2 to allow anyone to peer with it without any new server side configuration being required. The collector relies on users peering with it across the network so the more peers the better and the more comprehensive the collector data will be.

||Details|
|:--|:--|
| ASN | AS4242422602 |
| Hostname | collector.dn42 |
| IPv4 Address | 172.20.129.4 |
| IPv6 Address | fd42:4242:2601:ac12::1 |

### BGP Configuration

 - Unlike normal DN42 peerings, you must enable multihop to peer with the collector
 - The collector supports Multiprotocol BGP, so you don't need to configure separate IPv4 and IPv6 sessions
 - Please enable the Add Paths BGP extension to export all available routes

Example bird2 config:

```conf
protocol bgp ROUTE_COLLECTOR
{
  local as ***YOUR_ASN***;
  neighbor fd42:4242:2601:ac12::1 as 4242422602;

  # enable multihop as the collector is not locally connected
  multihop;

  ipv4 {
    # export all available paths to the collector    
    add paths tx;

    # import/export filters
    import none;
    export filter {
      # export all valid routes
      if ( is_valid_network() && source ~ [ RTS_STATIC, RTS_BGP ] )
      then {
        accept;
      }
      reject;
    };
  };

  ipv6 {
    # export all available paths to the collector    
    add paths tx;

    # import/export filters
    import none;
    export filter {
      # export all valid routes
      if ( is_valid_network_v6() && source ~ [ RTS_STATIC, RTS_BGP ] )
      then {
        accept;
      }
      reject;
    };
  };
}
```

Example VyOS 1.4 "Sagitta" config
```
# The route collector should never export routes, so let's make a route-map to reject them if it does.
set policy route-map Deny-All rule 1 action deny
set protocols bgp neighbor fd42:4242:2601:ac12::1 address-family ipv4-unicast route-map import 'Deny-All'
set protocols bgp neighbor fd42:4242:2601:ac12::1 address-family ipv6-unicast route-map import 'Deny-All'
set protocols bgp neighbor fd42:4242:2601:ac12::1 description 'https://lg.collector.dn42'
set protocols bgp neighbor fd42:4242:2601:ac12::1 ebgp-multihop '10'
set protocols bgp neighbor fd42:4242:2601:ac12::1 remote-as '4242422602'

```


## Querying the collector

### Looking Glass

The collector runs a looking glass based on [bird-lg-go](https://github.com/xddxdd/bird-lg-go). 

 - <https://lg.collector.dn42/>

### MRT Dumps

[MRT Dumps](https://tools.ietf.org/html/rfc6396) are produced by the collector every 10 minutes. Bird produces MRT dumps corresponding to tables, so two separate dumps are created, one for IPv4 (master4) and one for IPv6 (master6). The 10 minutes dumps are available for one week before being reduced down to one a day. 

 - <https://mrt.collector.dn42>

The latest dumps can always be found at the following URLs:

 - <https://mrt.collector.dn42/master4_latest.mrt.bz2>
 - <https://mrt.collector.dn42/master6_latest.mrt.bz2>

### Prometheus Metrics

The collector runs [bird_exporter](https://github.com/czerwonk/bird_exporter) and prometheus style metrics are available at the following URL:

 - <http://collector.dn42:9324/metrics>

### SSH Interface

The collector bird instance can be queried directly using a birdc shell.

 - ssh shell@collector.dn42

```sh
$ ssh shell@collector.dn42
------------------------------------
*    DN42 Global Route Collector   *
------------------------------------
* http://collector.dn42/

This service provides a bird2 shell
for querying the route collector

Be nice, access is logged and
abuse will not be tolerated
------------------------------------
BIRD burble-2.0.8-210322-1-ge6133456 ready.
Access restricted
bird> show route count
bird>       297441 of 297441 routes for 502 networks in table master4
286007 of 286007 routes for 427 networks in table master6
1437 of 1437 routes for 1437 networks in table dn42_roa4
1231 of 1231 routes for 1231 networks in table dn42_roa6
Total: 586116 of 586116 routes for 3597 networks in 4 tables
bird> 

```

