# IRC
> TLDR: irc.hackint.dn42, #dn42

We have an IRC Chatroom on the [hackint-Network](http://www.hackint.org). It is reachable from within DN42, ChaosVPN and the public internet. While a plain text connection is possible it is recommended to connect via TLS on port 9999.

There's a little [statistic script](https://dev.0l.dn42/stats/) running hourly.


## Servers

### via dn42
| Hostname                                          | IPv4                       | IPv6         |
|:------------------------------------------------- |:-------------------------- |:------------ |
| [irc.hackint.dn42](irc://irc.hackint.dn42)        | 172.23.96.42, 172.22.2.235               | fd42:23:cda::6667 |



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

## Internal IRC
We also have an IRC network accessible only from within dn42.
It's accessible via `irc.dn42` on port `6697` with SSL, or on `6667` without ssl.
### Channels
The main channel on this network is `#dn42`.
### Control Panel
There's an Anope Web Control Panel available on `https://naco.irc.dn42:8080`. You can register there, or register on IRC and then use your credentials to log into the web panel. It allows you to perform almost any task you are able to do on IRC.

## Bouncer Offers

* stv0g: offering ZNC accounts (ask in IRC, [Webinterface](https://dev.0l.dn42/znc/))
