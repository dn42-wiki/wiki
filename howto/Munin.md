## Graph your route count

Number of routes by AS (IPv4)
```#!/bin/bash
if [ "$1" = "config" ];then
	echo graph_title Number of routes
	echo graph_vlabel num. routes
	echo graph_category network
	echo graph_scale no
	for AS in $(ip r|sed 's/.* dev //;s/ .*//'|sort|uniq -c|grep as|awk '{print $2}');do
		echo $AS.label $AS
	done
else
	ip r|sed 's/.* dev //;s/ .*//'|sort|uniq -c|grep as|awk '{print $2".value "$1}'
fi
```

IPv6:
```
#!/bin/bash
if [ "$1" = "config" ];then
	echo graph_title Number of routes
	echo graph_vlabel num. routes
	echo graph_category network
	echo graph_scale no
	for AS in $(ip -6 r|sed 's/.* dev //;s/ .*//'|sort|uniq -c|grep as|awk '{print $2}');do
		echo $AS.label $AS
	done
else
	ip -6 r|sed 's/.* dev //;s/ .*//'|sort|uniq -c|grep as|awk '{print $2".value "$1}'
fi
```
(hint: The difference just the -6 on the ip command)

## Graph routes and activity for every neighbour

This munin-plugin makes it very easy to graph the announced routes and activity for each neighbour over time:  
https://github.com/luben/bird-multigraph-plugin

It's also possible to get notified by Munin when a problem with the peering persists. You have to define a critical value in line 138: 
```
imported.critical 1:
```
This will send execute the command (set in munin-node.conf) to alert you, if the imported route count falls under 1.

You might also want to change line 125 from 
```
graph_title $proto->{title} routes
```
to
```
graph_title $name routes
```

Example installation: 
http://stats.tbspace.de/munin-cgi/munin-cgi-graph/tbspace.de/server.tbspace.de/dn42_crest_routes-day.png