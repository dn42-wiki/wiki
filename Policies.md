# Policies

## Bridging from the Internet

dn42 is a separate network from Internet and users are actively discouraged from creating publicly accessible services that allow direct access to the dn42 IP space from the Internet; this includes public NAT and VPN services.

dn42 is primarily for learning about network technologies. Providing zero effort access to the network runs counter to that core purpose and prevents users gaining experience of learning how to create and configure their own network.

Preventing direct access also helps keep AI scrapers and other abuses that are common on the Internet, away from the network. 

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
