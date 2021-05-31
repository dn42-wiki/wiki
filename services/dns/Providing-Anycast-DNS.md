#DEPRECATED - Please have a look at [Hierarchical DNS](https://internal.dn42/Hierarchical-DNS) instead

You may want to participate in the anycast DNS cloud.

## Configuration

Configuration requirements for all members of the anycast group are:
 * maintain your own zones based on whois database (scripts included in monotone repository)
 * allow recursion (including `.`)
 * listen on a unicast IP too for testing/debugging reasons
 * with bind, please use ```minimal-responses yes;``` (goes into ```options```/```view```)

It is _really_ good to hang around in [IRC](/IRC) to get things sorted out, if something doesn't work. Letting some people test your DNS behavior before joining the anycast-group is considered best practice - better safe than sorry.

 * **IP:** 172.23.0.53
 * **Announciation Subnet:** 172.23.0.53/32

### Generating Zone Files

There are a few different scripts for generating zone files. They have been written in a few different languages. Please keep in mind that RFC 2317 is what keeps people from registering a /24 _just to have RDNS_, so scripts that support it have a positive effect on address space usage.

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
|zonegen.tinydns.sh | Bash |
|zongen.v2.sh | Sh | Author: Martin89, forward & reverse dns + RFC 2317

## Persons providing anycast DNS

| **Person**  | **Region** | **AS** | **Unicast Address**       | **Comments**       |
|-------------|---|:------:|:----------------------------------:|--------------------|
| siska       |SI | 76103  | a.resolvers.nic.dn42 (172.22.177.70)  ||
| xuu         |UT,US | 64737  | xuu.root.dn42   (172.22.141.132)   ||
| xuu         |ON,CA | 64737  | souris.root.dn42 (172.22.141.180)  ||          
| Nurtic-Vibe |EU | 4242420123 | ns1.grmml.dn42 (172.23.149.20) || 
| MWD         |AU | 4242420002 | nsr1.mwd.dn42 (172.23.227.20) ||
| Fritz       | ?? | 64712 | ?? (??) | Advertised over bgp |
| prauscher   | DE | 64720 | prauscher.root.dn42 (172.22.120.1) | advertised in BGP |
| hax404      | DE | 76114 | chero.hax404.dn42 (172.23.136.65) | advertised in BGP|
| psclrnnrt   | DE | 4242420205 | nsc421.root6.dn42 (172.23.65.5) |
| florianb    | AT | 4242423955 | resolver.flo.dn42 (172.20.2.65) | advertisted in BGP |

# IPv6 DNS

**IP:** fd42:d42:d42:53::1/64

[IPv6 Anycast Info](/services/IPv6-Anycast)

## Persons providing anycast DNS for IPv6


| **Person**  | **Region**    | **AS** | **Unicast Address**           | **Comments** |
|-------------|---|:---------:|:--------------------------------------:|--------------|
| xuu         |UT,US| 64737      | xuu.root.dn42 (fdea:a15a:77b9:d42::53) ||   
| xuu         |ON,CA| 64737 | souris.root.dn42 (fdea:a15a:77b9:53::1) | |
| Nurtic-Vibe |EU |4242420123 | ns1.grmml.dn42 (fd42:23:149:cccc::53)  ||
| hax404 | DE | 76114 | chero.hax404.dn42 (fd58:eb75:347d:101::1) ||
| florianb | AT | 4242423955 | resolver.flo.dn42 (fd42:d42:d42:53::1) | advertisted in BGP |