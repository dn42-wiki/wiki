[[_TOC_]]


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
ROA json/bird files can be generated using [dn42regsrv](https://git.dn42.us/burble/dn42regsrv).
It is also possible to integrate this with a RTR cache server such as [gortr](https://github.com/cloudflare/gortr).

### dn42regsrv 

You can find a hosted example of dn42regsrv at https://explorer.burble.com/ 

Instructions on how to host dn42regsrv yourself can be found on the git repo of [dn42regsrv](https://git.dn42.us/burble/dn42regsrv). 
     
You can also run dn42regsrv via docker (then available at 127.0.0.1:8042):

    git checkout https://git.dn42.us/burble/dn42regsrv.git .
    cd contrib/docker
    docker-compose build
    docker-compose up -d
  
Documentation for the api endpoints can be found here: https://git.dn42.us/burble/dn42regsrv/src/master/API.md

### gortr

burble kindly provides ready-to-use files for gortr here:

https://dn42.burble.com/roa/dn42_roa_46.json

You can use these to simply run gortr via docker:

    docker run -ti -p 8082:8082 cloudflare/gortr -cache https://dn42.burble.com/roa/dn42_roa_46.json -verify=false -checktime=false -bind :8082

### This is all to complicated, is there an easy all-in-one package for RTR?

TODO: Publish docker-compose-yml to git for gortr+dn42regsrv

### How do I integrate RTR with my BGP implementation

You have to consult the documentation of your implementation for that. We will provide configuration examples on the specific pages.