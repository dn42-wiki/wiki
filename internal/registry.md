# Registry cleanup process
This process is used to remove inactive objects based on MRT data.

A maintainer is classified as "inactive" if the following conditions have been fulfilled:
1. All of the ASNs the maintainer has been directly or indirectly associated with (in any way and by following all references, whether through mnt-by, admin-c, tech-c, etc or through an ORG) have not been observed originating a prefix in the global routing table at any point within the last three years. (Determined by analyzing the daily MRT RIB dumps provided by the DN42 Global Route collector)
2. The maintainer has not edited any of the ASNs they are associated with in the registry within the last three years. (Determined by analyzing the git commit history)

Process
Using registry_wizard v0.3.8

1. Generate a list of active asns based on MRT data
`./registry_wizard /path/to/registry mrt_activity /path/to/mrt/files --max-inactive-secs <value> --list > list.txt`
2. Generate the removal commands to remove inactive objects based on the previous list
`./registry_wizard /path/to/registry remove aut-num --list_file /path/to/list.txt --max-inactive-secs <value> --enable_subgraph_check`

For excluding ASNs from removal, they need to be added to the active ASN list that is produced by the first command.