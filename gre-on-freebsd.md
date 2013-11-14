# GRE on FreeBSD

This page describes how to configure GRE tunnels on FreeBSD.

## Requirements

* Root access to a FreeBSD system.
* Loaded if_gre.ko or device gre
* A static IPv4 address on both ends if you would like to preserve your sanity.

## Create a temporary gre tunnel

```bash
ifconfig gre$INDEX create
ifconfig gre$INDEX tunnel $TUNNEL_SRC $TUNNEL_DST
ifconfig gre$INDEX inet $LOCAL $REMOTE netmask 0xffffffff
ifconfig gre$INDEX descr $DESCR
```

## Create a persistent gre tunnel

Add this to your `rc.conf`.

```
cloned_interfaces="$cloned_interfaces gre0"
ifconfig_gre0="10.0.0.1 10.0.0.2 netmask 0xffffffff tunnel 1.2.3.4 5.6.7.8 descr foo"
```
