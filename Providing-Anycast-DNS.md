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

## Persons providing anycast DNS

| **Person**  | **Region** | **AS** | **Unicast Address**       | **Comments**       |
|-------------|---|:------:|:----------------------------------:|--------------------|
| siska       |EU | 76103  | nixnodes.root.dn42 (172.22.177.8)  | authoritative only |
| siska       |EU | 76103  | ns1.nixnodes.dn42  (172.22.177.2)  | caching            |
| siska       |EU | 76105  | ns2.nixnodes.dn42  (172.22.177.1)  | caching            |
| xuu         |US | 64737  | xuu.root.dn42   (172.22.141.132)   ||
| Nurtic-Vibe |EU | 4242420123 | ns1.grmml.dn42 (172.23.149.20) || 

# IPv6 DNS

**IP:** fd42:d42:d42:53::1/64

[IPv6 Anycast Info](https://dn42.net/IPv6-Anycast)

## Persons providing anycast DNS for IPv6


| **Person**  | **Region**    | **AS** | **Unicast Address**           | **Comments** |
|-------------|---|:---------:|:--------------------------------------:|--------------|
| xuu         |US |64737      | xuu.root.dn42 (fdea:a15a:77b9:d42::53) ||             
| Nurtic-Vibe |EU |4242420132 | ns1.grmml.dn42 (fd42:23:149:cccc::53)  ||
