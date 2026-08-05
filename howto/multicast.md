## Multicast

[RFC 8815](https://datatracker.ietf.org/doc/html/rfc8815) deprecated interdomain PIM-ASM (any-source multicast) so PIM-SSM (source-specific multicast) is the way to go!

### Setup

For it to work, you'll need to do the following:
  * Ask your peering to enable IPv4/IPv6 multicast AFI on your peering
  * Set up IPv4/IPv6 PIM for the (s,g) joins to pass through
  * Be prepared to setup IGMPv3 and MLDv2 from your listeners

You're done! You should receive the multicast routes from peers advertising them.

[FRR configuration](https://git.lemonsh.moe/C4TG1RL5/dn42/src/branch/master/lab.rtr.famfo.catgirls.dn42/frr) used by C4TG1RL5.
_Please make sure you understand how to configure and use frr before you use anything from this configuration!_

### Participants

Current participants:
  * NOP-MNT
  * GRAWITY-MNT
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
