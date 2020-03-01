This guide describes a simple configuration for [OpenBGPD](https://openbgpd.org) running on [OpenBSD](https://openbsd.org).
The [portable version](https://openbgpd.org/ftp.html) should run with little to no configuration changes on other operating systems as well.

# Setup
Only IPv6 is used for the sake of simplicity.
Neighbors use ULA addresses (/127 transfer net) assigned from one of the peer's allocation.

The goal is to have a small, yet complete setup for all peers with ROA validation and other safety measurements in place.

# Configuration
[`/etc/bgpd.conf`](https://man.openbsd.org/bgpd.conf.5) contains all information and may include further (automatically generated) files, as is done in this guide.

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
An optional description is provided such that [**bgpctl(8)**](http://man.openbsd.org/bgpctl.8) for example can be used with mnemonic names instead of AS numbers:
```
# peer A, transport over IPSec/GRE
$A_local="fd00:12:34:A::1"
$A_remote="fd00:12:34:A::2"
$A_ASN="4242425678"

listen on $A_local

neighbor  $A_remote {
    remote-as $A_ASN
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

Allow own announcements:
```
# Outbound EBGP: only allow self originated networks to ebgp peers
# Don't leak any routes from upstream or peering sessions. This is done
# by checking for routes that are tagged with the large-community $ASN:1:1
allow to ebgp prefix-set kn large-community $ASN:1:1
```

Allow all remaining UPDATES based on **O**rigin **V**alidation **S**tates:
```
# enforce ROA
allow from ebgp ovs valid
```

Note how the `ovs` filter requires the `roa-set {...}` to be defined;  see the `ROA` section below.

### path attributes
Besides `allow` and `deny` statements, filter rules can modify UPDATE messages, e.g.
```
# Scrub normal and large communities relevant to our ASN from EBGP neighbors
# https://tools.ietf.org/html/rfc7454#section-11
match from ebgp set { large-community delete $ASN:*:* }

# Honor requests to gracefully shutdown BGP sessions
# https://tools.ietf.org/html/rfc8326
match from any community GRACEFUL_SHUTDOWN set { localpref 0 }
```

Misbehaving peers can be adjusted;  for example Bird on FreeBSD is known to sometimes announce routes with incorrect `nexthop` attributes:
```
# XXX otherwise routes are installed with ::/128 nexthop
match from AS $A_ASN set { nexthop $A_remote }
```

# ROA
OpenBSD ships with [**rpki-client(8)**](http://man.openbsd.org/rpki-client.8) which nicely integrates with **bgpd**.
Since DN42 emulates an IRR WHOIS service through the registry repository instead of providing an RPKI repository, this tool cannot be used.

Instead, a shell script parses route objects from the registry repository and generates a `roa-set {...}` block that is to be included in the main configuration file.

One single `roa-set` may be defined, against which **bgpd** will validate the origin of each prefix;  this allows filter rules to use the `ovs` keyword as demonstrated above.

`/etc/dn42.roa-set` is the generated set:
```
roa-set {
    fd00:12:34::/48 source-as 4242421234
    fd00:ab:cd::/44 maxlen 64 source-as 4242427890
    ...
}
```

Include it in `/etc/bgpd.conf`:
```
# defines roat-set, see _rpki-client crontab
include "/etc/dn42.roa-set"
```

# Looking glass