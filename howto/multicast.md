## Multicast

[RFC 8815](https://datatracker.ietf.org/doc/html/rfc8815) deprecated interdomain PIM-ASM (any-source multicast) so PIM-SSM (source-specific multicast) is the way to go!

### Setup

For it to work, you'll need to do the following:
  * Ask your peering to enable IPv4/IPv6 multicast AFI on your peering
  * Set up IPv4/IPv6 PIM for the (s,g) joins to pass through
  * Be prepared to setup IGMPv3 and MLDv2 from your listeners

You're done! You should receive the multicast routes from peers advertising them.

### frr configuration

Enable PIM:
```
router pim
 no autorp discovery
exit
router pim6
exit
```
For IPv4, you can also add the `send-v6-secondary` directive, which allows you to omit an IPv4 address (as with "extended next hop") on the interface. (Does this work? I haven't tried it yet.)

Import kernel multicast table into frr:
```
ip import-table [TABLE NUMBER] mrib
ipv6 import-table [TABLE NUMBER] mrib
```
This method is suitable when frr is used only as a multicast routing daemon and another daemon, such as bird, is used for BGP. In this case, bird can export the unicast routes intended for multicast to a special kernel routing table, and frr can import them. If frr also handles BGP, it can store the corresponding unicast routes directly in mrib.

Enable PIM on a interface:
```
interface [INTERFACE NAME]
 ip pim
 ip pim use-source [SOURCE IPv4 ADDRESS]
 ipv6 pim
 ipv6 pim use-source [SOURCE IPv6 ADDRESS]
exit
```
Specifying a source address is optional, but can be used if frr would otherwise select the wrong address.

Enable multicast for a client:
```
interface client21
 ip igmp
 ip igmp max-groups 32
 ip igmp require-router-alert
 ip igmp version 3
 ip pim
 ip pim passive
 ipv6 mld
 ipv6 mld max-groups 32
 ipv6 mld require-router-alert
 ipv6 mld version 2
 ipv6 pim
 ipv6 pim passive
exit
```
The number of maximum groups is optional and can be set freely. For security reasons, this number should not be set unreasonably high. Specifying an explicit IGMP or MLD version is also optional.
IGMP and MLD messages must follow a specific format. `require-router-alert` filters out invalid messages.
Even if an interface is only intended to handle IGMP/MLD, PIM must be enabled on it. If you still do not want to use PIM, a firewall rule is recommended.

### nftables configuration

Allow PIM:
```
iifname "[INTERFACE NAME]" ip6 saddr [SOURCE ADDRESS] ip6 daddr ff02::d meta protocol ip6 meta l4proto pim counter accept;
```
```
iifname "[INTERFACE NAME]" ip saddr [SOURCE ADDRESS] ip daddr 224.0.0.13 meta protocol ip meta l4proto pim counter accept;
```

Allow MLDv2:
```
set icmp6_mld {
    type icmpv6_type . icmpv6_code;
    flags interval;
    elements = {
        mld-listener-query . 0,
        mld2-listener-report . 0
    };
}
```
```
iifname [CLIENT INTERFACE NAME] icmpv6 type . icmpv6 code @icmp6_mld ip6 hoplimit 1 exthdr hbh exists ip6 saddr fe80::/10 counter accept;
```

Allow IGMPv3:
```
iifname [CLIENT INTERFACE NAME] ip protocol igmp igmp type { membership-query, membership-report-v3 } ip ttl 1 counter accept;
```

### Participants

Current participants:
  * NOP-MNT
  * MIRSAL-MNT
  * C4TG1RL5-MNT
  * KIOUBIT-MNT
  * PREVARINITE-MNT
  * MARK22K-MNT

Feel free to ask for a peering and set it up!

## Current streams

### NOP music stream

cs broadcasts a 96 kHz, 24-bit music stream. An SDP file is required to receive it:
```
v=0
o=Node 0 0 IN IP4 172.23.199.110
s=None
c=IN IP4 232.2.3.2
t=0 0
m=audio 1234 RTP/AVP 96
a=rtpmap:96 L24/96000/2
a=source-filter: incl IN IP4 232.2.3.2 172.23.199.110
```
```
v=0
o=Node 0 0 IN IP6 fd40:cc1e:c0de::fffe
s=None
c=IN IP6 ff3e::8232:232
t=0 0
m=audio 1234 RTP/AVP 96
a=rtpmap:96 L24/96000/2
a=source-filter: incl IN IP6 ff3e::8232:232 fd40:cc1e:c0de::fffe
```

The stream can then be streamed with `ffplay -protocol_whitelist file,fd,udp,rtp -fflags +genpts /path/to/sdp_file`.

### mping

mping is a program that sends a ping message to a multicast group every second (and responds to incoming pings, if any).
There are two implementations:
* The [original implementation](https://github.com/troglobit/mping/) by troglobit, which supports both sending and receiving.
* A [reimplementation](https://codeberg.org/mark22k/mping-sender), which supports only sending.

* `mcjoin -j -i [INTERFACE] [fd00:8e13:ce5d::9bee],[ff3e::8000:42]:4321` mping-sender
