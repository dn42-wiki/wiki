# Looking Glasses

This is the list of **looking glasses** available for the dn42 network.  Some only display BGP information, while some others allow interactive queries (traceroute, details of a route, BGP-map visualisation, etc).

When a looking glass is described as `IPv4` or `IPv6`, it refers to the information displayed (or queried) by the looking glass, not to the reachability of the looking glass service itself.

## Reachable from the Internet & dn42

Please sort by AS number.

| Public/dn42 URL                            | AS                     | Remarks                    | State |
|:------------------------------------------------- |:---------------------- |:-------------------------- |:------|
|  http://lg.prauscher.de  http://lg.prauscher.dn42       |           64720         |                    | UP |
|  http://sour.is            |           64737        |         (IPv4 & IPv6) If you would like to submit your own site AS route information contact xuu@sour.is.   | UP |
|  http://ix.ucis.nl/routes.php   http://ix.ucis.dn42/routes.php        |           64766        |        interactive (traceroute)            | UP |
|  http://lg.nixnodes.net  http://lg.nixnodes.dn42         |           76103          |          interactive (traceroute)          | UP |
|  http://dn42.netrik.de/de-fra1/    http://172.22.232.5/de-fra1/         |          4242420013           |      interactive, bgpmap              | UP  |
|  http://edge1.core.chaos-darmstadt.de   http://lg.cda.dn42        |          4242420101          |                    |  |
|  http://peerfinder.polyno.me  http://peerfinder.polynome.dn42       |          4242420184           |        it can be used as a distributed looking glass if you give it a dn42 address.            | DOWN |
|  http://lg-fr-rbx.wolke7.me      http://lg-fr-rbx.wolke7.dn42      |          4242420300            |                    | DOWN |
|  http://as4242422222.hope.mx/lg.htm  http://as4242422222.dn42/lg.htm         |          4242422222           |                    | DOWN |
|  https://vpn01.weiti.org/ulg/  https://lg.weiti.dn42/      |          4242423905           |                    | UP |
|  http://zeus.nowhere.ws/dn42/routes.cgi  http://zeus.nihilus.dn42/dn42/routes.cgi      |          4242423905           |        (IPv4 & IPv6) → non-interactive (simply displays all known routes)             | DOWN |


## Reachable only from within dn42

Please sort by AS number.

| URL                                               | AS                     | Remarks                    | State |
|:------------------------------------------------- |:---------------------- |:-------------------------- |:------|
| http://lg.nordkapp-5.dn42 , http://172.22.235.4                       | 64835               |            interactive                | DOWN |
| http://lg.ffdn.dn42                        | 76142 |    interactive (traceroute, BGP-map) | DOWN |
| http://mhm.dn42:5001                             | 4242420022  | .  | UP |
| http://dataviz.polynome.dn42/dn42/lastseen/                            | 4242420184  | non-interactive ("BGP last seen" service: keeps an history of previously announced BGP prefixes)  | DOWN |
|  http://lg.punkt.dn42                          |  4242420200 | interactive (traceroute, BGP-map)  | DOWN |
|  http://lg.dn42                         |  4242420321 | interactive (traceroute, BGP-map) | UP |
|  http://lg.jan.dn42                          | 4242420812  | interactive (traceroute, BGP-map)  | UP |
|  http://lg.erg.dn4                          | 4242421092  | interactive (traceroute, BGP-map)  | DOWN |
|  http://lg.tech9computers.dn42                          | 4242421588  | interactive (traceroute, BGP-map)  | UP |
|  http://lg.gotroot.dn42                          | 4242422700  |  | UP |
|  http://lg.gbe.dn42                           | 4242422342  |  semi-interactive (no traceroute, no ping) | UP |
|  http://lg.flo.dn42                          | 4242423955  |   interactive (traceroute, ping) | DOWN |

## Reachable only from the Internet

Please sort by AS number.

* AS 65529: http://bgp.freifunk-bielefeld.de/ulg/ulg.py → interactive (BGP-map)
* AS 4242420123: http://lg.grmml.net → interactive (traceroute, BGP-map)