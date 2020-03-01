This guide describes a simple configuration for [OpenBGPD](https://openbgpd.org) running on [OpenBSD](https://openbsd.org).
The [portable version](https://openbgpd.org/ftp.html) should run with little to no configuration changes on other operating systems as well.

# Setup
Only IPv6 is used for the sake of simplicity.
Neighbors use ULA addresses (/127 transfer net) assigned from one of the peer's allocation.

The goal is to have a small, yet complete setup for all peers with ROA validation and other safety measurements in place.

# Configuration
[`/etc/bgpd.conf`](https://man.openbsd.org/bgpd.conf.5) contains all information and includes generated pieces such as ROA sets;  see the `ROA` section in this guide.

As per the manual, configuration is divided into logical sections;  [`/etc/examples/bgpd.conf`](http://cvsweb.openbsd.org/cgi-bin/cvsweb/~checkout~/src/etc/examples/bgpd.conf?rev=HEAD&content-type=text/plain&only_with_tag=MAIN) is a complete and commented example which this guide is roughly based on.

By default, **bgpd** listens on all local addresses (on the current default [`routing domain`](http://man.openbsd.org/rdomain.4)), but this guide explicitly listens on the configured transfer ULA only for each peer to better illustrate of this setup.

## local peer
Information such as ASN, router ID and allocated networks are required:
```
# macros
ASN="4242421234"

# global configuration
AS $ASN
router-id 1.2.3.4

prefix-set mynetworks {
        fd00:12:34::/48
}
```

These can be used in subsequent filter rules.
The local peer's announcements is then defined as follows:
```
# Generate routes for the networks our ASN will originate.
# The communities (read 'tags') are later used to match on what
# is announced to EBGP neighbors
network prefix-set mynetworks set large-community $ASN:1:1
```

## neighbors
For each neighbor its ASN and transfer ULA is required.
An optional description is provided such that [`bgpctl`](http://man.openbsd.org/bgpctl.8) for example can be used with mnemonic names instead of AS numbers:
```
$peerA-local="fd00:12:34:A::1"
$peerA-remote="fd00:12:34:A::2"
$peerA-ASN="4242425678"

listen on $peerA-local
neighbor  $peerA-remote {
    remote-as $peerA-ASN
    descr "peerA"
}
```

# ROA

# Looking glass