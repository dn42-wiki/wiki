You want to join dn42, but you don't know where to start. This guide gives general guidelines about dn42 and routing in general, but it assumes that you are knowledgeable with routing.

# Requirements

- you have at least one router running 24/7. Any Linux or BSD box can be turned into a router. If your home router runs OpenWRT, you might consider using it for dn42.
- your router is able to establish network tunnels over the Internet (GRE, OpenVPN, IPSec, Tinc...). Beware, your network operator might filter this kind of traffic, e.g. in schools or universities.
- you are generally knowledgable with networking and routing (i.e. you've heard about BGP, IGP, forwarding, and you're willing to configure a BGP router such as Quagga or Bird)

# Formalities

Don't worry, it's not as tedious as registering with a RIR ;)

## Subscribe to the mailing list

This is important, as it allows to stay up-to-date on best practices, new services, security issues...

See [Contact](/contact#contact_mailing-list) to subscribe.

## Fill in the registry

You must create several objects in the registry. The recommended method is to use the [web interface](https://io.nixnodes.net/?registry), but you may still work directly with the [monotone repository](/services/Whois#monotone).

This example assumes that your name is `<FOO>`, part of an organisation called `<FOO-ORG>` (for instance, your hackerspace).  Obviously, these should be replaced by the appropriate values in all examples below.

We will create several types of objects: **maintainer** objects, which have an associated password and allow you to authenticate so that you can edit your own objects; **person** objects, which describe people or organisations and provide contact information; and finally, all other objects, which are resources (AS number, IP subnet, DNS zone, etc).

### Create a maintainer object

Create a `mntner` object named `<FOO>-MNT`. It will be used to edit all the objects that are under your responsibility.

- choose a password, and don't forget it.  **Note:** even though the field is named `sha512-pw`, you must enter *your password* directly, *not* the sha512 of your password.
- use `DUMMY-DN42` as `admin-c` and `tech-c`. We will update this later.
- use `<FOO>-MNT` as `mnt-by`, otherwise, you won't be able to edit your maintainer object.

### Create person objects

Create a `person` object for **yourself** (not your organisation/hackerspace/whatever).

- use something like `<FOO>-DN42` as `nic-hdl`, it should end with `-DN42`.
- the `person` field is more freeform, you may use your nickname or even real name here.
- provide an email.
- you may provide additional ways of contacting you, using one or more `contact` field. For instance `xmpp:luke@theforce.net`, `irc:luke42@hackint`, `twitter: TheGreatLuke`.
- you may whish to add other fields, such as `pgp-id`, `pgp-fingerprint`, `remarks`, and so on.
- don't forget to set `mnt-by` to `<FOO>-MNT`.

You must now edit the maintainer object created earlier, to properly fill in the `admin-c` and `tech-c` fields (set them to `<FOO>-DN42`).

If you intend to register resources for an organisation (e.g. your hackerspace), you must also create an `organisation` object for your organisation:

- `organisation` is of the form `<ORG-FOO>`.
- email should be a contact address for your organisation, or maybe a mailing list (but people should be able to send email without subscribing).
- you may provide a website (`www` field).
- don't forget to set `mnt-by` to `<FOO>-MNT`, since you're managing this object on behalf of your organisation.

### Guidelines for future objects

From now on, you should use:

- `admin-c: <FOO>-DN42` and `tech-c: <FOO>-DN42` for your own resources.
- `admin-c: <ORG-FOO>` and `tech-c: <FOO>-DN42` for the resources of your organisation.
- `mnt-by: <FOO>-MNT` for all objects, so that you can edit them later.

This applies to AS numbers, network prefixes, routes, DNS records...

### Register an AS number

To register an AS number, simply create an `autnum` object.

Your AS number can be chosen arbitrarily in the dn42 ASN space, look at the `as-block` objects. The historic ASN space is around 64600-64855 and 76100-76200. Starting from June 2014, **you must allocate your AS number in the new 4242420000-4242423999 range**.

For a list of currently assigned AS numbers, see http://ix.ucis.nl/dn42/as.php. This list is automatically built from the registry.

If you intend to use an ASN outside of the native dn42 ranges, please check that it doesn't clash with the [Freifunk AS-Numbers] (http://wiki.freifunk.net/AS-Nummern) or other networks (ChaosVPN, etc). For a list of ASN currently announced in dn42, see [this map](http://nixnodes.net/dn42/graph/) or [this list](http://dataviz.polyno.me/lastseen/).

If unsure, ask on the mailing list or IRC.

### Register a network prefix

To register an IPv4 network prefix, simply create an `inetnum` object.

You may choose your network prefix in one of the currently open netblocks. You can get a list of unassigned subnets on the following sites, please mind the allocation guideline below.

 * http://core1.darmstadt.ccc.de/free/data.json
 * [Open Netblocks](http://dn42.netrik.de/)
 * [graphical visualisation of the assigned ranges](http://dataviz.polyno.me/dn42-netblock-visu/registry.html).

| Size | Comment                  |
|-----:|:-------------------------|
| /24  | are you an organization? |
| /25  | still a lot of IPs!      |
| /26  | usually enough           |
| **/27**  | **default allocation**       |
| /28  | usually enough           |
| /29  | starter pack             |

The current guideline is to allocate a /27 or smaller by default, keeping space for up to a /26 if possible. Don't allocate more than a /25 worth of addresses and please **think before you allocate**: If you are going to have 2-3 servers and two VPN-spaces, a /28 is enough to suit your needs. Same will go for most home-networks. This is not public internet, but our IPv4-space is valuable too! If you need a /24 or larger, please ask in the IRC chan or on the mailing list.

For example, if there is no /27 free, you can split up a /26 into two /27. If you are looking for a /27 but there are none showing in the Open Netblocks tool, instead pick one of the /26 and click Take it!
When registering your inetnum, instead of writing 172.2x.xxx.0-172.2x.xxx.63 then you can write 172.2x.xxx.0-172.2x.xxx.31. This will get you a /27 and save our IP space for others.

To register for example 172.20.150.0/27, you need to fill in 172.20.150.0-172.20.150.31.

**Note:** Reverse DNS works with _any_ prefix length, as long as your [recursive nameserver](/services/DNS) supports [RFC 2317](https://www.ietf.org/rfc/rfc2317.txt). Don't go for a /24 _just to have RDNS_.

If you want to register an [IPv6 prefix](/FAQ#frequently-asked-questions_what-about-ipv6-in-dn42), you can create an `inet6num` object. A single /48 allocation in [ULA space](https://www.sixxs.net/tools/grh/ula/) will likely provide more than enough room for all devices you will ever connect. Some people use “vanity” prefixes like fd42:_xyz_::/48 instead of the fully standard-conformant pseudorandom ones.

[Unique Local IPv6 Generator](http://unique-local-ipv6.com/)

example: inet6num/fd42:4992:6a6d::_48
```
inet6num:          fd42:4992:6a6d:0000:0000:0000:0000:0000 - fd42:4992:6a6d:ffff:ffff:ffff:ffff:ffff
netname:           EVE-NETWORK
descr:             Network of eve
country:           DE
admin-c:           MIC92-DN42
tech-c:            MIC92-DN42
mnt-by:            MIC92-MNT
nserver:           ns1.evenet.dn42
nserver:           ns2.evenet.dn42
status:            ASSIGNED
```

example: inetnum/172.23.75.0_24
```
inetnum:           172.23.75.0 - 172.23.75.255
netname:           EVE-NETWORK
admin-c:           MIC92-DN42
tech-c:            MIC92-DN42
mnt-by:            MIC92-MNT
nserver:           ns1.evenet.dn42
nserver:           ns2.evenet.dn42
status:            ASSIGNED
```

#### Create route objects

If you plan to announce your prefixes in dn42, which you probably want in most cases, you will also need to create a `route` object for ipv4 prefixes and a `route6` object for ipv6 prefixes. This information is used for ROA checks (route origin authorization). If you skip this step, your network will probably get filtered by some peers. Many people enforce ROA checks to prevent (accidental) hijacking of other people's prefixes.

example: route6/fd42:4992:6a6d::_48
```
route6:            fd42:4992:6a6d::/48
origin:            AS4242420092
mnt-by:            MIC92-MNT
```

example route/172.23.75.0_24:
```
route:             172.23.75.0/24
origin:            AS4242420092
mnt-by:            MIC92-MNT
bgp-status:        active
```

# Get some peers

In dn42, there is no real distinction between peering and transit: in most cases, everybody serves as an upstream provider to all its peers.  Note that if you have very slow connectivity to the Internet, you may want to avoid providing transit between your peers, which can be done by filtering or prepending your ASN. For the sake of sane routing, try to peer with people on the same continent to avoid inefficient routing, <50ms is a good rule of thumb. You can also look into Bird communities if you are using Bird to mark the latency for the [link](/howto/Bird-communities).

If you don't know anybody who can peer with you, you can use this tool: https://dn42.us/peers

It will let you find people to peer with.  You can then contact them on IRC or by email.  In case you're really at loss, you can also ask for peers on the mailing list.

## Establishing tunnels

Unless your dn42 peers are on the same network, you must establish tunnels. Choose anything you like: OpenVPN, GRE, GRE + IPSec, IPIP, Tinc, ...

There is some documentation in this wiki, like [gre-plus-ipsec](GRE-plus-IPsec).

## Running a routing daemon

You need a routing daemon to speak BGP with your peers. People usually run Quagga or Bird, but you may use anything (OpenBGPD, XORP, somebody even used an old [hardware router](bgp-on-extreme-summit1i) ).  See the relevant [FAQ entry](/FAQ#frequently-asked-questions_what-bgp-daemon-should-i-use).

You can find [configuration examples for Bird here](bird).

Some [documentation of the old wiki] (http://dn42.volcanis.me/initenv/wiki/HowToPeer.html) might still be handy, but remember that everything there is terribly outdated.

## Configuration Examples

* VPN/Tunnel:
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
* [Important Network configuration](networksettings)


# Configure DNS

See [Services DNS](/Services/DNS).

# Use and provide services

See [internal](/internal) for internal services.

Don't hesitate to provide interesting services, but *please*, document them on the wiki! Otherwise, nobody will use them because nobody can guess they even exist.
