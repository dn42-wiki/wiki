[[_TOC_]]

## Why are you using monotone for the registry? Why not GIT?

There is an important difference between the data model of monotone and GIT: In GIT branches *are* HEADs, while in monotone, branches are a list of HEADs. Or, to state it simpler and probably less correct: It is possible to sync merge conflicts in monotone. In GIT, conflicts are part of the index and/or working tree, and thus can't be pushed/pulled.

The DN42 registry is stored on multiple monotone servers which sync with each other. This is not possible in GIT, because the GIT servers don't know how to handle merge conflicts. In monotone, the servers just sync the conflict.


## What about IPv6 in DN42?

There are some ASes in DN42 that route IPv6 traffic. It is not yet agreed upon what prefixes should be used. The following proposals are the more sane ones:

* Use Unique Local Addresses (ULAs). This is the *fd00::/8* range. In theory, this would be the obvious winner of this debate. They were standardised for exactly this purpose (not publicly routed networks that still want to use unique prefixes). Sadly, this would require you to announce two prefixes in your LAN if you want to use stateless autoconfiguration and no NAT: The ULA and a globally routed prefix. It is not yet known if this really works. [RFC 3484](http://www.rfc-editor.org/rfc/rfc3484.txt) demands a behavior that would make this work at the moment (until globally routed addresses from 8000::/1 are used)
* Use your globally unique PA space. This fixes the LAN-issue, because you only need to announce a single prefix.  However, this complicates prefix filtering for everybody, and can lead to strange routing patterns, where packets are routed partially on dn42 and partially through the Internet.

(*TODO*)

At the moment, it is safe to assume that everyone doing IPv6 routing accepts at least prefixes from fd00::/8 with prefix lengths between 48 and 64 bits (inclusive) if they are part of the registry.


## Why are you using ASN in the 76100-76199 range?

Yes, we know that this is not private ASN space (rather, it is part of the reserved block 65552-131071, see [IANA](http://www.iana.org/assignments/as-numbers/as-numbers.xhtml)).

We used to assign ASN in the 64600-64855 range, where you would get ASN 64600+X if you had 172.22.X.0/24.  Since we are now assigning /25 by default, and we have extended the address range to include 172.23.0.0/16, this is legacy.

Another issue with the private ASN range 64512-65534: other projects are also using it (for instance, Freifunk, Anonet, etc), which can lead to conflicts.

Fortunately, [RFC6996](http://tools.ietf.org/html/rfc6996) defines a new private ASN range: 4200000000-4294967294.  Given the size of this range, there is little chance of running into a conflict.

We now encourage dn42 users to use the newly-allocated ranges in **4242420000-4242429999**. See the [registry page](Services-Whois#AS-numbers) for details.


## What BGP daemon should I use?

This is really up to you: that's the magic of open protocols.

As of 2014, most people seem to use either Quagga or Bird (with a growing preference for Bird). You may also encounter users of OpenBGPd. Even more occasionally, people use hardware BGP routers in dn42, for instance [Extreme Networks](howto/bgp-on-extreme-summit5i) hardware.

