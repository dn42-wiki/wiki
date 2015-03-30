#exabgp watchdog script

```

#!/bin/bash

URL=( "http://172.23.0.80" "https://172.23.0.80" )
VALIDATE_KEYWORD='gollum'
ROUTE='172.23.0.80/28'
NEXTHOP='172.22.177.72'

INTERVAL=60

###########################

RUN_STATE=0

check_urls() {
	for url in "${URL[@]}"; do
		curl --insecure -L -o - ${url} | egrep -q "${VALIDATE_KEYWORD}" || {
			return 1
		}
	done
	return 0
}

while [ 1 ]; do
	if [ ${RUN_STATE} -eq 0 ]; then
		check_urls && {
			RUN_STATE=1
			echo "announce route ${ROUTE} next-hop ${NEXTHOP}"
		}
	else
		check_urls || {
			RUN_STATE=0
			echo "withdraw route ${ROUTE} next-hop ${NEXTHOP}"
		}
	fi

	sleep ${INTERVAL}

done

exit 0


```


exabgp runs directly on the web server and peers with border-routers, configuration: http://paste42.de/7976/


bird is set up like this: http://paste42.de/7977/