# Whois
**aka** _The registry_.

## Whois daemons
 * welterde: thinkbase.srv.welterde.de (46.4.248.201)
 * planetcyborg: eudora.dn42.planetcyborg.de (172.22.192.14)
 * planetcyborg: thaleia.dn42.planetcyborg.de (172.22.192.94)
 * nixnodes: whois.nixnodes.dn42 (172.22.177.77)

### Usage
```sh
whois -h $host $query
```

### Running your own whoisd
```sh
cd /home/some/path/to/store/branch
sudo aptitude install ruby rubygems
sudo gem install netaddr
cd whoisd/ruby
sudo ruby whoisd.rb nobody
```

## Web access
* UFO: http://ix.ucis.dn42/dn42/ ([public](http://ix.ucis.nl/dn42/) or 172.22.166.3) (read only)

The used PHP scripts are available from UFO a.k.a. Ivo at request.

## Monotone
Monotone is an distributed revision control system. Monotone tracks revisions to files, groups sets of revisions into changesets, and tracks history across renames. The design principle is distributed operation making heavy use of cryptographic primitives to track file revisions (via the SHA-1 secure hash) and to authenticate user actions (via RSA cryptographic signatures). Each participant maintains their own revision history store in a local SQLite database. Monotone is especially strong in its support of a diverge/merge workflow, which it achieves in part by always allowing commit before merge. Revisions are exchanged using the custom netsync protocol which shares some conceptual ground with rsync and cvsup.
 * [Website](http://monotone.ca/)
 * [Tutorial](http://monotone.ca/docs/Tutorial.html)

### Monotone servers
 * crest: mtn.crest.dn42
 * welterde: headend.srv.welterde.de(46.4.248.203)
 * somerandomnick: mtn1.srn.dn42(172.22.131.102)

### Monotone branches
 * net.dn42.registry: Contains the registry and some related code

### Setup
```sh
mtn genkey you@domain.tld
mtn pubkey you@domain.tld # send the output to some $monotone_server operator(do NOT send the keypair!)
mtn --db /home/some/path/to/db.mtn db init
mtn --db /home/some/path/to/db.mtn pull -k "" $monotone_server "*"
mtn --db /home/some/path/to/db.mtn --branch net.dn42.registry co /home/some/path/to/store/branch
cd /home/some/path/to/store/branch
$add_your_objects
mtn add --unknown
mtn ci -k you@domain.tld
mtn sync -k you@domain.tld $monotone_server "*"  # once the monotone server operator has deployed your key
# Note: that "*" is not there by accident. it specifies, which branches to sync, the default is to sync none!
```