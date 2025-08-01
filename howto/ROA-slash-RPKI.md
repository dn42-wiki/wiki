## What is ROA?

A Route Origination Authorization details which AS is authorised to advertise which originating IP prefixes. A ROA may also include prefix length information.

## What is RPKI?

Resource Public Key Infrastructure is basically a framework for securing the routing infrastructure.  
It provides a way to connect number resource information to a trust anchor.

## What is RTR?

The Resource Public Key Infrastructure (RPKI) to Router Protocol provides a way for a router to access RPKI validation information.  
It provides the router with validity information regarding prefix origination:  

* VALID  
  The route announcement is covered by a ROA and the announcing AS is validated
* INVALID  
  The route announcement is covered by a ROA and the announcing AS is invalid (possibly hijacking)
* UNKNOWN  
  There exists no ROA for the route announcement

## How can I implement ROA on dn42?

On dn42 we generate ROA information from the dn42 registry.  
ROA json/bird files can be generated using [dn42regsrv](https://git.burble.com/burble.dn42/dn42regsrv).
It is also possible to integrate this with a RTR cache server such as [gortr](https://github.com/cloudflare/gortr).

### dn42regsrv 

You can find a hosted example of dn42regsrv at <https://explorer.burble.com/>

Instructions on how to host dn42regsrv yourself can be found on the git repo of [dn42regsrv](https://git.burble.com/burble.dn42/dn42regsrv). 

You can also run dn42regsrv via docker (then available at 127.0.0.1:8042):
```sh
git checkout https://git.burble.com/burble.dn42/dn42regsrv.git .
cd contrib/docker
./build.sh
docker-compose up -d
```

Documentation for the api endpoints can be found here: <https://git.burble.com/burble.dn42/dn42regsrv/src/master/API.md>

### gortr

burble kindly provides ready-to-use files for gortr here:

<https://dn42.burble.com/roa/dn42_roa_46.json>

You can use these to simply run gortr via docker:

```sh
docker run -ti -p 8082:8082 cloudflare/gortr -cache https://dn42.burble.com/roa/dn42_roa_46.json -verify=false -checktime=false -bind :8082
```

### rtrtr

rtrtr is a RTR server from NLNet Labs. It's compatible with the dn42regsrv ROA-JSON or burbles provided one (https://dn42.burble.com/roa/dn42_roa_46.json) too. 

NLNet Labs provides an official docker image. You just have to bind mount a suitable configuration file:

```sh
docker run -d -v /etc/rtrtr.conf:/etc/rtrtr.conf -p 323:323/tcp nlnetlabs/rtrtr -c /etc/rtrtr.conf
```

This is a working configuration file for dn42. Maybe change the listen addresses:

```conf
log_level = "debug"
log_target = "stderr"
http-listen = []
[units.dn42-json]
type = "json"
uri = "https://dn42.burble.com/roa/dn42_roa_46.json"
refresh = 600
[targets.dn42-rtr]
type = "rtr"
listen = ["0.0.0.0:323", "[::]:323"]
unit = "dn42-json"
```

For more information cosult the official documentation: <https://rtrtr.docs.nlnetlabs.nl/en/stable/>

### Kioubit's DN42 Registry Wizard

[DN42 Registry Wizard](https://github.com/Kioubit/dn42_registry_wizard) is a comprehensive tool for DN42 registry interactions. **Unlike other solutions, it can parse the registry and host an RTR server all-in-one** without requiring separate components.

#### All-in-One RTR Server

```sh
# Clone the DN42 registry
git clone https://git.dn42.dev/dn42/registry.git

# Start RTR server directly from registry
./registry_wizard <path to registry> rtr 

# Setup a cronjob to continously update the registry and notify registry_wizard
git fetch --all
git reset --hard origin/master
kill -SIGUSR1 "$(pidof 'registry_wizard')"
```

```
Usage: registry_wizard <registry_root> rtr [OPTIONS]

Options:
  -p, --port <port>        Port to listen on [default: 9323]
      --refresh <refresh>  RTR refresh timing [default: 3600]
      --expire <expire>    RTR expire timing [default: 7200]
      --retry <retry>      RTR retry timing [default: 600]
  -h, --help               Print help
```

### Other tools / generators
- bauen1's dn42-roagen: <https://gitlab.com/bauen1/dn42-roagen>
- Kioubit's registry wizard: <https://github.com/Kioubit/dn42_registry_wizard>
- chuangzhu's pure bash script: <https://paste.sr.ht/~chuang/e98d2fe791de68a6cf5aade7877cd0dbc1cdb84e>

### This is all to complicated, is there an easy all-in-one package for RTR?

TODO: Publish docker-compose-yml to git for gortr+dn42regsrv

### How do I integrate RTR with my BGP implementation

You have to consult the documentation of your implementation for that. We will provide configuration examples on the specific pages.
