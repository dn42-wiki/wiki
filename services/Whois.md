# Whois registry
**aka** _The registry_ contains:

  * AS numbers assignations
  * Subnet assignations
  * DNS root zone for `dn42.`

The registry is a git repository, hosted here: [https://git.dn42.dev/dn42/registry.git](https://git.dn42.dev/dn42/registry.git), Changes to the registry are managed by submitting pull requests to the repository, these are then reviewed by the registry maintainers before acceptance. 

# Names and numbers

dn42 uses some names and numbers, which are declared in the registry.  Whenever possible, we try to stick to names and numbers that do not conflict with the ICANN-net or other networks similar to dn42, for instance by using private numbers space.

## Address space

dn42 uses **172.20.0.0/14** for IPv4.

For IPv6, we use ULA (that is, **fd00::/8**).

See also the howto page covering the [DN42 address space](/howto/Address-Space).

## AS numbers

Since June 2014, dn42 is using the **4242420000-4242429999** ASN range for allocations. This range is further subdivided:
* **4242420000-4242423999** for end-users allocations
* **4242424000-4242426999** reserved for future use
* **4242427000-4242429999** for sub-allocations

If you are running a project similar to dn42, please use another range of ASN. The "sub-allocations" range is meant for dn42 users willing to have administrative control over a small, consecutive range of ASN (e.g. to use them directly or to distribute them).

Note that currently, most AS are using one of the legacy ASN range (and will probably continue to do so, as renumbering is painful). See the [FAQ](/FAQ#frequently-asked-questions_why-are-you-using-asn-in-the-76100-76199-range) for a discussion on AS ranges.

## DNS zones

dn42 uses the `dn42.` TLD, which is not present in the root DNS zone of the ICANN-net.  For details, see [DNS](/DNS).

Note that other TLDs should also be usable from dn42, most notably from Freifunk and ChaosVPN. A tentative list is available at [External DNS](/services/dns/External-DNS).

# Drone CI/CD

The gitea instance hosting the registry has an associated [Drone CI/CD](https://drone.io/) service:

- [https://drone.git.dn42/](https://drone.git.dn42/)

Users are free to add drone pipelines to their own repositories. Repositories can be enabled using the Drone server [user interface](https://drone.git.dn42/).

# Telegram Bot
A telegram whois bot owned by [@Oxygen233](https://t.me/oxygen233) is hosted on [@DN42WhoisBot](https://t.me/DN42WhoisBot).

Privacy mode is enabled, please call the bot with @DN42WhoisBot when necessary.

# Web interface and REST API

[https://explorer.burble.dn42/](https://explorer.burble.dn42/) ([https://explorer.burble.com/](https://explorer.burble.com/) via clearnet) provides a web interface and REST API for querying the DN42 registry.

The service is provided by [dn42regsrv](https://git.dn42.us/burble/dn42regsrv) which can also be run locally.

## Authentication

See the page on [Registry Authentication](howto/Registry-Authentication)

# DNS interface

There is also a DNS-based interface to query AS information from the registry. The DNS zone is `asn.dn42`. 
Mirrors are hosted at `asn.grmml.dn42` and `asn.lorkep.dn42`.

Example:

    $ dig +short AS4242420000.asn.dn42 TXT
    "4242420000 | DN42 | dn42 |  | PYROPETER-AS PyroPeters AS"

The Python code for generating the zone from the registry is available on the monotone repository.

The idea comes from the guys at cymru.com, who provide this service for the Internet (e.g. `AS1.asn.cymru.com`), see https://www.team-cymru.org/Services/ip-to-asn.html#dns

# Software

 * [lglass](/internal/lglass) is a python implementation for working with the registry. It features a whois server, tools to manipulate the data (DNS zone generation, etc).
 * [whois42d](https://github.com/dn42/whois42d) written in golang, lightweight/fast, whois server with support for all registry objects, type filtering and systemd socket activation.

# Whois daemons

We have anycast IPv4 and IPv6, both reachable under whois.dn42. IPs are 172.22.0.43 respective fd42:d42:d42:43::1. Please consider joining these anycast-adresses when you setup your server. Updates every 1 hour would be nice for a start.

| **person**  | **dns**                   | **ip**          |
|-------------|---------------------------|-----------------|
| org-cccda   | whois.cda.dn42            | 172.23.96.1 / fd42:23:cda::1 |
| StrExp / NIA| whois.nia.dn42            | 172.20.158.153 / fd00:1926:817:43::1 |
| Lan Tian    | whois.lantian.dn42        | 172.22.76.108 / fdbc:f9dc:67ad:2547::43 |
| burble      | whois.burble.dn42         | 172.20.129.8 / fd42:4242:2601:ac43::1 |
| taavi       | whois.svc.as4242423270.dn42 | 172.22.130.143 / fd96:70f6:b174:<span>ac</span>::43 |

### Down?

| **person**  | **dns**                   | **ip**          |
|------------|---------------------------|-----------------|
| welterde   | thinkbase.srv.welterde.de | 46.4.248.201    |
| prauscher  | sheldon.prauscher.dn42    | 172.22.120.1    |
| w0h        | whois.w0h.dn42               | 172.22.232.6 / fd2d:a6da:8d1a:1408::6 |
| Mic92      | whois.evenet.dn42 ([whois42d](https://git.dn42.us/dn42/whois42d)) | 172.23.75.1 / fd42:4992:6a6d::6 |
| Fritz      | whois.flhb.de                | 172.22.70.69 / 2001:67c:708:102:5054:ff:fe57:9573 / fdd6:aff6:5f6f:102:5054:ff:fe57:9573 |
| weiti       | whois.weiti.dn42          | 172.20.175.253 / fdf7:17d5:de49::43 |

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
^172\.2[0-3]\.[0-9]{1,3}\.[0-9]{1,3}(/(1[56789]|2[0-9]|3[012]))?$ whois.dn42

# dn42 ula ipv6 address space
^fd**:****:****:****:****:****:****:**** whois.dn42

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
## Whois restful API
Note: this service is in beta testing, use at your own risk.
https://whois.rest.dn42/
