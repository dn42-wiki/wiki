# Getting Started

This guide walks you through joining dn42. It provides general guidelines about dn42 and routing, but assumes you have a working knowledge of networking concepts.

## Requirements

- A router running 24/7. Any Linux or BSD box can serve as a router. If your home router runs OpenWRT, you can use it for dn42.
- The ability to establish network tunnels over the Internet (WireGuard, GRE, OpenVPN, IPsec, Tinc, etc.). Note that some network operators filter tunnel traffic, particularly in schools or universities.
- Familiarity with networking and routing concepts (BGP, IGP, forwarding) and willingness to configure a BGP daemon such as BIRD or FRR.

## Formalities

Don't worry, it's not as tedious as registering with a RIR.

## Subscribe to the mailing list

Subscribing keeps you informed about best practices, new services, and security issues. See [Contact](/contact#contact_mailing-list) to subscribe.

## Fill in the registry

You must create several objects in the DN42 registry: <https://git.dn42.dev/dn42/registry>

The registry is a git repository. To create objects, fork the main repository, make your changes, and submit a pull request for review. Detailed instructions are available in the [README](https://git.dn42.dev/dn42/registry/src/branch/master/README.md). See also the [git documentation](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes) and [GitHub guides](https://help.github.com/en/github/using-git) for working with remote repositories.

When filling out registry objects, refer to the [schema](https://explorer.dn42.dev/#/schema) to speed up the review process.

When submitting your pull request, you must squash multiple changes into a single commit (instructions are in the [README](https://git.dn42.dev/dn42/registry/src/branch/master/README.md)). Remember to add authentication to your `mntner` object and [sign your commit](/howto/Registry-Authentication).

### Validation scripts

The registry includes scripts to help check your request:

| Script | Purpose |
|--------|---------|
| `fmt-my-stuff <FOO>-MNT` | Automatically fixes minor formatting errors |
| `check-my-stuff <FOO>-MNT` | Validates your objects against the registry schema |
| `check-pol origin/master <FOO>-MNT` | Checks for policy violations |
| `squash-my-commits` | Automatically updates and squashes your local commits |
| `sign-my-commit` | Signs your commit using a PGP key or SSH signing |

Registry maintainers run these scripts against each request, so please run them yourself first to catch simple errors.

Browse the registry and the [pull request queue](https://git.dn42.dev/dn42/registry/pulls) to see examples, understand the process, and see the types of questions maintainers ask.

**Note:** Do not use the Gitea web interface to edit files. Doing so creates multiple commits and prevents the registry scripts from running properly.

## Creating registry objects

This example assumes your name is `<FOO>`, part of an organisation called `<ORG-FOO>` (e.g., your hackerspace). Replace these placeholders with appropriate values throughout. Organisation objects are optional if you're registering as an individual.

You will create several types of objects:

- **Maintainer objects** (`mntner`): Authenticated objects that ensure only you can edit your own records
- **Person objects** (`person`): Describe individuals or organisations and provide contact information
- **Resource objects**: AS numbers, IP subnets, DNS zones, etc.

All objects are plain text files in specific subfolders. Files must use spaces (not tabs), and attribute values must start at column 20.

### Create a maintainer object

Create a `mntner` object in `data/mntner/` named `<FOO>-MNT`. This object controls editing permissions for all objects under your responsibility.

- Set `mnt-by` to `<FOO>-MNT` so you can edit your own maintainer object.
- Add an `auth` attribute so changes to your objects can be verified.

See [registry authentication](/howto/Registry-Authentication) for details on authentication methods and commit signing.

Common authentication methods:

- PGP key: `auth: pgp-fingerprint <fingerprint>`
- SSH key: `auth: ssh-{rsa,ed25519} <key>`

Example: `data/mntner/FOO-MNT`

```
mntner:             FOO-MNT
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
auth:               pgp-fingerprint 0123456789ABCDEF0123456789ABCDEF01234567
source:             DN42
```

### Create a person object

Create a `person` object in `data/person/` for yourself (not your organisation).

- Set `nic-hdl` to something like `<FOO>-DN42` (must end with `-DN42`).
- The `person` field is freeform - use your nickname or real name.
- Provide an email address.
- Optionally add `contact` fields for other contact methods (e.g., `xmpp:luke@theforce.net`, `irc:luke42@hackint`).
- Optionally add fields like `pgp-fingerprint` or `remarks`.
- Set `mnt-by` to `<FOO>-MNT`.

> **Privacy note:** Contact attributes are optional, but dn42 is a dynamic network and being able to reach users is important when issues arise. Be aware that the DN42 registry is public. Any details you provide will be visible and cannot be fully removed. If this concerns you, provide anonymous details specific to DN42 or omit them entirely. Please do not provide bogus contact information.

Example: `data/person/FOO-DN42`

```
person:             John Doe
e-mail:             john.doe@example.com
nic-hdl:            FOO-DN42
mnt-by:             FOO-MNT
source:             DN42
```

### Create an organisation object (optional)

Organisation objects are not required if you're joining as an individual.

If you're registering resources for an organisation (e.g., your hackerspace), create an `organisation` object:

- Set `organisation` in the format `<ORG-FOO>`.
- Set `org-name` to your organisation's name.
- Set `e-mail` to a contact address or mailing list (people should be able to send email without subscribing).
- Set `admin-c`, `tech-c`, and `abuse-c` to point to responsible `person` objects.
- Optionally add a `www` field for your website.
- Set `mnt-by` to `<FOO>-MNT`.

Example: `data/organisation/ORG-FOO`

```
organisation:       ORG-FOO
org-name:           Foo Organisation
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
source:             DN42
```

### Guidelines for resource objects

For all resource objects (AS numbers, network prefixes, routes, DNS records), use:

- `admin-c: <FOO>-DN42` and `tech-c: <FOO>-DN42` for personal resources
- `admin-c: <FOO>-DN42`, `tech-c: <FOO>-DN42`: `org: <ORG-FOO>` for organisation resources
- `mnt-by: <FOO>-MNT` for all objects

### Register an AS number

Create an `aut-num` object in `data/aut-num/`. Set `as-name` to a name for your AS.

Choose your AS number from the dn42 ASN space (see [as-block objects](https://git.dn42.dev/dn42/registry/src/master/data/as-block)). **Allocate your AS number in the 4242420000–4242423999 range.**

Use [dn42regsrv](https://explorer.burble.com/free#/asn) to find free ASNs, or browse the [aut-num directory](https://explorer.burble.com/#/aut-num/).

If using an ASN outside native dn42 ranges, verify it doesn't conflict with [Freifunk AS numbers](http://wiki.freifunk.net/AS-Nummern) or other networks (ChaosVPN, etc.).

Internet ASNs may be used, but you must clearly separate Internet and DN42 routes to prevent leaks. For Internet ASNs, set the `source` attribute to the originating registry and be prepared to prove ownership. If unsure, ask on the mailing list or IRC.

Example: `data/aut-num/AS4242423999`

```
aut-num:            AS4242423999
as-name:            AS-FOO-DN42
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
source:             DN42
```

### Register a network prefix

#### IPv6

Create an `inet6num` object. dn42 uses the fd00::/8 ([ULA](https://tools.ietf.org/html/rfc4193)) range. A single /48 allocation is typical and provides more than enough addresses for most use cases. The smallest announceable prefix is /64.

Since dn42 interconnects with other networks (like ICVPN) that also use ULA space, registry allocation cannot prevent IPv6 conflicts. Use a fully random prefix per [RFC 4193](https://tools.ietf.org/html/rfc4193). Renumbering after discovering a conflict is painful.

Tools for generating random ULA prefixes:

- [dn42regsrv](https://explorer.burble.com/free#/6)
- [SimpleDNS](https://simpledns.com/private-ipv6)
- [Ultratools](https://www.ultratools.com/tools/rangeGenerator)
- [ulagen.py script](https://git.dn42.dev/netravnen/dn42-repo-utils/src/master/ulagen.py)

Example: `data/inet6num/fd35:4992:6a6d::_48`

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

#### IPv4

Create an `inetnum` object. Choose your prefix from an open netblock, following the allocation guidelines below.

Tools for finding free blocks:

- [dn42regsrv free blocks](https://explorer.burble.com/free#/4)
- [Open Netblocks](https://dn42.us/peers/free)

If no free subnets of your desired size exist, you may split a larger block. Check `data/inetnum` to ensure your chosen prefix is unassigned, and verify the parent block has an 'open' policy (`grep "^policy" data/inetnum/*`).

#### Allocation guidelines

| Size | Guidance |
|-----:|:---------|
| /29 | Minimum allocation |
| /28 | Usually sufficient |
| **/27** | **Default allocation** |
| /26 | Usually sufficient |
| /25 | Maximum without justification |

The default allocation is /27 or smaller, with room to expand to /26 if needed. Do not allocate more than /25 without justification.

dn42 typically uses point-to-point addressing for VPN tunnels, so a single IP per host is usually sufficient. For 2–3 servers, a /28 is plenty. Prefixes smaller than /29 are not permitted.

For /24 or larger, ask on IRC or the mailing list and provide justification.

> **Note:** Reverse DNS works with any prefix length as long as your [recursive nameserver](/services/DNS) supports [RFC 2317](https://www.ietf.org/rfc/rfc2317.txt). Don't request a /24 solely for reverse DNS.

Example: `data/inetnum/172.20.150.0_27`

```
inetnum:            172.20.150.0 - 172.20.150.31
cidr:               172.20.150.0/27
netname:            FOO-NETWORK
descr:              Network of FOO
country:            XD
admin-c:            FOO-DN42
tech-c:             FOO-DN42
mnt-by:             FOO-MNT
status:             ASSIGNED
source:             DN42
```

### Create route objects

To announce your prefixes in dn42, create route objects for Route Origin Authorization (ROA) checks. Without these, most peers will filter your announcements. ROA prevents accidental prefix hijacking.

Create a `route6` object for IPv6 prefixes:

Example: `data/route6/fd35:4992:6a6d::_48`

```
route6:             fd35:4992:6a6d::/48
origin:             AS4242423999
max-length:         48
mnt-by:             FOO-MNT
source:             DN42
```

Create a `route` object for IPv4 prefixes:

Example: `data/route/172.20.150.0_27`

```
route:              172.20.150.0/27
origin:             AS4242423999
max-length:         27
mnt-by:             FOO-MNT
source:             DN42
```

> **Note:** Set `max-length` to match your prefix length (27 for default IPv4, 48 for default IPv6) unless you have specific needs for announcing larger prefixes.

### Register a domain (optional)

Create a `dns` object in `data/dns/`. Domain names and `nserver` attributes must be lowercase.

Example: `data/dns/foo.dn42`

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

For DNSSEC, add `ds-rdata` attributes:

```
ds-rdata:           61857 13 2 bd35e3efe3325d2029fb652e01604a48b677cc2f44226eeabee54b456c67680c
```

For reverse DNS, add `nserver` attributes to your `inet6num` or `inetnum` objects:

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

## Find peers

In dn42, there's no strict distinction between peering and transit. Most participants provide upstream connectivity to all their peers. If you have slow Internet connectivity, you may want to avoid providing transit by filtering or prepending your ASN.

For efficient routing, peer with others on the same continent. A latency under 50 ms is a good guideline. If using BIRD, you can use [BGP communities](/howto/BGP-communities) to mark link latency.

Use the [Peerfinder](https://peerfinder.dn42.dev/) to find potential peers near you, then contact them via IRC or email. You can also request peers on the mailing list.

## Establish tunnels

Unless your peers are on the same local network, you'll need tunnels. Choose any protocol you prefer: WireGuard, OpenVPN, GRE, GRE + IPsec, IPIP, Tinc, etc.

See [GRE + IPsec](/howto/GRE-plus-IPsec) and other documentation in this wiki.

## Run a routing daemon

You need a BGP daemon to exchange routes with peers. Common choices are BIRD and FRR, but you can use anything: OpenBGPD, XORP, ExaBGP. See the [FAQ](/FAQ#frequently-asked-questions_what-bgp-daemon-should-i-use) for guidance.

See [BIRD configuration examples](/howto/Bird2).

## Configuration examples

### General

- [Network configuration](/howto/networksettings)

### VPN/Tunnels

- [WireGuard](/howto/wireguard)
- [OpenVPN](/howto/openvpn)
- [Tinc](/howto/tinc)
- [IPsec with public keys](/howto/IPsec-with-PublicKeys)

### BGP daemons

- [BIRD](/howto/Bird2)
- [FRR](/howto/frr)
- [OpenBGPD](/howto/OpenBGPD)

### Router-specific

- [OpenWRT](/howto/OpenWRT)
- [EdgeOS configuration](/howto/EdgeOS-Config-Example)
- [EdgeOS GRE/IPsec](/howto/EdgeOS-GRE-IPsec-Example)
- [Extreme Networks Summit 1i](/howto/BGP-on-Extreme-Summit1i)

## Configure DNS

See [DNS services](/services/DNS).

## Use and provide services

See [internal services](/internal/Internal-Services) for available services.

If you provide a service, please document it on the wiki, otherwise nobody will know it exists!