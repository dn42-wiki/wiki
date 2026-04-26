# Distributed Wiki

This wiki is mirrored by multiple operators across both the public internet and on dn42.
Current operators providing the wiki service can be found on the page footer.

## Hosting the wiki

For hosting the wiki one will need to decide whether they desire to host a public internet mirror, join the wiki.dn42 anycast or both.
The idea is to deploy mirrors across dn42 using [anycast](https://en.wikipedia.org/wiki/Anycast) addressing (BGP), thus providing redundancy, load-balancing and improved access times to the wiki.

## Prerequisites

  - Stable 24/7 connection to dn42.
  - Stable host machine, present in dn42.
  - Solid networking/sysadmin skills (to keep the system as a whole stable).

    In contrast with the general spirit in dn42, this service should NOT be deployed by unskilled members for the purpose of learning and exploring - since it is the primary source of information related to the project, it should not be used as a playground. 

  - Wiki Software (choose one):
    + [dn42-wiki-go](https://github.com/iedon/dn42-wiki-go)  (Recommended)
    + [wiki-ng](https://git.dn42.dev/wiki/wiki-ng)
    + [gollum](https://github.com/gollum/gollum)

  - Other Software:
    + [Git](https://en.wikipedia.org/wiki/Git_(software))
    + [Nginx](https://en.wikipedia.org/wiki/Nginx)
    + [ExaBGP](https://github.com/Exa-Networks/exabgp)

    It's recommended to use a debian distro on the host machine, where most of the above software can be installed using package manager(s) with ease. On some distros (RHEL based for example) it might not be an easy task.


The local webserver is to be monitored with a simple [shell script](/services/Distributed-Wiki#watchdog-script) working [in conjunction](/services/Distributed-Wiki/#exabgp) with [ExaBGP](https://github.com/Exa-Networks/exabgp), announcing/withdrawing the assigned route if the service is up/down.
Nginx acts as a reverse proxy and handles the encryption.

## Network

 - Install wiki anycast IP addresses `172.23.0.80/32` and `fd42:d42:d42:80::1/64` on the system
 - Assign a unicast IP address to be used by Nginx
 - Establish connectivity to the dn42 network

## Data replication

Site files are stored in a local [DVCS](https://en.wikipedia.org/wiki/Distributed_revision_control) repository (Git) on each node and replicated through a central server.
Since the wiki is hosted on top of Git, it is not overly complicated to keep the local site in sync with others, each site only triggers periodic pulls/pushes from/to the Git server. 

### Setup the repo

 - Clone the dn42 wiki repo:

    `git clone git@git.dn42.dev:wiki/wiki.git <path>`

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

   Running in 10 minute intervals is reasonable, if you choose to change this, please keep it in the range from 5 to 15 minutes. Also consider using dn42notifyd to get a callbacks instead.

## Nginx reverse proxy

### SSL

 - Generate an SSL certificate using the main Certificate authority using a method that supports anycasted IP addresses.

#### Header

##### Site identification

A custom header `X-SiteID` identifies the site you're connecting to:

  - `add_header X-SiteID   '<AS>-<ISO country code>';`
     - \<AS> is the as number prefixed with `as` like `as64737`

##### Enabling [HPKP](https://en.wikipedia.org/wiki/HTTP_Public_Key_Pinning)

  - Extract base64 encoded SPKI fingerprint from private key `wiki.key`:

    ```sh
    openssl rsa -in wiki.key -outform der -pubout | openssl dgst -sha256 -binary | openssl enc -base64
    ```

  - Configure Nginx to send the fingerprint in header (SSL block):

    ```conf
    add_header Public-Key-Pins  pin-sha256="<primary>"; pin-sha256="<backup>"; max-age=5184000; includeSubDomains';
    ```

   + `<primary>` - the fingerprint extracted from `wiki.key`
   + `<backup>`  - the CA fingerprint: `of00RDinhPeVRNnXm1jXQDagktOL75qQo1pT+xc7VIE=`

   Read more about this [here](https://developer.mozilla.org/en-US/docs/Web/Security/Public_Key_Pinning).

#### Domains

The proxy should accept the following domain names:

  - internal.dn42
  - wiki.dn42

#### Config example

```conf
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

### Announcing

The prefix AS-PATH should show the announcement is originating from your AS. After peering ExaBGP to the nearest speaker(s), check if the prefix is routing properly inside your network. Try not to blackhole the passing traffic (e.g. no static routes to `172.23.0.80/32`). Test the whole thing by shutting down nginx/gollum and watch what happens.

#### Configuration

```conf
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

## Setting up the wiki software
See the documentation below for both gollum and dn42-wiki-go.


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


## dn42-wiki-go

### Introduction

[dn42-wiki-go](https://github.com/iedon/dn42-wiki-go) is a lightweight, Git-backed wiki engine designed for DN42. It is based on [wiki-ng](https://git.dn42.dev/wiki/wiki-ng), aims to replace the old Gollum-based DN42 distributed wiki.

It can serve pages live through its built-in Go HTTP server or generate a fully static HTML export for external hosting. All content is stored in a Git repository, making it easy to replicate across nodes or run in disconnected environments.

### Live Version
- [https://wiki.dn42/](https://wiki.dn42/) (DN42 Access)
- [https://dn42.jp/](https://dn42.jp/) (Clearnet)

![Screenshot of iEdon DN42 Wiki Go](/services/images/iedon-dn42-wiki-go.png)

### Operating Modes

You can run `dn42-wiki-go` in three different ways:

1. **Run static build once then exit (`--build` or `live=false`)**  
   The App renders all Markdown files into HTML under outputDir and exits.  
   Best for setups where your own cron job handles Git sync and file publishing.

2. **Live mode without reverse proxy (`live=true`)**  
   The built-in HTTP server directly serves pages, assets, and APIs.  
   Suitable for simple deployments.

3. **Live mode behind a reverse proxy**  
   The reverse proxy (nginx, Caddy, HAProxy, etc.) serves the generated files, and only API endpoints are forwarded to the App.  
   See `config.example.json` for example configs and `nginx-vhost.conf` for a reverse-proxy reference.

   **Recommended for production and anycast nodes**.

### Features

- Live mode with automatic Markdown rendering and scheduled Git pull/push.
- Static mode for fully pre-built HTML exports.
- Optional in-browser editor with commit metadata (author, message prefix, remote IP).
- Webhook endpoints for remote pull/push triggers and optional polling integration(see `dn42notifyd`).
- Themeable templates and bundled UI assets.
- Designed for distributed, multi-node and anycast environments.

### Quick Start

Pre-built binaries are available in the [GitHub releases](https://github.com/iedon/dn42-wiki-go/releases).

Please do not forget to clone the repository to copy `config.example.json`(to `config.json`) and the `template` folder. They should be put together in the same production directory.

#### Manual Build

1. Install Go 1.24+ and ensure the git executable is available in PATH.
2. Copy the example config:
   cp config.example.json config.json
   Then edit the settings you need.
3. Build for your platform (example: Linux amd64):
   ```bash
   export GOOS=linux
   export GOARCH=amd64
   ./build.sh
   ```

Determine which user is used to run `dn42-wiki-go`, then create `~/.gitconfig` for this user, which will be used by `git`.

For example:

```ini
[user]
        email = noreply@dn42.jp    
        name = IEDON.DN42 Wiki Mirror(116)
```

To create the isolated, low-privileged user `dn42-wiki-go`, you may run:
```bash
# This user cannot log in or get a shell.
# This user has /opt/dn42-wiki-go as working directory.
# No default home folder created automatically.
sudo useradd -r -s /usr/sbin/nologin -d /opt/dn42-wiki-go -M dn42-wiki-go
```

Both systemd socket enabler and UNIX domain socket are supported, check example configuration files: `dn42-wiki-go.socket` and `dn42-wiki-go.service`.

If you would like to use with Docker: `Dockerfile` and `docker-compose.yml` are also provided, bind proper directories and map `config.json` for the App to use, then you are all set.

### Webhook Endpoints

When `webhook.enabled` = true, the server exposes:

- GET | POST /api/webhook/pull  
  Runs git pull and rebuilds the cached HTML.

- GET | POST /api/webhook/push  
  Pushes local commits to the remote.

If `webhook.secret` is set, requests must include an Authorization header that matches the secret.
If `webhook.secret` is empty, a random secret will be generated on startup to secure the endpoint (used for polling).  

#### Polling Integration

When `webhook.polling.enabled` = true, the server registers with a remote notify service and triggers `/api/webhook/pull` whenever a refresh completes.

This is compatible with `dn42notifyd` and similar tools.

### Configuration Reference

All settings are provided through a JSON file. Below is a concise reference of all options.

#### Runtime

- `live` *(bool, default `false`)*:  
  true -> run HTTP server and render on demand.  
  false -> render once to outputDir and exit.

- `editable` *(bool, default `false`)*:  
  Enables in-browser editing and write operations.

- `listen` *(string, default `":8080"`)*:  
  TCP address (host:port) or UNIX socket (unix:/path).

  Advanced: See example systemd files `dn42-wiki-go.socket` and `dn42-wiki-go.service` in the repository.

- `baseUrl` *(string, optional)*:  
  URL prefix when hosting under a subdirectory.

- `siteName` *(string, default `"DN42 Wiki Go"`)*:  
  Display name of the wiki.

#### Git
- `git.binPath` *(string, default `git`)*: Path to the Git executable.
- `git.remote` *(string, default empty)*: Remote URL. Leave empty for standalone/local repositories.
- `git.localDirectory` *(string, default `./repo`)*: Directory where the wiki repository is cloned or initialised.
- `git.pullIntervalSec` *(int, default `3600`)*: Seconds between background `git pull` operations in live mode. Disabled if no remote is set.
- `git.author` *(string, default `"Anonymous <anonymous@localhost>"`)*: Author string used for commits generated by the application.
- `git.commitMessagePrefix` *(string, default empty)*: Optional prefix prepended verbatim to commit messages supplied by users.
- `git.commitMessageAppendRemoteAddr` *(string, default empty)*: Optional suffix appended when a request carries a remote address. If the value contains `%s` it is treated as a `fmt` format string; otherwise it is concatenated.

#### Webhook
- `webhook.enabled` *(bool, default `false`)*: Expose webhook endpoints on the main HTTP server.
- `webhook.secret` *(string, default empty)*: Shared secret expected in the `Authorization` header. If empty, a random secret is generated on startup.
- `webhook.polling.enabled` *(bool, default `false`)*: Keep a registration active with the remote notification service and trigger periodic pulls.
- `webhook.polling.endpoint` *(string, default empty)*: URL of the notification service (eg. Usage with [dn42notifyd](https://git.dn42.dev/dn42/dn42notifyd): `https://git.dn42/dn42notify/poll`).
- `webhook.polling.callbackUrl` *(string, default empty)*: Public URL for `/api/webhook/pull`. Required when `webhook.polling.enabled` is `true`.
- `webhook.polling.pollingIntervalSec` *(int, default `3600`)*: Seconds between refresh attempts. Must be positive when polling is enabled.
- `webhook.polling.skipRemoteCert` *(bool, default `false`)*: Insecure: Skip TLS verification.

#### Paths and templating
- `outputDir` *(string, default `./dist`)*: Destination directory for static builds or asset exports.
- `templateDir` *(string, default `./template`)*: Location of layout templates and static assets bundled into the server/UI.
- `homeDoc` *(string, default `Home.md`)*: Repository document to treat as the home page. Normalised to a `.md` path relative to the repo root.
- `privatePagesPrefix` *(array of strings, default empty)*: Request to routes started with these prefixes will be blocked.

#### Layout and footer
- `ignoreHeader` *(bool, default `false`)*: Skip loading `_Header.md` when `true`. Leave `false` to include the fragment when present.
- `ignoreFooter` *(bool, default `false`)*: Skip `_Footer.md` when `true`; otherwise render it if available.
- `serverFooter` *(string, default empty)*: Markdown snippet rendered into the global footer at runtime.

#### TLS
- `enableTLS` *(bool, default `false`)*: Serve HTTPS using the provided certificate and key.
- `tlsCert` *(string)*: Path to the TLS certificate. Required only when `enableTLS` is true.
- `tlsKey` *(string)*: Path to the TLS private key. Required when `enableTLS` is true.

#### Logging and client IP handling
- `logLevel` *(string, default `info`)*: Minimum log level (`debug`, `info`, `warn`, or `error`).
- `trustedProxies` *(array of strings, default empty)*: CIDR blocks or literal IPs that are trusted to populate `X-Forwarded-For`.
- `trustedRemoteAddrLevel` *(int, default `1`)*: Number of additional trusted hops to peel off when deriving the end-user IP from the forwarded chain. Values less than `1` are coerced to `1` during load.

### Notes

- live = true requires write access to the Git repo for local commits.
- With no remote configured, `dn42-wiki-go` initializes a local-only repository.
- Template changes require restarting the server or rebuilding static output.

