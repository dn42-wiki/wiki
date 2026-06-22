# Policies

## Resource Allocations

dn42 global resources are limited so, by necessity, all requests for resources above the typical allocations must be treated as exceptions. If users genuinely need additional resources they can be provided, but the registry maintainers can only do this by challenging every request to ensure it is justified. It's not fair to throw away resources now and then require much stricter limits in the future because we ran out of space.

Users applying for additional resources should consider their requests against the following criteria:

 - Does your allocation provide **additional value** to the rest of the dn42 community?

Allocations for personal use are harder to justify, why should you get global, community resources if that doesn't provide any value back to the community?

 - Why is your use case **exceptional**?

Requests for additional resources cannot be the norm, so they must be exceptional.\
What makes your use special over everybody else in the network?\
Why should you get additional resources when other people will not be able to have them?

 - Is your request based on **demonstrable need**?

Additional resources should only be requested where there is clear evidence that existing resources are being fully utilised and are insufficient for the current requirement. Resources will not be granted based on anticipated future demand or the possibility that they may be needed later.

If you are developing a new service that requires additional resources, you should first deploy and operate the service so that the requirement can be demonstrated and justified.

### New Users

Typical and maximum allocations for new users

| | ASN | IPv6 Allocation | IPv4 Allocation |
|:--|:--|:--|:--|
| Typical | 1 | /48 | /27 |
| Maximum | 1 | /48 | /26 |

The typical allocation provides enough resources for most new users; it is suitable for doing all the fun things in dn42 and growing your network.

For IPv4, new users should be aware that allocations smaller than a /27 can become restrictive. Even if your network is currently small, a /27 usually provides a better starting point and gives you room to expand later without requiring a second resource request or fragmenting the IP space.

For IPv6, there is generally no advantage to request anything other than a /48. Even if your network doesn't support IPv6 yet, some parts of dn42 are IPv6-only, and IPv6 is the preferred protocol for many services. Allocations smaller than a /48 can be very restrictive, while a /48 provides enough address space for even very large networks.

When registering for the first time, new users will not be allocated more than the maximum amounts shown above.

These limits are based on the principle of demonstrable need. Additional resources are provided when there is clear evidence that existing resources are being fully used and are no longer sufficient.

For this reason, please do not register and immediately request more resources. Requests for additional allocations will normally only be considered after you can demonstrate that your current allocations are actively in use and no longer meet your requirements.

### ASN Allocations

ASNs are administrative, not geographical boundaries; additional ASNs are not required for additional nodes or to provide geographic services.

Additional ASNs 'for testing' are also unlikely to be provided.

Alternative approaches include:

- Setting up local lab environment using private ASNs in the 42xxxxxxxx range, outside the dn42 424242xxxx range. There's no problem peering locally with your dn42 router as an external peer to testing policies and what it looks like, or even having your own local mini dn42 with multiple peers and transits.

- You can already view how the rest of the network sees your ASN via looking glasses; it's also instructive to see how other user's policies impact your announcements. Learning what to look for or diagnose problems using a looking glass is a valuable skill in its own right.

- Talk to your peers! dn42 is a community and most users will be happy to help.

### IPv4 Allocations

A typical IPv4 allocation is /27. This provides you with 32 x /32 IPv4 addresses which is often more than enough for most networks. You should typically only require 1 IPv4 address for each peering node or service on dn42.

The smallest routable IPv4 prefix size in dn42 is /29.

Total IPv4 allocations greater than /26 will be challenged and you will be expected to provide robust justification.

Allocations >= /25 are rare and >= /24 require mailing list consensus; these will require extraordinary justification and are almost never granted apart from the most exceptional (and usually temporary) circumstances.

If you have a requirement for a large number of IP addresses you should consider using IPv6 first.

You should also consider using NAT for anything not directly providing dn42 services (for example, where you want to access dn42 from a homelab or laptop, but are not providing services to the rest of the dn42 community).

### IPv6 Allocations

The smallest routable IPv6 prefix size in dn42 is /64.

There are very few reasons for not allocating the standard /48 address block; it provides a huge address space sufficient to run a global network whilst smaller block sizes can often be limiting.

