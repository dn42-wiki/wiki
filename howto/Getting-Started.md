You want to join dn42, but you don't know where to start. This guide gives general guidelines about dn42 and routing in general, but it assumes that you are knowledgeable with routing.

# Requirements

- you have at least one router running 24/7. Any Linux or BSD box can be turned into a router. If your home router runs OpenWRT, you might consider using it for dn42.
- your router is able to establish network tunnels over the Internet (Wireguard, GRE, OpenVPN, IPSec, Tinc...). Beware, your network operator might filter this kind of traffic, e.g. in schools or universities.
- you are generally knowledgeable with networking and routing (i.e. you've heard about BGP, IGP, forwarding, and you're willing to configure a BGP router such as Quagga or Bird)

# Formalities

Don't worry, it's not as tedious as registering with a RIR ;)

## Subscribe to the mailing list

This is important, as it allows to stay up-to-date on best practices, new services, security issues...

See [Contact](/contact#contact_mailing-list) to subscribe.

## Fill in the registry

You must create several objects in the DN42 registry: <https://git.dn42.dev/dn42/registry>

The registry is a git repository, objects are created by creating a branch in the main repository, making your changes and then submitting a pull request for review. There are detailed instructions in the registry [README](https://git.dn42.dev/dn42/registry/src/branch/master/README.md) how to do this. See also the the generic git documentation [git documentation](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes) and guides on [github](https://help.github.com/en/github/using-git) for how to use git to work with remote repositories.

When submitting your pull request, please squash your commits, again there are instructions in the [README](https://git.dn42.dev/dn42/registry/src/branch/master/README.md) for how to do this. 

Remember to add authentication to your `mntner` object, and [sign your commit](/howto/Registry-Authentication)

The registry includes a number of scripts to help check your request:

 - `fmt-my-stuff <FOO>-MNT`: automatically fixes minor formatting errors
 - `check-my-stuff <FOO>-MNT`: validates your objects against the registry schema
 - `check-pol origin/master <FOO>-MNT`: checks for policy violations

The registry maintainers run all three scripts against each request, so please run these yourself first to check for simple errors. 

Do browse through the registry and look at the [pull request queue](https://git.dn42.dev/dn42/registry/pulls) to see examples, understand how the process works and see the types of questions asked by the registry maintainers.

*Whilst it is possible to use the web interface to edit files, you are encouraged to clone your repo locally and use the command line git tools. It's easy to do and learning how to use git is a skill worth knowing. Using the web interface creates a large number of commits and prevents you from checking your changes with the registry scripts*

---

This example assumes that your name is `<FOO>`, part of an organisation called `<FOO-ORG>` (for instance, your hackerspace). *Organisation objects are not required if your are registering as an individual*. Obviously, these should be replaced by the appropriate values in all examples below.

We will create several types of objects: 
 - **maintainer** objects, which are authenticated so that only you can edit your own objects
 - **person** objects, which describe people or organisations and provide contact information
 - and **resource** objects (AS number, IP subnet, DNS zone, etc).

All objects are simple text files in the specific subfolders, but the files do have a particular format. The files should use spaces and not tabs, and the attribute values must start on the 20th column. 

### Create a maintainer object

Create a `mntner` object in `data/mntner/` named `<FOO>-MNT`. It will be used to edit all the objects that are under your responsibility.

- use `<FOO>-MNT` as `mnt-by`, otherwise, you won't be able to edit your maintainer object.
- Add an 'auth' attribute so that changes to your objects can be verified.  

The `auth` attribute is used to verify changes to your object. There is a separate page on [registry authentication](/howto/Registry-Authentication) which details what to include in your mntner object, how to sign and verify your commits. 

Common authentication methods are:
  - PGP Key: `auth: pgp-fingerprint <pgp-fingerprint>`
  - SSH Key: `auth: ssh-{rsa,ed25519} <key>`

Example: data/mntner/FOO-MNT
```
mntner:             FOO-MNT
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
auth:               pgp-fingerprint 0123456789ABCDEF0123456789ABCDEF01234567
source:             DN42
```

### Create person objects

Create a  `person` object in `data/person/` for **yourself** (not your organisation/hackerspace/whatever).

- use something like `<FOO>-DN42` as `nic-hdl`, it should end with `-DN42`.
- the `person` field is more freeform, you may use your nickname or even real name here.
- provide an email.
- you may provide additional ways of contacting you, using one or more `contact` field. For instance `xmpp:luke@theforce.net`, `irc:luke42@hackint`, `twitter: TheGreatLuke`.
- you may wish to add other fields, such as `pgp-fingerprint`, `remarks`, and so on.
- don't forget to set `mnt-by` to `<FOO>-MNT`.

Example: data/person/FOO-DN42
```
person:             John Doe
e-mail:             john.doe@example.com
nic-hdl:            FOO-DN42
mnt-by:             FOO-MNT
source:             DN42
```

---

*(Optional)*  
**Organisations are not required if you are joining dn42 as an individual**

If you intend to register resources for an organisation (e.g. your hackerspace), you must also create an `organisation` object for your organisation:

- `organisation` is of the form `<ORG-FOO>`.
- `org-name` should be the name of your organisation.
- `e-mail` should be a contact address for your organisation, or maybe a mailing list (but people should be able to send email without subscribing).
- `admin-c`, `tech-c`, and `abuse-c` may point to `person` objects responsible for the respective role in your organisation.
- you may provide a website (`www` field).
- don't forget to set `mnt-by` to `<FOO>-MNT`, since you're managing this object on behalf of your organisation.

Example: data/organisation/ORG-EXAMPLE
```
organisation:       ORG-FOO
org-name:           Foo Organisation
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
source:             DN42
```

### Guidelines for future objects

From now on, you should use:

- `admin-c: <FOO>-DN42` and `tech-c: <FOO>-DN42` for your own resources.
- `admin-c: <FOO>-DN42`, `tech-c: <FOO>-DN42` and `org: <ORG-FOO>` for the resources of your organisation.
- `mnt-by: <FOO>-MNT` for all objects, so that you can edit them later.

This applies to AS numbers, network prefixes, routes, DNS records...

### Register an AS number

To register an AS number, simply create an `aut-num` object in `data/aut-num/`.  
`as-name` should be a name for your AS.

Your AS number can be chosen arbitrarily in the dn42 ASN space, see the [as-block objects](https://git.dn42.dev/dn42/registry/src/master/data/as-block) in the registry. 

**You should allocate your AS number in the 4242420000-4242423999 range**

For a list of currently assigned AS numbers browse the registry data/aut-num/ directory or [online](https://explorer.burble.com/#/aut-num/). 

If you intend to use an ASN outside of the native dn42 ranges, please check that it doesn't clash with the [Freifunk AS-Numbers] (http://wiki.freifunk.net/AS-Nummern) or other networks (ChaosVPN, etc). For a list of ASN currently announced in dn42, see [this map](http://nixnodes.net/dn42/graph/).

If unsure, ask on the mailing list or IRC.

Example: data/aut-num/AS4242423999
```
aut-num:            AS4242423999
as-name:            AS for FOO Network
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
source:             DN42
```

### Register a network prefix

#### IPv6

To register an IPv6 prefix, you create an `inet6num` object. dn42 uses the fd00::/8 ([ULA](https://tools.ietf.org/html/rfc4193)) range. A single /48 allocation is typical and will likely provide more than enough room for all devices you will ever connect. 

dn42 is interconnected with other networks, like icvpn, which also use the same ULA range so a registration in the dn42 registry can't prevent IPv6 conflicts. A fully random prefix (see [RFC4193](https://tools.ietf.org/html/rfc4193)) is recommended; finding a conflict and needing to renumber your network is no fun. 

A few websites can generate random ULA prefixes for you:
* [SimpleDNS](https://simpledns.com/private-ipv6)
* [Ultratools](https://www.ultratools.com/tools/rangeGenerator)

or a small script is available: [ulagen.py](https://git.dn42.dev/netravnen/dn42-repo-utils/src/master/ulagen.py)

example: data/inet6num/fd35:4992:6a6d::_48
```
inet6num:           fd35:4992:6a6d:0000:0000:0000:0000:0000 - fd35:4992:6a6d:ffff:ffff:ffff:ffff:ffff
cidr:               fd35:4992:6a6d::/48
netname:            FOO-NETWORK
descr:              Network of FOO
country:            XD
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
status:             ASSIGNED
source:             DN42
```

#### IPv4 (Legacy)

If you also want to register an IPv4 network prefix, simply create an `inetnum` object. 

You may choose your network prefix in one of the currently open netblocks. You can get a list of unassigned subnets on the following site, please mind the allocation guideline below.

 * [Open Netblocks](https://dn42.us/peers/free)

Check the registry (data/inetnum) to make sure no-one else has allocated the same prefix. There are some IP ranges that are not open for assignments or are reserved for specific uses, so you should also check that the parent block has an 'open' policy. A quick and simple way to see the block policies is to run `grep "^policy" data/inetnum/*`. 

| Size | Comment                  |
|-----:|:-------------------------|
| /29  | starter pack             |
| /28  | usually enough           |
| **/27**  | **default allocation**       |
| /26  | usually enough           |
| /25  | still a lot of IPs!      |
| /24  | are you an organization? |

The current guideline is to allocate a /27 or smaller by default, keeping space for up to a /26 if possible. Don't allocate more than a /25 worth of addresses and please **think before you allocate**. 

dn42 typically uses point-to-point addressing in VPN tunnels making transit network unnecessary, a single IP address per host should be sufficient. If you are going to have 2-3 servers, a /28 is plenty; same will go for most home-networks. dn42 is not the public internet, but our IPv4-space is valuable too! 

If you need a /24 or larger, please ask in the IRC chan or on the mailing list and expect to provide justification. You should also ensure the range you've requested is in a suitable block.

**Note:** Reverse DNS works with _any_ prefix length, as long as your [recursive nameserver](/services/DNS) supports [RFC 2317](https://www.ietf.org/rfc/rfc2317.txt). Don't go for a /24 _just to have RDNS_.

example: data/inetnum/172.20.150.0_27
```
inetnum:            172.20.150.0 - 172.20.150.31
cidr:               172.20.150.0/27
netname:            FOO-NETWORK
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
status:             ASSIGNED
source:             DN42
```

#### Create route objects

If you plan to announce your prefixes in dn42, which you probably want in most cases, you will also need to create a `route6` object for ipv6 prefixes and a `route` object for ipv4 prefixes. This information is used for Route Origin Authorization (ROA) checks. If you skip this step, your network will probably get filtered by most major peers.  Checking ROA will prevent (accidental) hijacking of other people's prefixes.

example: data/route6/fd35:4992:6a6d::_48
```
route6:             fd35:4992:6a6d::/48
origin:             AS4242423999
max-length:         48
mnt-by:             FOO-MNT
source:             DN42
```

example data/route/172.20.150.0_27:
```
route:              172.20.150.0/27
origin:             AS4242423999
mnt-by:             FOO-MNT
source:             DN42
```

#### DNS and Domain Registration

*(Optional)*  
To register a domain name, create a `dns` object in the data/dns directory.

example: data/dns/foo.dn42
```
domain:             foo.dn42
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
nserver:            ns1.foo.dn42 172.20.150.1
nserver:            ns1.foo.dn42 fd35:4992:6a6d:53::1
nserver:            ns2.foo.dn42 172.20.150.2
nserver:            ns2.foo.dn42 fd35:4992:6a6d:53::2
source:             DN42
```

You can also add DNSSEC delegations using `ds-rdata` attributes to your domain:

```
ds-rdata:           61857 13 2 bd35e3efe3325d2029fb652e01604a48b677cc2f44226eeabee54b456c67680c
```

For reverse DNS, add `nserver` attributes to you inet{,6}num objects:

```
inet6num:           fd35:4992:6a6d:0000:0000:0000:0000:0000 - fd35:4992:6a6d:ffff:ffff:ffff:ffff:ffff
cidr:               fd35:4992:6a6d::/48
netname:            FOO-NETWORK
descr:              Network of FOO
country:            XD
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
status:             ASSIGNED
nserver:            ns1.foo.dn42
nserver:            ns2.foo.dn42
source:             DN42
```

# Get some peers

In dn42, there is no real distinction between peering and transit: in most cases, everybody serves as an upstream provider to all its peers.  Note that if you have very slow connectivity to the Internet, you may want to avoid providing transit between your peers, which can be done by filtering or prepending your ASN. For the sake of sane routing, try to peer with people on the same continent to avoid inefficient routing, <50ms is a good rule of thumb. You can also look into Bird communities if you are using Bird to mark the latency for the [link](/howto/Bird-communities).

You can use the peerfinder to help you find potential peers close to you: https://dn42.us/peers

You can then contact them on IRC or by email. In case you're really at loss, you can also ask for peers on the mailing list.

## Establishing tunnels

Unless your dn42 peers are on the same network, you must establish tunnels. Choose anything you like: Wireguard, OpenVPN, GRE, GRE + IPSec, IPIP, Tinc, ...

There is some documentation in this wiki, like [gre-plus-ipsec](GRE-plus-IPsec).

## Running a routing daemon

You need a routing daemon to speak BGP with your peers. People usually run Quagga or Bird, but you may use anything (OpenBGPD, XORP, somebody even used an old [hardware router](bgp-on-extreme-summit1i) ).  See the relevant [FAQ entry](/FAQ#frequently-asked-questions_what-bgp-daemon-should-i-use).

You can find [configuration examples for Bird here](bird).

## Configuration Examples

* [Important Network configuration](networksettings)

* VPN/Tunnel:
  * [Wireguard](/howto/wireguard)
  * [Openvpn](/howto/openvpn)
  * [Tinc](/howto/tinc)
  * [IPsec with public key authentication](/howto/IPsec-with-PublicKeys)
* BGP:
  * [Bird](/howto/Bird)
  * [Quagga](/howto/Quagga)
* Router specific:
  * [dn42 on OpenWRT](OpenWRT)
  * [EdgeOS Configuration](EdgeOS-Config-Example)
  * [EdgeOS GRE/IPsec Example](EdgeOS-GRE-IPsec-Example)
  * [BGP on Extreme Networks Summit 1i](BGP-on-Extreme-Summit1i)

# Configure DNS

See [Services DNS](/Services/DNS).

# Use and provide services

See [internal](/internal/Internal-Services) for internal services.

Don't hesitate to provide interesting services, but *please*, document them on the wiki! Otherwise, nobody will use them because nobody can guess they even exist.
