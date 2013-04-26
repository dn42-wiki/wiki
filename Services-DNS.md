# DNS

We have a TLD for the dn42, which is .dn42 and the Anycast-DNS-server for it runs on 172.22.0.53.
**DNS is build from whois database. So please edit your DNS-records there.**

## Anycast DNS

Provides a resolver for, but not only, the dn42 zones(.dn42 currently) on a dns-server close to you.

The nameservers in that cloud will happily accept any request and will try to resolve it, but please be aware, that by hitting those servers with queries for e.g. google.com they could fake those result and point you towards the russian mafia. They probably won't, but think about what you are doing. - At the end of the day, your ISP could be evil as well, so it always boils down to a question of trust.

Configuration requirements for all members of the anycast group are:
 * maintain your own zones based on whois database (scripts included in repository)
 * allow recursion (including ".")
 * be authoritative for .dn42
 * listen on a unicast IP too for testing/debugging reasons
 * with bind, please use ```minimal-responses yes;``` (goes into ```options```/```view```)

It is _really_ good to hang around in [[IRC|Services IRC]] to get things sorted out, if something doesn't work. Letting some people test you DNS' behavior before joining the anycast-group is considered best practice - better safe than sorry.

 * **IP:** 172.22.0.53
 * **Announciation Subnet:** 172.22.0.53/32

| **person**   | **AS** | **unicast-name**            | **unicast address** | **comments**                                            |
|----|:-------:|:-------:|:-------:|----------------------------------------------------|
| nihilus        | 64692    | dnscache.zeus.dn42.nowhere.ws | 172.22.92.123         |                                                           |
| wintix         | 64822    | ns1.wintix.dn42               | 172.22.222.1          |                                                           |
| wintix         | 64823    | ns2.wintix.dn42               | 172.22.223.1          |                                                           |
| somerandomnick | 64731    | -                             | 172.22.131.38         | down pending rDNS debate                                  |
| crest          | 64828    | ns3.crest.dn42                | 172.22.228.84         | authorative only                                          |
| crest          | 64828    | ns2.crest.dn42                | 172.22.228.85         | public caching resolver                                   |
| siska          | 76103    | nixnodes.root.dn42             | 172.22.177.2          | authoritative only |
| siska          | 76103    | ns1.nixnodes.dn42             | 172.22.177.2          | caching   |
| siska          | 76105    | ns2.nixnodes.dn42             | 172.22.177.1        | caching                                                   |

For configuring concrete DNS caches see: [[DNS Configuration|Services DNS Configuration]]