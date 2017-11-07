# IRC
> TLDR: irc.hackint.dn42, #dn42

We have an IRC Chatroom on the [hackint-Network](http://www.hackint.org). It is reachable from within DN42, ChaosVPN and the public internet. While a plain text connection is possible it is recommended to connect via TLS on port 9999.

There's a little [statistic script](https://dev.0l.dn42/stats/) running hourly.


## Servers

### via dn42
| Hostname                                 |  SSL        | IPv4                       | IPv6         |
|:------------------------------------------|:------ |:-------------------------- |:------------ |
| [irc.hackint.dn42](irc://irc.hackint.dn42)|  Yes    | 172.23.96.42, 172.22.2.235 |  fd42:d42:d42:6667::1           | fd42:23:cda::6667
| [irc.dn42](ircs://irc.dn42)               |  Yes   | many                        | also many |
| [irc.2f30.dn42](ircs://irc.2f30.dn42)       |  Yes    | 172.23.35.0.2, 172.23.0.10 | N/A |



### via chaosvpn
| Hostname                                          | IPv4                       | IPv6         |
|:------------------------------------------------- |:-------------------------- |:------------ |
| [irc.hackint.hack](irc://irc.hackint.hack) | 172.31.0.30 | - |

### via public internet
| Hostname                                          | Comment                    |
|:------------------------------------------------- |:-------------------------- |
| [irc.hackint.org](irc://irc.hackint.org)                                   |                            |
| [irc.eu.hackint.org](irc://irc.eu.hackint.org)                               | european server pool       |
| [irc.us.hackint.org](irc://irc.us.hackint.org)                                | american server pool       |


## Bouncer Offers

* stv0g: offering ZNC accounts (ask in IRC, [Webinterface](https://dev.0l.dn42/znc/))
