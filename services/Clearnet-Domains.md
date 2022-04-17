# DN42 Clearnet Domains

To provide services over the public internet some community members have contributed clearnet domain names to be used for DN42.

|Domain|
|:--|
|dn42.dev|
|dn42.no|
|dn42.tk|


DNS records for these domains are managed by a gitea repository:

- [https://git.dn42.dev/dns/clearnet](https://git.dn42.dev/dns/clearnet)

The repository uses the registry drone CI service and [DNSControl](https://stackexchange.github.io/dnscontrol/) to automatically build and push changes. This provides a degree of independence for the domains and allows them to be updated by any of the registry maintainers.

## Requesting Changes

If you have a public DN42 service or a DN42 related domain that you'd like to add, simply submit a PR to the [repository](https://git.dn42.dev/dns/clearnet).

Domains and services should meet the following criteria:

- Services and DNS should be resilient and highly available, where possible
- The owner must ensure their contact details are current and respond promptly to any failures
- If a domain or service is unavailable for a length of time, it may be removed