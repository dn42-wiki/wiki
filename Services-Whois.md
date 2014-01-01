# Whois registry
**aka** _The registry_.

The registry contains:

  * AS numbers assignations
  * Subnet assignations
  * DNS root zone for `dn42.`

## Web interface

Nixnodes provides a nice web interface, that allows you to **add/edit records** easily.  It is available at https://io.nixnodes.net/?registry. A full guide is available at [Getting started](Getting-started-with-dn42#Fill-in-the-registry).

### Authentication

To add or edit records with the web interface, authentication is done thanks to **maintainer objects**. Each maintainer object has a password associated to it.

The password are not stored in cleartext in the registry: a hash is computed from the password and the name of the maintainer object. To generate such a hash (e.g. in case you forgot your password), use https://io.nixnodes.net/nctlio.php?m=dnr&gen=mypassword&mnt=MYMAINTAINER-MNT

### Misc

A read-only interface is also available at http://ix.ucis.dn42/dn42/ ([public](http://ix.ucis.nl/dn42/) or 172.22.166.3). The used PHP scripts are available from UFO a.k.a. Ivo at request.

## Address space

There is nice 3djs visualisation showing current address space usage: http://dataviz.polynome.dn42/dn42-netblock-visu/registry.html ([public](http://109.24.208.244:8888/dn42-netblock-visu/registry.html) or 172.23.184.98). The input data is taken from the registry.

Another visualisation shows the prefixes seen by BGP: http://dataviz.polynome.dn42/dn42-netblock-visu/index.html ([public](http://109.24.208.244:8888/dn42-netblock-visu/index.html) or 172.23.184.98).

## Whois daemons
 * welterde: thinkbase.srv.welterde.de (46.4.248.201)
 * fritz: whois.fritz.dn42 (172.22.119.139)
 * nixnodes: whois.nixnodes.dn42 (172.22.177.77)

### Usage
```sh
whois -h $host $query
```
### Using a whois config
```sh
$ cat /etc/whois.conf 
\.dn42$           172.22.177.77
\-DN42$           172.22.177.77
as[64512-65534]   172.22.177.77
as[76100-76199]   172.22.177.77
```
You can then use whois without specifying the server. Works at least with Marco d'Itri's whois client.

### Running your own whoisd
```sh
cd /home/some/path/to/store/branch
sudo aptitude install ruby rubygems
sudo gem install netaddr
cd whoisd/ruby
sudo ruby whoisd.rb nobody
```


## Monotone
Monotone is an distributed revision control system. Monotone tracks revisions to files, groups sets of revisions into changesets, and tracks history across renames. The design principle is distributed operation making heavy use of cryptographic primitives to track file revisions (via the SHA-1 secure hash) and to authenticate user actions (via RSA cryptographic signatures). Each participant maintains their own revision history store in a local SQLite database. Monotone is especially strong in its support of a diverge/merge workflow, which it achieves in part by always allowing commit before merge. Revisions are exchanged using the custom netsync protocol which shares some conceptual ground with rsync and cvs.
 * [Website](http://monotone.ca/)
 * [Tutorial](http://monotone.ca/docs/Tutorial.html)

### Monotone servers
 * crest: mtn.crest.dn42
 * welterde: headend.srv.welterde.de(46.4.248.203)
 * somerandomnick: mtn1.srn.dn42(172.22.131.102)
 * dracoling: dn42.smrsh.net (net.smrsh.dn42)

### Monotone branches
 * net.dn42.registry: Contains the registry and some related code

### Setup
```sh
mtn genkey you@domain.tld
mtn pubkey you@domain.tld # send the output to some $monotone_server operator(do NOT send the keypair!)
mtn clone 'mtn://$monotone_server/?net.dn42.*' --branch net.dn42.registry
cd net.dn42.registry
$add_your_objects
mtn add --unknown
mtn ci -k you@domain.tld
mtn sync
```