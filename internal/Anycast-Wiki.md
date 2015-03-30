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
gollum --css <path>/custom.css --gollum-path <path>/public_html/ --host 127.0.0.1  --port 4568
    ```
   Plain (read-only):
    ```
gollum --css <path>/custom.css --gollum-path <path>/public_html/ --host 127.0.0.1  --port 4567 --no-edit
    ```

###Install/configure nginx:

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

###Setup [ExaBGP](https://github.com/Exa-Networks/exabgp)

```

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

