# Frequently Asked Questions

[[_TOC_]]

### How do I connect to DN42 ?

We have a [page](/howto/Getting-started) for that !

### What BGP daemon should I use?

This is really up to you: that's the magic of open protocols.

Many users run Bird or FRRouting in a VPS, but there is a variety of BGP implementations in use across the network and some networks even run hardware routers or experimental software. DN42 is a great place to try out and get familiar with your implementation of choice in a live network. 

### Can I connect to DN42 from home through my ISP ?

Sure. As long as you have a 24/7 connection, can configure VPN tunnels and run BGP then you can connect to DN42. Do let your peers know if you have a dynamic IP address and you may also want to consider preventing transit traffic through your network if you have a relatively slow connection.

Many users do use a virtual private server (VPS) for connecting to DN42. A VPS is a relatively cheap option that  is always on and which (typically) provides a faster, more stable connection than your home. VPS have another advantage that if you mess something up, you won't break your home access to the internet. 

### Does the registry use monotone or git ?

The DN42 registry used to be maintained in monotone, but was moved to git following resource and performance
issues. There may still be references back to monotone in some of the documentation, but the registry location is now:

https://git.dn42.us/dn42/registry


### What about IPv6 in dn42?

DN42 supports IPv6 and some users do run IPv6 only networks. DN42 uses the "Unique Local Address" range (ULA, *fd00::/8*) defined by [RFC 4193](https://tools.ietf.org/html/rfc4193). 

Globally routable IPv6 prefixes are not supported in DN42; whilst in theory they could be announced, most networks will reject the prefixes and some networks will filter traffic to the ULA range only. 

### Why are you using ASN in the 76100-76199 range?

DN42 now uses ASNs in the 4242420000-4242429999 range, but the 76100-76199 range was used historically, and there are still a very small number of networks using legacy ASNs that have not migrated. 

Previously, DN42 has used a two other ASN ranges:

We used to assign ASN in the 64600-64855 range, where you would get ASN 64600+X if you had 172.22.X.0/24. Once the address range was extended to include additional blocks, this process was obsoleted. Another issue with the private ASN range 64512-65534 was that other projects are also using it (for instance, Freifunk, Anonet, etc), which can lead to conflicts. 

Prior to using ASNs in the new private ASN range 4200000000-4294967294 ([RFC6996](http://tools.ietf.org/html/rfc6996)) Some ASNs were allocated from the [ASN reserved block](http://www.iana.org/assignments/as-numbers/as-numbers.xhtml) in the 76100-76199 range. 

### Can I update the wiki?

Yes, the wiki can be edited when browsing to [wiki.dn42](https://wiki.dn42).