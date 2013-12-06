## Graph your routes

```
#!/bin/bash
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