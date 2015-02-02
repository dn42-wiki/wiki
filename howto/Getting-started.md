You want to join dn42, but you don't know where to start. This guide gives general guidelines about dn42 and routing in general, but it assumes that you are knowledgeable with routing.

# Requirements

- you have at least one router running 24/7. Any Linux or BSD box can be turned into a router. If your home router runs OpenWRT, you might consider using it for dn42.
- your router is able to establish network tunnels over the Internet (GRE, OpenVPN, IPSec, Tinc...). Beware, your network operator might filter this kind of traffic, e.g. in schools or universities.
- you are generally knowledgable with networking and routing (i.e. you've heard about BGP, IGP, forwarding, and you're willing to configure a BGP router such as Quagga or Bird)

# Formalities

Don't worry, it's not as tedious as registering with a RIR ;)

## Subscribe to the mailing list

This is important, as it allows to stay up-to-date on best practices, new services, security issues...

See [Contact#contact_mailing-list](contact#contact_mailing-list) to subscribe.

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

If you intend to use an ASN outside of the native dn42 ranges, please check that it doesn't clash with the [Freifunk AS-Numbers] (http://wiki.freifunk.net/AS-Nummern) or other networks (ChaosVPN, etc). For a list of ASN currently announced in dn42, see [this map](http://nixnodes.net/dn42/graph/) or [this list](http://109.24.208.244:8888/dn42/lastseen/).

If unsure, ask on the mailing list or IRC.

### Register a network prefix

To register an IPv4 network prefix, simply create an `inetnum` object.

You may choose your network prefix in one of the currently open netblocks. There is also a [graphical visualisation of the assigned ranges](http://109.24.208.244:8888/dn42-netblock-visu/registry.html).

The current guideline is to allocate a /25 by default, keeping space for a /23. You may allocate more than a /25 if you need to, but no more than a /23. In particular, if you want reverse DNS for your prefix, you will need at least a /24. (check? maybe the scripts in the repo are clever enough)


# Get some peers

In dn42, there is no real distinction between peering and transit: in most cases, everybody serves as an upstream provider to all its peers.  Note that if you have very slow connectivity to the Internet, you may want to avoid providing transit between your peers, which can be done by filtering or prepending your ASN.

If you don't know anybody who can peer with you, you can use this tool: http://peerfinder.polyno.me

It will let you find people to peer with.  You can then contact them on IRC or by email.  In case you're really at loss, you can also ask for peers on the mailing list.

## Establishing tunnels

Unless your dn42 peers are on the same network, you must establish tunnels. Choose anything you like: OpenVPN, GRE, GRE + IPSec, IPIP, Tinc, ...

There is some documentation in this wiki, like [gre-plus-ipsec](GRE-plus-IPsec).

## Running a routing daemon

You need a routing daemon to speak BGP with your peers. People usually run Quagga or Bird, but you may use anything (OpenBGPD, XORP, somebody even used an old [hardware router](bgp-on-extreme-summit1i) ).  See the relevant [FAQ entry](/FAQ#frequently-asked-questions_what-bgp-daemon-should-i-use).

You can find [configuration examples for Bird here](howto/bird).

Some [documentation of the old wiki] (http://dn42.volcanis.me/initenv/wiki/HowToPeer.html) might still be handy, but remember that everything there is terribly outdated.

## Configuration Examples

* [EdgeOS Configuration](EdgeOS-Config-Example)
* [EdgeOS GRE/IPsec Example](howto/EdgeOS-GRE-IPsec-Example)
* [BGP on Extreme Networks Summit 1i](howto/bgp-on-extreme-summit1i)
* [dn42 on OpenWRT](howto/dn42-on-OpenWRT)
* [Bird](howto/bird)
* [IPsec with public key authentication](/howto/IPsecWithPublicKeys)

# Configure DNS

See [Services DNS](Services-DNS).

# Use and provide services

See [internal](internal) for internal services.

Don't hesitate to provide interesting services, but *please*, document them on the wiki! Otherwise, nobody will use them because nobody can guess they even exist.