You may want to participate in the anycast DNS cloud.

## Configuration

Configuration requirements for all members of the anycast group are:
 * maintain your own zones based on whois database (scripts included in monotone repository)
 * allow recursion (including `.`)
 * listen on a unicast IP too for testing/debugging reasons
 * with bind, please use ```minimal-responses yes;``` (goes into ```options```/```view```)

It is _really_ good to hang around in [[IRC|Services IRC]] to get things sorted out, if something doesn't work. Letting some people test your DNS behavior before joining the anycast-group is considered best practice - better safe than sorry.

 * **IP:** 172.22.0.53
 * **Announciation Subnet:** 172.22.0.53/32

### Generating Zone Files

There are a few different scripts for generating zone files. They have been written in a few different languages. 

| **Script** | **Language** | **Notes** |
|---------------------|--------------|-----------|
|rfc2317.rb | Ruby |
|subnettr.py | Python 3 | Author: xuu, forward & reverse dns + RFC 2317
|zonegen.bind.php | PHP |
|zonegen.bind.sh | Bash |
|zonegen.rb | Ruby |
|zonegen.rdns.bind.sh | Bash |
|zonegen.rdns.tinydns.sh | Bash |
|zonegen.rev.bind.sh | Bash |
|zonegen.smallblockrdns.tinydns.sh | Bash |
|zonegen.tinydns.sh | Bash

## Persons providing anycast DNS

| **Person**  | **Region** | **AS** | **Unicast Address**       | **Comments**       |
|-------------|---|:------:|:----------------------------------:|--------------------|
| siska       |EU | 76103  | nixnodes.root.dn42 (172.22.177.8)  | authoritative only |
| siska       |EU | 76103  | ns1.nixnodes.dn42  (172.22.177.2)  | caching            |
| siska       |EU | 76105  | ns2.nixnodes.dn42  (172.22.177.1)  | caching            |
| xuu         |UT,US | 64737  | xuu.root.dn42   (172.22.141.132)   ||
| xuu         |ON,CA | 64737  | souris.root.dn42 (172.22.141.180)  ||          
| Nurtic-Vibe |EU | 4242420123 | ns1.grmml.dn42 (172.23.149.20) || 
| MWD         |AU | 4242420002 | nsr1.mwd.dn42 (172.23.227.20) ||
| Martin89    | FR, DE | 64713 | dns-anycast-r.martin89.dn42 (172.22.113.10) | More describtion on http://martin89.dn42 |
| Fritz       | ?? | 64712 | ?? (??) | Advertised over bgp |
| PyroPeter   | EU | 76115 | nscache.pyropeter.dn42 (172.23.196.3) ||

# IPv6 DNS

**IP:** fd42:d42:d42:53::1/64

[IPv6 Anycast Info](https://dn42.net/IPv6-Anycast)

## Persons providing anycast DNS for IPv6


| **Person**  | **Region**    | **AS** | **Unicast Address**           | **Comments** |
|-------------|---|:---------:|:--------------------------------------:|--------------|
| xuu         |UT,US| 64737      | xuu.root.dn42 (fdea:a15a:77b9:d42::53) ||   
| xuu         |ON,CA| 64737 | souris.root.dn42 (fdea:a15a:77b9:53::1) | |
| Nurtic-Vibe |EU |4242420123 | ns1.grmml.dn42 (fd42:23:149:cccc::53)  ||
