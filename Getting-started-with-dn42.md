You want to join dn42, but you don't know where to start. This guide gives general guidelines about dn42 and routing in general, but it assumes that you are knowledgeable with routing.

# Requirements

- you have at least one router running 24/7. Any Linux or BSD box can be turned into a router. If your home router runs OpenWRT, you might consider using it for dn42.
- your router is able to establish network tunnels over the Internet (GRE, OpenVPN, IPSec, Tinc...). Beware, your network operator might filter this kind of traffic, e.g. in schools or universities.
- you are generally knowledgable with networking and routing (i.e. you've heard about BGP, IGP, forwarding, and you're willing to configure a BGP router such as Quagga or Bird)

# Formalities

Don't worry, it's not as tedious as registering with a RIR ;)

## Subscribe to the mailing list

This is important, as it allows to stay up-to-date on best practices, new services, security issues...

See [Home#Contact](Home#Contact) to subscribe.

## Fill in the registry

You must create several objects in the registry. The recommended method is to use the [web interface](https://io.nixnodes.net/?registry), but you may still work directly with the [monotone repository](Services-Whois#Monotone).

This example assumes that your name is `Luke`, part of an organisation called the `Rebel Alliance`, and that you want to register an AS and a network block for your home network `Naboo`. You may easily extrapolate to the case of a hackerspace you're a member of.

### Create a maintainer object

Create a `mntner` object named `LUKE-MNT`. It will be used to edit all the objects that are under your responsibility.

- choose a password, and don't forget it.  **Note:** even though the field is named `sha512-pw`, you must enter *your password* directly, *not* the sha512 of your password.
- use `DUMMY-DN42` as `admin-c` and `tech-c`. We will update this later.
- use `LUKE-MNT` as `mnt-by`, otherwise, you won't be able to edit your maintainer object.

### Create person objects

Create a `person` object for yourself.

- use `LUKE-DN42` as `nic-hdl`, it should end with `-DN42`.
- the `person` field is more freeform, you may use `Luke` here.
- provide an email.
- you may provide additional ways of contacting you, using one or more `contact` field. For instance `xmpp:luke@theforce.net`, `irc:luke42@hackint`, `twitter: TheGreatLuke`.
- you may which to add other fields, such as `pgp-id`, `pgp-fingerprint`, `remarks`, and so on.
- don't forget to set `mnt-by` to `LUKE-MNT`.

You must now edit the maintainer object created earlier, to properly fill in the `admin-c` and `tech-c` fields (set them to `LUKE-DN42`).

If you intend to register resources for an organisation (e.g. your hackerspace), you must also create a `person` object for your organisation:

- `nic-hdl` is `REBEL-ALLIANCE-DN42`.
- email should be a contact address for your organisation, or maybe a mailing list (but people should be able to send email without subscribing).
- `contact` could be an irc channel (e.g. `#rebelalliance@irc.theforce.net`).
- you may provide a website (`www` field).
- don't forget to set `mnt-by` to `LUKE-MNT`, since you're managing this object on behalf of your organisation.

### Guidelines for future objects

From now on, you should use:

- `admin-c: LUKE-DN42` and `tech-c: LUKE-DN42` for your own resources.
- `admin-c: REBEL-ALLIANCE-DN42` and `tech-c: LUKE-DN42` for the resources of your organisation.

This applies to AS numbers, network prefixes, routes, DNS records...

### Register an AS number

To register an AS number, simply create an `autnum` object.

Your AS number can be chosen arbitrarily in the dn42 ASN space, look at the `as-block` objects. The historic ASN space is around 64600-64855 and 76100-76200. Starting from June 2014, **you should allocate your AS number in the new 4242420000-4242423999 range**.

For a list of currently assigned AS numbers, see http://ix.ucis.nl/dn42/as.php. This list is automatically built from the registry.

If you intend to use an ASN outside of the native dn42 ranges, please check that it doesn't clash with the [Freifunk AS-Numbers] (http://wiki.freifunk.net/AS-Nummern) or other networks (ChaosVPN, etc). For a list of ASN currently announced in dn42, see [this map](http://nixnodes.net/dn42/graph/) or [this list](http://109.24.208.244:8888/dn42/lastseen/).

If unsure, ask on the mailing list or IRC.

### Register a network prefix

To register an IPv4 network prefix, simply create an `inetnum` object.

You may choose your network prefix in one of the currently open netblocks. There is also a [graphical visualisation of the assigned ranges](http://109.24.208.244:8888/dn42-netblock-visu/registry.html).

The current guideline is to allocate a /25 by default, keeping space for a /23. You may allocate more than a /25 if you need to, but no more than a /23. In particular, if you want reverse DNS for your prefix, you will need at least a /24. (check? maybe the scripts in the repo are clever enough)


# Get some peers

In dn42, there is no distinction between peering and transit: by default, everybody serves as an upstream provider to all its peers.

If you don't know anybody who can peer with you, ask on IRC or the mailing list.

## Establishing tunnels

Unless your dn42 peers are on the same network, you must establish tunnels. Choose anything you like: OpenVPN, GRE, GRE + IPSec, IPIP, Tinc, ...

There is some documentation in this wiki, like [gre-plus-ipsec](howto/gre-plus-ipsec).

## Running a routing daemon

You need a routing daemon to speak BGP with your peers. People usually run Quagga or Bird, but you may use anything (OpenBGPD, XORP, somebody even used an old [hardware router](howto/bgp-on-extreme-summit5i) ).  See the relevant [FAQ entry](Frequently-Asked-Questions#What-BGP-daemon-should-I-use?).

You can find [configuration examples for Bird here](howto/bird).

Some [documentation of the old wiki] (http://dn42.volcanis.me/initenv/wiki/HowToPeer.html) might still be handy, but remember that everything there is terribly outdated.

## Configuration Examples

* [EdgeOS Configuration](EdgeOS-Config-Example)
* [EdgeOS GRE/IPsec Example](howto/EdgeOS-GRE-IPsec-Example)
* [BGP on Extreme Networks Summit 5i](howto/bgp-on-extreme-summit5i)
* [dn42 on OpenWRT](howto/dn42-on-OpenWRT)
* [Bird](howto/bird)
* [IPsec with public key authentication](/howto/IPsecWithPublicKeys)

# Configure DNS

See [Services DNS](Services-DNS).

# Use and provide services

See [internal](internal) for internal services.

Don't hesitate to provide interesting services, but *please*, document them on the wiki! Otherwise, nobody will use them because nobody can guess they even exist.