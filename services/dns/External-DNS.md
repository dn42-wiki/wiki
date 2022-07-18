This page lists external DNS zones, provided by networks that are interconnected with dn42.

## Authoritative nameservers


| **Network name** | **Contact** | **DNS zone** | **Reverse zone** | **Authoritative nameservers** | **Last update** | **Comments** |
|:----------------:|:----------:|:------------:|:----------------:|-------------------------------|--------------|---------|
| ChaosVPN | - | `hack.` | `31.172.in-addr.arpa.` `100.10.in-addr.arpa` `101.10.in-addr.arpa` `102.10.in-addr.arpa` `103.10.in-addr.arpa` | `172.31.0.5` `172.31.255.53` (anycast) | Nov. 2013 | - |
| NeoNetwork | - | `neo.` | `127.10.in-addr.arpa` `7.2.1.0.0.1.d.f.ip6.arpa` | `10.127.255.53` `fd10:127:ffff:53::` | Jul. 2022 | - |

## Freifunk

Freifunk generates its zone configuration from the [icvpn-meta](https://github.com/freifunk/icvpn-meta) repositority via the **mkdns**-script contained in [icvpn-scripts](https://github.com/freifunk/icvpn-scripts). As there are many Freifunk-Communities this is the easiest way to get them all.

    git clone https://github.com/freifunk/icvpn-scripts.git
    git clone https://github.com/freifunk/icvpn-meta.git
    cd icvpn-scripts
    ./mkdns -f bind -s ../icvpn-meta/ -x dn42 -x chaosvpn -x rzl -x neonetwork

_Note: dn42, chaosvpn (, rzl which has been removed, but is still in the script) and neonetwork are excluded (-x), because they are not part of Freifunk and you might want to configure them yourself._

The mkdns script currently supports the following setups:
* bind (static-stub)
* bind-forward (forward zone)
* dnsmasq
* unbound (forward-zone)

## NeoNetwork

The NeoNetwork also has a recursive DNS server at `10.127.255.54` and `fd10:127:53:53::`.
The zone files can be found here: https://github.com/NeoCloud/NeoNetwork/tree/master/dns

## Configuration

See [DNS forwarding configuration](/services/dns/Configuration).
