# Statistics


## IRC

Channel statistics for #dn42@hackint are available at: https://dev.0l.dn42/stats/.

## Scripts

### Number of prefixes for collectd

#### collectd.conf

```
LoadPlugin exec
<Plugin exec>
   Exec nobody "/etc/collectd/bgp_prefixes-quagga.sh"
</Plugin>
```

collectd refuses to exec scripts as root. On Debian vtysh is compiled with PAM support: adding nobody to the quaggavty group suffices.

#### bgp_prefixes-quagga.sh

```sh
#!/bin/bash

INTERVAL=10
HOSTNAME=dn42.hq.c3d2.de

while true; do
n4=$(vtysh -d bgpd -c "show ip bgp"|grep Total|sed -e 's/Total number of prefixes //')
n6=$(vtysh -d bgpd -c "show ipv6 bgp"|grep Total|sed -e 's/Total number of prefixes //')

echo "PUTVAL $HOSTNAME/quagga-bgpd/routes-IPv4 interval=$INTERVAL N:$n4"
echo "PUTVAL $HOSTNAME/quagga-bgpd/routes-IPv6 interval=$INTERVAL N:$n6"

sleep $INTERVAL
done
```

#### Number of prefixes per neighbour for bird

```sh
#!/bin/sh
#
# Collectd script for collecting the number of routes going through each
# BGP neighour.  Works for bird.
#
# See https://dn42.net/Services-Statistics

INTERVAL=60
HOSTNAME=mydn42router
[ -n "$COLLECTD_HOSTNAME" ] && HOSTNAME="$COLLECTD_HOSTNAME"

while true
do
    birdc 'show protocols "*"' | grep ' BGP' | cut -d ' ' -f 1 | while read neighbour
    do
        nbroutes=$(birdc "show route protocol $neighbour primary count" | grep -v 'BIRD' | cut -d ' ' -f 1)
        echo "PUTVAL $HOSTNAME/bird-bgpd/routes-$neighbour interval=$INTERVAL N:$nbroutes"
    done
    # FIXME: we probably count non-BGP routes here
    totalroutes=$(birdc "show route primary count" | grep -v 'BIRD' | cut -d ' ' -f 1)
    echo "PUTVAL $HOSTNAME/bird-bgpd/routes-all interval=$INTERVAL N:$totalroutes"
    sleep $INTERVAL
done
```

### munin plugin
* add the following to /etc/munin/plugin-conf.d/munin-node

```
[quagga_bgp]
user root
```

* place the script as quagga_bgp in /etc/munin/plugins

```sh
#!/bin/sh
#
#
# Munin Plugin to show quagga bgp4 routes

# Standard Config Section Begin ##
  if [ "$1" = "autoconf" ]; then
        echo yes
        exit 0
  fi

  if [ "$1" = "config" ]; then

       echo 'graph_title Quagga BGP4 Routes'
       echo 'graph_args --base 1000 -l 0'
       echo 'graph_scale yes'
       echo 'graph_vlabel Received routes via BGP4'
       echo 'graph_category Network'
       echo 'bgproutes.label Routes'
       echo 'graph_info Route information provided by quagga daemon via vtysh'
       exit 0
  fi
# Standard Config Section End ####

# Measure Section Begin ##########
  data=($(vtysh -c "show ip bgp"|grep Total|cut -d" " -f5))

  if [ "$data" = "" ]; then
     echo bgproutes.value    0
  else
     echo bgproutes.value    $data
  fi
# Measure Section ##########
```
* restart munin-node
