The idea is to deploy mirrors across dn42 using [anycast](https://en.wikipedia.org/wiki/Anycast) addressing (BGP), thus providing redundancy, load-balancing and improved access times to the wiki. Sites are powered by [gollum](https://github.com/gollum/gollum) which has no native SSL support, so Nginx acts as a reverse proxy and handles the encryption.
The local webserver is monitored with a simple [[shell script|Distributed-Wiki#exabgp_watchdog-script]] working [[in conjunction|Distributed-Wiki#exabgp]] with [ExaBGP](https://github.com/Exa-Networks/exabgp), announcing/withdrawing the assigned route if the service is up/down.

## Prerequisites

  - Stable 24/7 connection to dn42.
  - Stable host machine, present in dn42.
  - Solid networking/sysadmin skills (to keep the system as a whole stable).

    In contrast with the general spirit in dn42, this service should NOT be deployed by unskilled members for the purpose of learning and exploring - since it is the primary source of information related to the project, it should not be used as a playground. 

  - Software:
    + [Git](https://en.wikipedia.org/wiki/Git_(software))
    + [gollum](https://github.com/gollum/gollum)
    + [Nginx](https://en.wikipedia.org/wiki/Nginx)
    + [ExaBGP](https://github.com/Exa-Networks/exabgp)

    It's recommended to use a debian distro on the host machine, where most of the above software can be installed using package manager(s) with ease. On some distros (RHEL based for example) it might not be an easy task.

## Network

 - Install wiki anycast IP addresses `172.23.0.80/32` and `fd42:d42:d42:80::1/64` on the system
 - Assign a unicast IP address to be used by Nginx
 - Establish connectivity to the dn42 network

## Data replication

Site files are stored in a local [DVCS](https://en.wikipedia.org/wiki/Distributed_revision_control) repository (Git) on each node and replicated through a central server hosted by [XUU-DN42](https://io.nixnodes.net?t=person&l=XUU-DN42). 
Since gollum is built on top of Git, it is not overly complicated to keep the local site in sync with others, each site only triggers periodic pulls/pushes from/to the Git server. 

### Setup the repo

 - Clone the dn42 wiki repo:

    `git clone ssh://git@dn42.us/dn42/wiki <path>`

 - Contact [XUU-DN42](https://io.nixnodes.net?t=person&l=XUU-DN42) and ask for write access to the repo
 - Setup cron for periodic pull/push jobs for the repo (simple example):
     
     + **wiki-sync.sh**:

     ```sh
#!/bin/bash

WIKI_PATH=<repo path>
GIT=/usr/bin/git

cd "${WIKI_PATH}"
${GIT} push
${GIT} pull

exit 0
     ```

     + **Cron entry**:

     `*/10 * * * * <path>/wiki-sync.sh &> /dev/null`

       Running in 10 minute intervals is reasonable, if you choose to change this, please keep it in the range from 5 to 15 minutes.

## gollum

 - Install [gollum](https://github.com/gollum/gollum)
 - Start two gollum instances, read-only and read/write on `127.0.0.1`:
 
   Read/write (SSL only):
    ```
RACK_ENV=production gollum --css --host 127.0.0.1  --port 4568 <path>
    ```
   Read-only:
    ```
RACK_ENV=production gollum --css --host 127.0.0.1  --port 4567 --no-edit <path>
    ```

    Set `<path>` to the location where wiki Git repo was cloned. 

## Nginx reverse proxy

#### SSL

 - Setup your maintainer object according to [Automatic CA](https://internal.dn42/services/Automatic-CA)
 - Generate a [CSR](/services/Certificate-Authority) and send DNS Key Pin to [xuu@sour.is](mailto:xuu@sour.is): 
 - <AS> is the as number with the prefix `as` like `as64737-ca.wiki.dn42`

```
./ca.dn42 tls-gen \
   <AS>-<CC>(-<UID>).wiki.dn42 \
   EXAMPLE-MNT \
   mail@example.com \
   DNS:<AS>-<CC>(-<ID>).wiki.dn42,DNS:wiki.dn42,DNS:www.wiki.dn42,DNS:internal.dn42,DNS:www.internal.dn42
```

   Wait for a reply and then sign the certificate:

   `./ca.dn42 tls-sign wiki.dn42 MIC92-MNT`

#### Header

##### Site identification

A custom header `X-SiteID` identifies the site you're connecting to:

  - `add_header X-SiteID   '<AS>-<ISO country code>';`
     - <AS> is the as number prefixed with `as`

##### Enabling [HPKP](https://en.wikipedia.org/wiki/HTTP_Public_Key_Pinning)

  - Extract base64 encoded SPKI fingerprint from private key `wiki.key`:

    ```
openssl rsa -in wiki.key -outform der -pubout | openssl dgst -sha256 -binary | openssl enc -base64
    ```

  - Configure Nginx to send the fingerprint in header (SSL block):

    ```
add_header Public-Key-Pins  pin-sha256="<primary>"; pin-sha256="<backup>"; max-age=5184000; includeSubDomains';
    ```

   + `<primary>` - the fingerprint extracted from `wiki.key`
   + `<backup>`  - the CA fingerprint: `of00RDinhPeVRNnXm1jXQDagktOL75qQo1pT+xc7VIE=`

   Read more about this [here](https://developer.mozilla.org/en-US/docs/Web/Security/Public_Key_Pinning).

#### Domains

The proxy should accept the following domain names:

  - internal.dn42
  - wiki.dn42

Nginx should listen on a unicast address as well, so your site can be reached exclusively. Assign an IP address for the occasion and send it to [XUU-DN42](https://io.nixnodes.net?t=person&l=XUU-DN42) including your AS `<aut-num>` and the country code `<CC>` where your site is located. A forward DNS record will be created, pointing to the unicast IP address:

  - `<aut-num>`-`<CC>``(-<UID>)`.wiki.dn42 

#### Config example

```
ssl_protocols  TLSv1.2 TLSv1.1 TLSv1;
ssl_session_cache shared:SSL:2m;
 
ssl_ciphers   ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA;

ssl_prefer_server_ciphers   on;

upstream wiki { server 127.0.0.1:4567; }

server { 
        server_name internal.dn42 wiki.dn42 as<aut-num>-<cc>.wiki.dn42;

        listen 172.23.0.80:80 default;
        listen [fd42:d42:d42:80::1]:80 default;
        listen <unicast ipv4> 80;
        listen [<unicast ipv6>]:80;

        add_header X-SiteID                   '<aut-num>-<cc>';

        location / {
                proxy_pass http://wiki;
        }
}

upstream wikirw { server 127.0.0.1:4568; }


server { 
        server_name internal.dn42 wiki.dn42 as<aut-num>-<cc>.wiki.dn42;

        listen 172.23.0.80:443 ssl default;
        listen [fd42:d42:d42:80::1]:443 ssl default;
        listen <unicast ipv4> 443 ssl;
        listen [<unicast ipv6>]:443 ssl;

	ssl on;
        ssl_certificate      <path>/ssl.crt;  
        ssl_certificate_key  <path>/ssl.key;

        add_header strict-transport-security  "max-age=5184000; includeSubDomains";
        add_header Public-Key-Pins            'pin-sha256="<primary-pin>";pin-sha256="<backup-pin>";  max-age=5184000; includeSubDomains';
        add_header X-SiteID                   '<aut-num>-<cc>';

        location / {
                proxy_pass http://wikirw;
        }
}

```

## ExaBGP

#### Announcing

The prefix AS-PATH should show the announcement is originating from your AS. After peering ExaBGP to the nearest speaker(s), check if the prefix is routing properly inside your network. Try not to blackhole the passing traffic (e.g. no static routes to `172.23.0.80/32`). Test the whole thing by shutting down nginx/gollum and watch what happens.

#### Configuration

```
# exabgp.conf

group gollum-watchdog {
  neighbor <peer1> {
    router-id x.x.x.x;
    local-address <source-address>;
    local-as <ownas>;
    peer-as <peeras>;
  }

  ## (example ipv4) peer with one of our iBGP speakers:
  neighbor 172.22.0.1 {
    router-id 172.23.0.80;
    local-address 172.22.0.2;
    local-as 123456;
    peer-as 123456;
  }

  ## (example ipv6) peer with one of our iBGP speakers:
  neighbor fd42:4992:6a6d::1 {
    router-id 172.23.0.80;
    local-address fd42:4992:6a6d::2;
    local-as 123456;
    peer-as 123456;
  }

  ## ...

  process watch-gollum {
     run <path>/gollum-watchdog.sh;
  }
}

```

#### Watchdog script

Watchdog runs in an infinite loop, sending the appropriate commands to stdout. [ExaBGP](https://github.com/Exa-Networks/exabgp) attaches to the process' stdout and listens for instructions. Watchdog sends either a route announce or widthdraw.

Run `gollum-watchdog.sh` in a shell first to validate it's working:

```sh
#!/bin/bash

CURL=curl

## url's to check (all listed must be alive to send announce)
URL=("http://172.23.0.80" "https://172.23.0.80" "http://[fd42:d42:d42:80::1]" "https://[fd42:d42:d42:80::1]")

ROUTE='172.23.0.80/32'
## the anycast v6 route (/64 due to prefix size limits)
ROUTE6='fd42:d42:d42:80::/64'
                        
## the next-hop we'll be advertising to neighbor(s)
NEXTHOP='<source-address>' 
NEXTHOP6='<source-address-v6>'

## regex match this keyword against HTTP response from curl
VALIDATE_KEYWORD='gollum'

INTERVAL=60

###########################
                
RUN_STATE=0             
                        
check_urls() {          
        for url in "${URL[@]}"; do
        
                ## workaround curl errno 23 when piping
                http_response=`${CURL} --insecure -g -s -L -o - "${url}"`
                        
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
                        echo "announce route ${ROUTE6} next-hop ${NEXTHOP6}"
                }       
        else    
                check_urls || {
                        RUN_STATE=0
                        echo "withdraw route ${ROUTE} next-hop ${NEXTHOP}"
                        echo "withdraw route ${ROUTE6} next-hop ${NEXTHOP6}"
                }       
        fi

        sleep ${INTERVAL}
        
done    

exit 0
```

#### Run

  Normally SIGUSR1 to the exabgp process triggers a configuration update, but at occasion the process might need to be restarted - since its gracefull shutdown can be glitchy , this might be a bit difficult. Sending SIGKILL to the child(ren) and immediately after, the parent, does the job (quick-and-dirty). 

`USAGE: /etc/exabgp/run.sh [start|stop|restart]`

```sh
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
        echo ${cpid} > ${PID_FILE}
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



    