A typical IPv6 address plan will use /56 subnets per site, with the typical /48 providing for 256 x /56 sites and 256 x /64 networks per site.

## Bridging from the Internet

dn42 is a separate network from the public Internet. While dual-homed services can be beneficial (for example, looking glasses), users should not provide services that grant direct access to the dn42 address space from the Internet. This includes, public NAT gateways, VPN services, proxies, or other mechanisms that allow non-dn42 users to reach dn42 resources.

dn42 is primarily for learning about network technologies. Providing zero effort access to the network runs counter to that core purpose and prevents users gaining experience of learning how to create and configure their own network.

Preventing direct access from the public Internet also helps protect the network from abuse, including AI scrapers, large-scale crawlers, and other unwanted traffic that is commonplace on the Internet. This reduces unnecessary load on network participants and contributes to a safer and more stable environment.

## Network / Port scans

A network scan involves examining hosts in a network. The general aim is to find information such as open ports, software versions, and vulnerabilities. Depending on how the scan is performed, many packets are sent to the individual hosts at short intervals.

### Rules

There are no technically enforceable rules for port scanning (i.e., rules that apply only to port scanning; general restriction rules such as rate limits can of course be set), but that does not mean that you should just go ahead and do it - depending on how it is done, it can be perceived as very intrusive. Therefore, the following rules of politeness have become established over time:

1. Announce the network scan in advance on the mailing list.
2. Provide the option to opt out.

### Opt out

Unfortunately, there is currently no standard way to signal that a network should not be scanned. Several proposals have been discussed in the past - communication via BGP, via the registry, or via the mailing list.

Another option would be to signal opt-outs via the wiki:

| Maintainer | Network | Opt-out? |
| --- | --- | --- |
| `EXAMPLE-MNT` | 172.0.0.1/24 | Yes |


## Registry cleanup

A maintainer is classified as "inactive" if the following conditions have been fulfilled:
1. All of the ASNs the maintainer has been directly or indirectly associated with (in any way and by following all references, whether through mnt-by, admin-c, tech-c, etc. or through an ORG) have not been observed originating any prefix in the global routing table at any point within the last three years. (Determined by analyzing the daily MRT RIB dumps provided by the DN42 Global Route collector.)
2. The maintainer has not edited any of the ASN objects they are associated with in the registry within the last three years. (Determined by analyzing the git commit history.)

Maintainers that are not affiliated with an ASN (whether directly or indirectly or through other maintainers) are also considered inactive regardless of whether they fulfill the above conditions.
A maintainer may also ask to opt out from a cleanup operation.

### Process
The process is to be executed on a regular basis (yearly).

Using **registry_wizard (written for v0.4.12)**:

1. Download the MRT files from the Global Route Collector (GRC):
`wget -r -np -nH --cut-dirs=1 -A "*.mrt.bz2" --reject "*:*" http://collector.dn42/`
2. Generate a list of active ASNs based on MRT data:
`./registry_wizard /path/to/registry mrt_activity parse /path/to/mrt/files --cutoff-time <value> --list > active_list.txt`
3. Based on the list of active ASNs and through referencing the registry git commit log, generate a list of inactive ASNs:
`./registry_wizard /path/to/registry mrt_activity active_asn_to_inactive --list_file /path/to/active_list.txt --cutoff-time <value> > inactive_list.txt`
4. Generate the removal commands to remove inactive objects based on the previous list:
`./registry_wizard /path/to/registry remove aut-num --list_file /path/to/inactive_list.txt --enable_subgraph_check`

ASNs can be excluded from removal by removing them from the list produced in step 3.

Manual review of a few resources (primarily those affiliated with "DN42-MNT") will be required as they cannot be removed in an automated way (for example, resources associated with an inactive maintainer that used to host the DN42 anycast DNS will be affiliated with DN42-MNT and will require manual removal).
To identify the exact conflicts leading to the manual review requirement the following command can be used:
`./registry_wizard /path/to/registry graph path mntner YAMAKAYA-MNT mntner DN42-MNT` (To list conflicts between YAMAKAYA-MNT and DN42-MNT)
