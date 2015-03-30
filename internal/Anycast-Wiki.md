The idea is to setup mirrors of this wiki across dn42, using [anycast](https://en.wikipedia.org/wiki/Anycast) to provide redundancy and load-balancing.
The local webserver is monitored with a simple shell script (below) working in conjuction with [ExaBGP](https://github.com/Exa-Networks/exabgp), announcing/withdrawing the assigned route if the service is up/down.  

### Checklist:

 * Install [gollum](https://github.com/gollum/gollum)
 * Clone the dn42 wiki repo:

    `git clone ssh://git@xuu.me/dn42/wiki`

 * Request a [certificate](/services/Certificate-Authority)


#####gollum-watchdog.sh:

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

