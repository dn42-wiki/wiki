# Network / Port scans

A network scan involves examining hosts in a network. The general aim is to find information such as open ports, software versions, and vulnerabilities. Depending on how the scan is performed, many packets are sent to the individual hosts at short intervals.

## Rules

There are no technically enforceable rules for port scanning (i.e., rules that apply only to port scanning; general restriction rules such as rate limits can of course be set), but that does not mean that you should just go ahead and do it - depending on how it is done, it can be perceived as very intrusive. Therefore, the following rules of politeness have become established over time:

1. Announce the network scan in advance on the mailing list.
2. Provide the option to opt out.

## Opt out

Unfortunately, there is currently no standard way to signal that a network should not be scanned. Several proposals have been discussed in the past - communication via BGP, via the registry, or via the mailing list.

Another option would be to signal opt-outs via the wiki:

| Maintainer | Network | Opt-out? |
| `EXAMPLE-MNT` | 172.0.0.1/24 | Yes |
