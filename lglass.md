lglass is a Python software package designed for Internet Registries like the DN42. You can generate zone files for DNS and rDNS IPv4/v6, and handle the registry. It is available on GitHub as free software:

    $ git clone git://github.com/fritz0705/lglass.git

## Running your own Whois daemon

lglass provides an event-based whois daemon with internal caching, which was written in Python. It is very simple to run an instance:

    $ python -m lglass.whoisd -D $PATH_TO_DATA_DIR -H $HOST -P $PORT
