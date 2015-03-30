## Anycasted wiki mirrors

### Intro:

The idea is to setup mirrors of this wiki across dn42, using [anycast](https://en.wikipedia.org/wiki/Anycast) to provide redundancy and load-balancing.
The local webserver is monitored with a simple shell script (below) working in conjuction with [ExaBGP](https://github.com/Exa-Networks/exabgp), announcing/withdrawing the assigned route if the service is up/down.  

### Network:

 * Install wiki anycast address `172.23.0.80/32` on the system
 * Setup tunnel(s) to the dn42 network (routing daemon not required)

### Set up gollum:

 * Install [gollum](https://github.com/gollum/gollum)
 * Clone the dn42 wiki repo:

    `git clone ssh://git@xuu.me/dn42/wiki <path>`

 * Generate a [CSR](/services/Certificate-Authority) and send to `xuu@sour.is`. Wait for a reply containing internal.dn42/wiki.dn42 certificates.
 * Start two gollum instances, read-only and editing on `127.0.0.1`:
 
   SSL (read/write):
    ```
gollum --css <path>/custom.css --gollum-path <path> --host 127.0.0.1  --port 4568
    ```
   Plain (read-only):
    ```
gollum --css <path>/custom.css --gollum-path <path> --host 127.0.0.1  --port 4567 --no-edit
    ```

### Setup nginx proxy:

##### /etc/nginx/sites-enabled/wiki.dn42:

```
ssl_protocols  TLSv1.2 TLSv1.1 TLSv1;
ssl_session_cache shared:SSL:2m;
 
ssl_ciphers   ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA;

ssl_prefer_server_ciphers   on;

upstream wiki { server 127.0.0.1:4567; }

server { 
        server_name internal.dn42 wiki.dn42;

        listen 172.23.0.80:80 default;

        add_header strict-transport-security  "max-age=0; includeSubDomains";

        location / {
                location =/robots.txt { root <path>/wiki.dn42/; }
                location =/custom.css { root <path>/wiki.dn42/; }
                proxy_pass http://wiki;
        }
}

upstream wikirw { server 127.0.0.1:4568; }


server { 
        server_name internal.dn42 wiki.dn42;

        listen 172.23.0.80:443 ssl default;

	ssl on;
        ssl_certificate      <path>/ssl.crt;  
        ssl_certificate_key  <path>/ssl.key;

        add_header strict-transport-security  "max-age=0; includeSubDomains";
        add_header Public-Key-Pins            'pin-sha256="mJ1xUCzfru8Ckq2+M6VkNKGOGgSETImRAHBF24mjalw="; pin-sha256="/gOyi7syRMR+d2jZoB/MzcSD++8ciZkSl/hZAQgzWws="; max-age=0; includeSubDomains';

        location / {
                location =/robots.txt { root <path>/wiki.dn42/; }
                location =/custom.css { root <path>/wiki.dn42/; }
                proxy_pass http://wikirw;
        }
}

```

### Setup [ExaBGP](https://github.com/Exa-Networks/exabgp):

##### exabgp.conf:

```

group gollum-watchdog {
  neighbor <peer1> {
    router-id x.x.x.x;
    local-address <source-address>;
    local-as <ownas>;
    peer-as <peeras>;
  }

  ## (example) peer with one of our iBGP speakers:
  neighbor <172.22.0.1> {
    router-id 172.23.0.80;
    local-address <172.22.0.2>;
    local-as 123456;
    peer-as 123456;
  }

  ## ...

  process watch-gollum {
     run <path>/gollum-watchdog.sh;
  }
}

```

Watchdog runs in an infinite loop, sending the appropriate commands to stdout. [ExaBGP](https://github.com/Exa-Networks/exabgp) attaches to the process' stdout and listens for instructions. Watchdog sends either a route announce or widthdraw.

Run the script in a shell first to validate it's working.

##### gollum-watchdog.sh:

```
#!/bin/bash

CURL=curl

## url's to check (all listed must be alive to send announce)
URL=( "http://172.23.0.80" "https://172.23.0.80" )

## the anycast route (/28 due to prefix size limits)
ROUTE='172.23.0.80/28'

## the next-hop we'll be advertising to neighbor(s)
NEXTHOP='<source-address>' 

## regex match this keyword against HTTP response from curl
VALIDATE_KEYWORD='gollum'

INTERVAL=60

###########################

RUN_STATE=0

check_urls() {
	for url in "${URL[@]}"; do

                ## workaround curl errno 23 when piping
                http_response=`${CURL} --insecure -L -o - "${url}"`

		echo "${http_response}" | egrep -q "${VALIDATE_KEYWORD}" || {
			return 1
		}

                ## add more checks

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

  Normally SIGUSR1 to the exabgp process triggers a configuration update, but at occasion the process might need to be restarted - since its gracefull shutdown can be glitchy , this might be a bit difficult. Sending SIGKILL to the child(ren) and immediately after, the parent, does the job (quick-and-dirty). 

##### /etc/exabgp/run.sh
`USAGE: /etc/exabgp/run.sh [start|stop|restart]`

```

#!/bin/bash

PID_FILE=/var/run/exaBGP/exabgp_PID

######################################

EXABGP=<path>/sbin/exabgp
EXA_LOG=/var/log/exabgp.log
CONF=/etc/exabgp/exabgp.conf


start() {
	[ -f ${PID_FILE} ] && {
		echo "WARNING: `cat ${PID_FILE}`: exabgp already running"; return 1
	}
	${EXABGP} ${CONF} &> ${EXA_LOG} &
	cpid=$!
	[ ${cpid} -eq 0 ] && {
		echo "ERROR: could not start process"; return 1
		
	}
        echo $! > ${PID_FILE}
}

stop(){
	[ -f ${PID_FILE} ] || return 1 
	pkill -9 -P $(cat ${PID_FILE})
        kill -9  $(cat ${PID_FILE})
	rm -f ${PID_FILE}
}

case ${1} in
    start )
	start
    ;;
    stop )
	stop
   ;;
    restart )
	stop
	sleep 1
	start
   ;;
esac

exit 0

```

