# Registry cleanup process
This process is used to remove inactive objects based on MRT data and the git commit history.
The process is to be executed on a regular basis.

A maintainer is classified as "inactive" if the following conditions have been fulfilled:
1. All of the ASNs the maintainer has been directly or indirectly associated with (in any way and by following all references, whether through mnt-by, admin-c, tech-c, etc or through an ORG) have not been observed originating a prefix in the global routing table at any point within the last three years. (Determined by analyzing the daily MRT RIB dumps provided by the DN42 Global Route collector)
2. The maintainer has not edited any of the ASNs they are associated with in the registry within the last three years. (Determined by analyzing the git commit history)

## Process

Using **registry_wizard v0.3.10**:

1. Generate a list of active ASNs based on MRT data
`./registry_wizard /path/to/registry mrt_activity parse /path/to/mrt/files --cutoff-time <value> --list > active_list.txt`
2. Based on the list of active ASNs and through consultation of the registry, generate a list of inactive ASNs.
`./registry_wizard /path/to/registry mrt_activity active_asn_to_inactive --list_file /path/to/active_list.txt --cutoff-time <value> > inactive_list.txt`
3. Generate the removal commands to remove inactive objects based on the previous list
`./registry_wizard /path/to/registry remove aut-num --list_file /path/to/list.txt --enable_subgraph_check`

ASNs can be excluded from removal by removing them from the list produced in the second step.
Manual review of a few resources affiliated with "DN42-MNT" will be required as they cannot be removed in an automated way (For example resources associated with an inactive maintainer that used to host the DN42 anycast DNS will be affiliated with DN42-MNT and will require manual removal).