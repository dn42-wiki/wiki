This page lists external DNS zones, provided by networks that are interconnected with dn42.

## Authoritative nameservers

| **Network name** | **Contact** | **DNS zone** | **Reverse zone** | **Authoritative nameservers** | **Last update** | **Comments** |
|:----------------:|:----------:|:------------:|:----------------:|-------------------------------|--------------|---------|
| ChaosVPN | - | `hack.` | `31.172.in-addr.arpa.` | `172.31.0.5` | Nov. 2013 | - |
| RaumZeitLabor | - | `rzl.` | - | `172.22.36.1` | Nov. 2013 | - |

## Freifunk

Freifunk generates its zone configuration from the [icvpn-meta](https://github.com/freifunk/icvpn-meta) repositority via the **mkdns**-script contained in [icvpn-scripts](https://github.com/freifunk/icvpn-scripts). As there are many Freifunk-Communities this is the easiest way to get them all.

    git clone https://github.com/freifunk/icvpn-scripts.git
    git clone https://github.com/freifunk/icvpn-meta.git
    cd icvpn-scripts
    ./mkdns -f bind -s ../icvpn-meta/ -x dn42

The mkdns script currently supports the following setups:
* bind (static-stub)
* bind-forward (forward zone)
* dnsmasq
## Configuration

See [[Recursive DNS resolver]] or [[DNS forwarding configuration|/services/dns/Configuration]].