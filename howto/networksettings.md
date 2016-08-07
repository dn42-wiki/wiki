The first rule of dn42: Always disable `rp_filter`. 

The second rule of dn42: Always disable `rp_filter`. 

The third rule of dn42: Allow ip forwarding!

No serious, in case some packets are dropped check first if your settings are correct.

`rp_filter` also known as reverse path filtering is a security measure. 
When the route to return a packet uses a different interface than it arrived from, the packet is dropped. 
Some attackers will set a wrong return address on their packets. This security measure was created to address when this happens. Core internet routing can however be asymmetric. This means that packets can take different routes on the return path.
That is why `rp_filter` needs to be disabled.

**Note** using sysctl is not persistent. Depending on your linux distribution put it into `/etc/sysctl.conf` or `/etc/sysctl.d`

```
sysctl -w net.ipv4.conf.all.rp_filter=0 net.ipv4.conf.default.rp_filter=0
```

Check that its really disabled:
```
sysctl -a | grep rp_filter
```

Also the following options must be set.
```
$ sysctl -w net.ipv4.conf.all.forwarding=1 net.ipv6.conf.all.forwarding=1
```

Check that ALL your vpn interfaces allow ip forwarding for ipv6/ipv4.
```
$ sysctl -a | grep forwarding
```

Happy Routing!