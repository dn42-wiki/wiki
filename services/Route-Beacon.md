There is a route beacon periodically advertising and withdrawing the prefixes 172.21.100.24/29 and fd40:e3b7:1d77:1234::/64. These are the only prefixes of as4242421933.
The schedule is the following: the prefixes are announced in every hour in between minutes 15-45, and withdrawn when the clock is outside of the above mentioned period.
The main purpose of the whole experiment is to be able to spot the ghosting implementations.
The current state could be monitored on the prometheus or nrpe endpoint at test.nop.dn42 or one can query the last recorded ghosting state by telnetting the same endpoint and issuing commands from the banner.
