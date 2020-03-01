This guide describes a simple configuration for [OpenBGPD](https://openbgpd.org) running on [OpenBSD](https://openbsd.org).
The [portable version](https://openbgpd.org/ftp.html) should run with little to no configuration changes on other operating systems as well.

# Setup
Only IPv6 is used for the sake of simplicity.
Neighbors use ULA addresses (/127 transfer net) assigned from one of the peer's allocation.

The goal is to have a small, yet complete setup for all peers with ROA validation and other safety measurements in place.

# Configuration
[`/etc/bgpd.conf`](https://man.openbsd.org/bgpd.conf.5) contains all information and includes generated pieces such as ROA sets;  see the `ROA` section in this guide.

As per the manual, configuration is divided into logical sections;  [`/etc/examples/bgpd.conf`](http://cvsweb.openbsd.org/cgi-bin/cvsweb/~checkout~/src/etc/examples/bgpd.conf?rev=HEAD&content-type=text/plain&only_with_tag=MAIN) is a complete and commented example which this guide is roughly based on.

By default, [`bgpd(8)`](http://man.openbsd.org/bgpd.8) listens on all local addresses (on the current default [`routing domain`](http://man.openbsd.org/rdomain.4)), but this guide explicitly listens on the configured transfer ULA only for each peer to better illustrate of this setup.

## local host
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
# peer A, transport over IPSec/GRE
$A-local="fd00:12:34:A::1"
$A-remote="fd00:12:34:A::2"
$A-ASN="4242425678"

listen on $A-local

neighbor  $A-remote {
    remote-as $A-ASN
    descr "A"
}
```

## filter rules
**bgpd** blocks all BGP __UPDATE__ messages by default.
The filter rules are evaluated in sequential order, form first to last.
The last matching allow or deny rule decides what action is taken.

Start off with basic protection and sanity rules:
```
# deny more-specifics of our own originated prefixes
deny quick from ebgp prefix-set mynetworks or-longer

# filter out too long paths, establish more peerings instead
deny quick from any max-as-len 8
```

`quick` rules are considered the last matching rule, and evaluation of subsequent rules is skipped.

Next IBGP as well as our own __UPDATES__ are allowed:
```
# IBGP: allow all updates to and from our IBGP neighbors
allow from ibgp
allow to ibgp

# Outbound EBGP: only allow self originated networks to ebgp peers
# Don't leak any routes from upstream or peering sessions. This is done
# by checking for routes that are tagged with the large-community $ASN:1:1
allow to ebgp prefix-set kn large-community $ASN:1:1
```

# ROA

# Looking glass