## Multicast

RFC 8815 deprecated PIM-SM so PIM-SSM is the way to go!

### Setup

For it to work, you'll need to do the following:
  * Ask your peering to enable ipv4/ipv6 multicast AFI on your peering
  * Set up IPv4/IPv6 PIM for the (s,g) joins to pass through
  * Be prepared to setup IGMP v3 and MLD v2 from your listeners

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

## Current streams

Public multicast to unicast relay with vlc4 and above:
* `vlc rtp://172.23.199.110@232.2.3.2:1234/`
* `vlc rtp://[fd40:cc1e:c0de::fffe]@[ff07::232:2:3:2]:1234/`
* `vlc --amt-relay amt-relay.rtr2.c4e.hbone.hu amt://10.2.255.1@232.2.3.2:1234/`
* `vlc --amt-relay amt-relay.geant.org amt://10.2.255.1@232.2.3.2:1234/`
the last one needs vlc4 off their nighty, the amt relay usually defuncs as geant rents the p4 testbed....

Controllable at <http://webdj.nop.dn42/>

Feel free to ask for a peering and set it up!
