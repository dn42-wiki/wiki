# Whois registry
**aka** _The registry_ contains:

  * AS numbers assignations
  * Subnet assignations
  * DNS root zone for `dn42.`

# Names and numbers

dn42 uses some names and numbers, which are declared in the registry.  Whenever possible, we try to stick to names and numbers that do not conflict with the ICANN-net or other networks similar to dn42, for instance by using private numbers space.

## Address space

dn42 uses **172.22.0.0/15** for IPv4.

For IPv6, we use both ULA (that is, **fd00::/8**) and globally unique PI/PA address space of participants. ULA is prefered for various reasons, see the [FAQ](/FAQ#frequently-asked-questions_what-about-ipv6-in-dn42).

## AS numbers

Since June 2014, dn42 is using the **4242420000-4242429999** ASN range for allocations. This range is further subdivided:
* **4242420000-4242423999** for end-users allocations
* **4242424000-4242426999** reserved for future use
* **4242427000-4242429999** for sub-allocations

If you are running a project similar to dn42, please use another range of ASN. The "sub-allocations" range is meant for dn42 users willing to have administrative control over a small, consecutive range of ASN (e.g. to use them directly or to distribute them).

Note that currently, most AS are using one of the legacy ASN range (and will probably continue to do so, as renumbering is painful). See the [FAQ](/FAQ#frequently-asked-questions_why-are-you-using-asn-in-the-76100-76199-range) for a discussion on AS ranges.

## DNS zones

dn42 uses the `dn42.` TLD, which is not present in the root DNS zone of the ICANN-net.  For details, see [[DNS]].

Note that other TLDs should also be usable from dn42, most notably from Freifunk and ChaosVPN. A tentative list is available at [External DNS](/services/dns/External-DNS).

# Web interface

Nixnodes provides a nice web interface, that allows you to **add/edit records** easily.  It is available at https://io.nixnodes.net/?registry. A full guide is available at [Getting started](/howto/Getting-started#Fill-in-the-registry).

## Authentication

To add or edit records with the web interface, authentication is done thanks to **maintainer objects**. Each maintainer object has a password associated to it.

The password are not stored in cleartext in the registry: a hash is computed from the password and the name of the maintainer object. To generate such a hash (e.g. in case you forgot your password), use https://io.nixnodes.net/nctlio.php?m=dnr&gen=mypassword&mnt=MYMAINTAINER-MNT

## Misc

A read-only interface is also available at http://ix.ucis.dn42/dn42/ ([public](http://ix.ucis.nl/dn42/) or 172.22.166.3). The used PHP scripts are available from UFO a.k.a. Ivo at request.

# DNS interface

There is also a DNS-based interface to query AS information from the registry. The DNS zone is `asn.dn42`. Example:

    $ dig +short AS76103.asn.dn42 TXT
    "76103 | DN42 | dn42 |  | NIXNODES-IX - NixNodes CORE Network"

The Python code for generating the zone from the registry is available on the monotone repository.

The idea comes from the guys at cymru.com, who provide this service for the Internet (e.g. `AS1.asn.cymru.com`), see https://www.team-cymru.org/Services/ip-to-asn.html#dns

# Address space

There is nice 3djs visualisation showing current address space usage: http://dataviz.polynome.dn42/dn42-netblock-visu/registry.html ([public](http://109.24.208.244:8888/dn42-netblock-visu/registry.html) or 172.23.184.98). The input data is taken from the registry.

Another visualisation shows the prefixes seen by BGP: http://dataviz.polynome.dn42/dn42-netblock-visu/index.html ([public](http://109.24.208.244:8888/dn42-netblock-visu/index.html) or 172.23.184.98).

# Software

 * [[lglass]] is a python implementation for working with the registry. It features a whois server, tools to manipulate the data (DNS zone generation, etc).

# Whois daemons

We have anycast IPv4 and IPv6, both reachable under whois.dn42. IPs are 172.22.0.43 respective fd42:d42:d42:43::1. Please consider joining these anycast-adresses when you setup your server. Updates every 1 hour would be nice for a start.

| **person**  | **dns**                   | **ip**          |
|------------|---------------------------|-----------------|
| welterde   | thinkbase.srv.welterde.de | 46.4.248.201    |
| fritz      | whois.fritz.dn42          | 172.22.119.139  |
| nixnodes   | whois.nixnodes.dn42       | 172.22.177.77   |
| prauscher  | sheldon.prauscher.dn42    | 172.22.120.1    |

## Usage
```sh
whois -h $host $query
```

## Using a whois config

```sh
$ cat /etc/whois.conf 
\.dn42$           whois.dn42
\-DN42$           whois.dn42
# dn42 range 64512-65534
^as6(4(5(1[2-9]|[2-9][0-9])|[6-9][0-9]{2})|5([0-4][0-9]{2}|5([0-2][0-9]|3[0-4])))$ whois.dn42
# dn42 range 76100-76199
^as761[0-9][0-9]$   whois.dn42
# dn42 range 4242420000-4242429999
^as424242[0-9]{4}$ whois.dn42
# dn42 ipv4 address space
^172\.2[2-3]\.[0-9]{1,3}\.[0-9]{1,3}(/(1[56789]|2[0-9]|3[012]))?$ whois.dn42

# dn42 ula ipv6 address space
fd**:****:****:****:****:****:****:**** whois.dn42

```

You can then use whois without specifying the server. Works at least with Marco d'Itri's whois client.

## Running your own whoisd
```sh
cd /home/some/path/to/store/branch
sudo aptitude install ruby rubygems
sudo gem install netaddr
cd whoisd/ruby
sudo ruby whoisd.rb nobody
```

# Monotone
Monotone is an distributed revision control system. Monotone tracks revisions to files, groups sets of revisions into changesets, and tracks history across renames. The design principle is distributed operation making heavy use of cryptographic primitives to track file revisions (via the SHA-1 secure hash) and to authenticate user actions (via RSA cryptographic signatures). Each participant maintains their own revision history store in a local SQLite database. Monotone is especially strong in its support of a diverge/merge workflow, which it achieves in part by always allowing commit before merge. Revisions are exchanged using the custom netsync protocol which shares some conceptual ground with rsync and cvs.
 * [Website](http://monotone.ca/)
 * [Tutorial](http://monotone.ca/docs/Tutorial.html)

## Monotone servers

| Person   | Address                                | Status |
|----------|----------------------------------------|--------|
| crest    | mtn.crest.dn42                         | UP     |
| dracoling | dn42.smrsh.net (net.smrsh.dn42)       | UP     |
| siska    | mtn.nixnodes.net / mtn.nixnodes.dn42 (172.22.177.77) | UP |
| xuu      | mtn.xuu.dn42 (172.22.141.131)          | UP     |  
| zorun    | mtn.polyno.me / mtn.polynome.dn42 (172.23.184.71)| UP |
| Nurtic-Vibe | mtn.grmml.dn42 (172.23.149.20)| UP |

## Monotone branches
 * net.dn42.registry: Contains the registry and some related code

## Client setup
```sh
mtn genkey you@domain.tld
mtn pubkey you@domain.tld # send the output to some $monotone_server operator (do NOT send the keypair!)
mtn clone 'mtn://$monotone_server/?net.dn42.*' --branch net.dn42.registry
cd net.dn42.registry
$add_your_objects
mtn add --unknown
mtn ci -k you@domain.tld
mtn sync
```

## Server setup

Debian has a package "monotone-server", with config located in "/etc/monotone".

### Allowing somebody to write to a monotone server

If you want to allow somebody else to write to your monotone server (for instance for somebody to sync with you), you first need to import their key, here on Debian:

    mtn --db /var/lib/monotone/default.mtn read < pubkey

Then edit the file `write-permissions` (`/etc/monotone/write-permissions` on Debian) to add the email address associated with the public key.

References: http://www.monotone.ca/docs/Basic-Network-Service.html#Basic-Network-Service and https://geti2p.net/en/get-involved/guides/monotone#obtaining-and-deploying-developers-keys

### Tips and tricks

Pro-tip: monotone seems to use `SO_V6ONLY`, which is annoying. To bind to both IPv4 and IPv6, use `ADDRESS=":: --bind 0.0.0.0"` in `/etc/default/monotone`.