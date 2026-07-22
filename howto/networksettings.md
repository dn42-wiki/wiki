## Important rules for Linux

The first rule of dn42: Always disable `rp_filter`. 

The second rule of dn42: Always disable `rp_filter`. 

The third rule of dn42: Allow ip forwarding!

No seriously, in case some packets are dropped, first check if your settings are correct.

`rp_filter` also known as reverse path filtering is a security measure. 
When the route to return a packet uses a different interface than it arrived from, the packet is dropped. 
Some attackers will set a wrong return address on their packets. This security measure was created to address when this happens. Core internet routing can however be asymmetric. This means that packets can take different routes on the return path.
That is why `rp_filter` needs to be disabled.

**Note** using sysctl is not persistent. Depending on your linux distribution put it into `/etc/sysctl.conf` or `/etc/sysctl.d`  
*(see also the note below if you are running Debian Trixie or your OS is using systemd-sysctl)*

```sh
sysctl -w net.ipv4.conf.all.rp_filter=0 net.ipv4.conf.default.rp_filter=0
```

Check that its really disabled:
```sh
sysctl -a | grep rp_filter
```

**Note** The max value from `conf/{all,interface}/rp_filter` is used when doing source validation on the `{interface}`. In case you don't want to disable `rp_filter` for the entire system, the correct setup is to set both `net.ipv4.conf.all.rp_filter=0` and `net.ipv4.conf.{interface}.rp_filter=0`, where `{interface}` is your vpn interface.

Also the following options must be set.
```sh
$ sysctl -w net.ipv4.ip_forward=1 net.ipv4.conf.all.forwarding=1 net.ipv6.conf.all.forwarding=1
```

Check that ALL your vpn interfaces allow ip forwarding for ipv6/ipv4.
```sh
$ sysctl -a | grep forwarding
```

**systemd-sysctl**

systemd-sysctl parses files in **lexical ordering** regardless of which directory they are in.  
See the [sysctl.d documentation](https://www.freedesktop.org/software/systemd/man/latest/sysctl.d.html#Configuration%20Directories%20and%20Precedence) for more details on this; the documentation also notes:

 *It is recommended to use the range 10-40 for configuration files in /usr/ and the range 60-90 for configuration files in /etc/ and /run/, to make sure that local and transient configuration files will always take priority over configuration files shipped by the OS vendor.*

Debian Trixie ships with a `/usr/lib/sysctl.d/50-default.conf` file which sets rp_filter to 2.

The net result is you must ensure that the filenames you create in /etc/sysctl.d for overrides are lexically after '50-default.conf' or your settings will not have any effect. Additionally for Debian Trixie and newer, you should consider also setting `net.ipv4.conf.*.rp_filter=0` to override the per-interface defaults.


## Note on firewalls, conntrack and asymmetric routing

Do not configure iptables/nftables to drop packets with invalid conntrack state in forward chain.

In some cases your router will not see traffic from both sides e.g. requests are sent via different path not including your networks
but responses are forwarded via your network. This will prevent conntrack from assigning any meaningful state information to these packets
and your firewall will drop it if it is configured to drop packets with invalid state.


## Avoiding Issues with Peer Addressing

When configuring BGP peers in dn42, be cautious about using DN42 IP addresses as peer addresses, particularly in the following scenario:
- You are NOT using extended next hop
- You ARE using the same IP for other services

### The Problem
If your peering link goes down (while the interface remains configured), your peer may have a static route for your service IP via the non-functional tunnel interface. This can make your services inaccessible, even if you have other working peers, because:
- The static route typically has higher priority than BGP routes
- Wireguard interfaces don't automatically go down when peers are unreachable
- Traffic might be routed through peers with broken static routes

This is especially likely to occur if your peers are major transit providers within DN42.

### Solutions
To avoid this issue, use one of these approaches:
1. Use extended next hop in BGP configuration
2. Use non-DN42 addresses for BGP peering
3. Use different IPs for services than for peering
4. De-couple services from specific nodes

## MTU, Anycast and TCP MSS Clamping

In DN42, you are almost always running over tunnels (WireGuard, GRE, etc.). Different routers and tunnels may use different MTU. Combined with Anycast, this creates a specific failure mode for Path MTU Discovery (PMTUD).

### The Anycast PMTUD Blackhole

When a router sends an ICMPv6 "Packet Too Big" (PTB) message to an Anycast address, the network may route that ICMP packet to a different Anycast instance than the one that sent the original data. As a result, the actual sender never learns to reduce its packet size, leading to "hanging" connections that pass the TCP handshake but fail when transferring data.

### The Solution: MSS Clamping

To ensure stability, you must clamp the TCP Maximum Segment Size (MSS) on all nodes. This forces the TCP handshake to negotiate a segment size that fits within your smallest link MTU, bypassing the need for PMTUD.

It is highly recommended to set this on both `FORWARD` and `OUTPUT` chains of all routers.

Using nftables (recommended):
```
chain forward {
    type filter hook forward priority filter; policy accept;
    tcp flags syn tcp option maxseg size set rt mtu
}
chain output {
    type filter hook output priority filter; policy accept;
    tcp flags syn tcp option maxseg size set rt mtu
}
```

Using ip(6)tables:
```
# For IPv4
iptables -t mangle -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
iptables -t mangle -A OUTPUT -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu

# For IPv6
ip6tables -t mangle -A FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
ip6tables -t mangle -A OUTPUT -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
```

## Other Non-Trivial Pitfalls

- **accept_local sysctl settings**: When running anycast services on routers, ensure the accept_local sysctl is enabled. Without this setting, a router might drop transit traffic from other origins that has the anycasted IP as the source address, breaking connectivity through your network for those services.

- **Inadequate source IP filtering**: Services with public internet access require careful source IP filtering. For example, a DNS server in DN42 might receive requests with spoofed source IPs from inside DN42 that appear to come from public internet addresses. Without proper filtering, your server could respond to these spoofed requests, potentially participating in reflection attacks or exposing internal services to the public internet.

Happy Routing!
