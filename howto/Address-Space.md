DN42 uses network addresses in the [rfc1918](https://tools.ietf.org/html/rfc1918) and [ULA](https://tools.ietf.org/html/rfc4193) ranges. These are described in detail in the sections below. 

The [DN42 registry](https://git.dn42.us/dn42/registry) is the authoritative source of information on address space assignment. Within the registry, the DN42 address space is divided in to blocks based on _policies_ that define how the addresses may be used. Policies are defined in `inetnum` and `inet6num` objects and can be:

 - **open** - users may request prefixes in this range, subject to any constraints that are described in the `remark` attributes
 - **closed** - these ranges cannot be assigned
 - **reserved** - these ranges are reserved for future use
 - **ask** - these ranges are for specific uses, please ask on the mailing list before requesting assignments

The [filter.txt](https://git.dn42.us/dn42/registry/src/master/data/filter.txt) and [filter6.txt](https://git.dn42.us/dn42/registry/src/master/data/filter6.txt) files within the registry detail the network wide constraints on what address ranges are in use together with the global limits on what can be announced. 

`inetnum` and `inet6num` objects within the registry are used to describe the allocation of address space to users. `route` and `route6` objects in the registry are used to validate routing announcements through [ROA](https://wiki.dn42/howto/Bird#route-origin-authorization). 

In addition to the native DN42 address ranges, the registry also contains allocations for the address space used by affiliate networks. These are updated by a regular [sync script](https://git.dn42.us/dn42/registry-sync). 

Globally routable prefixes are not supported in DN42; they are denied via the registry filter{6,}.txt files and many networks will filter both announcements and traffic for prefixes that are outside of the allowable ranges.

# IPv6 Address Space

DN42 uses the fd00::/8 ULA range for IPv6 addresses; the whole fd00::/8 block has an open policy and users are free to request any prefix in this range, that is not already allocated. 

**The DN42 registry is not authoritative for the fd00::/8 range**

DN42 is interconnected with other networks, like icvpn, which also use the same ULA range and many users will also use this range for their own networks. A registration in the dn42 registry cannot prevent IPv6 conflicts, so a fully random prefix (see [RFC4193](https://tools.ietf.org/html/rfc4193)) is strongly recommended. If an address conflict is found, then needing to renumber your network is no fun.

# IPv4 Address Space

DN42 uses the 172.20.0.0/14 range for IPv4 addresses. As with the public internet, IPv4 space is limited and users are encouraged to conserve space where possible. 

Unlike the IPv6 address space, the DN42 IPv4 space is not fully open for assignment to users; some ranges are intended for specific uses and other ranges are reserved. See the policy section, below. Users should always check the policy in the registry before requesting a prefix to be assigned. 

There are other IPv4 ranges in use within DN42 related to the affiliate networks, see the [filter.txt](https://git.dn42.us/dn42/registry/src/master/data/filter.txt) file in the registry. 

## IPv4 Policies

The diagram below shows the allocation policies for the DN42 address space. 

... image ...

Specific policy restrictions:

| Prefix | Usage |
|--------|-------|
| 172.20.0.0/24<br/>172.21.0.0/24<br/>172.22.0.0/24<br/>172.23.0.0/24 | Reserved for anycast addresses |
| 172.20.240.0/20<br/>172.22.240.0/20 | Reserved for transfer networks |
| 172.20.64.0/18 | Reserved for allocations larger than /23, up to /21 |
| 172.22.0.0/18 | Reserved for allocations of /24 or larger, up to /21 |
| 172.23.16.0/21 | Closed to new allocations |


