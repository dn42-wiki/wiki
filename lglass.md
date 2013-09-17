lglass is a Python software package designed for Internet Registries like the DN42. You can generate zone files for DNS and rDNS IPv4/v6, and handle the registry. It is available on GitHub as free software:

    $ git clone git://github.com/fritz0705/lglass.git

## Running your own Whois daemon

lglass provides an event-based whois daemon with internal caching, which was written in Python. It is very simple to run an instance:

    $ ./bin/lglass-whoisd -D $PATH_TO_DATA_DIR -H $HOST -P $PORT

## Generate zone files

lglass also provides a script to generate zone files from the registry. It's named zonegen.py and requires a registry dump from Monotone.

To generate DNS zones:

    $ ./bin/lglass-zonegen -d $PATH_TO_DATA_DIR -n ns1... -n ns2... -e foo.bar.com dns -z dn42

To generate IPv4 rDNS zones:

    $ ./bin/lglass-zonegen -d $PATH_TO_DATA_DIR -n ns1... -n ns2... -e foo.bar.com rdns4 -N 172.22.0.0/16

To generate IPv6 rDNS zones:

    $ ./bin/lglass-zonegen -d $PATH_TO_DATA_DIR -n ns1... -n ns2... -e foo.bar.com rdns6 -N fd00::/8

## Reformat RPSL files

You can also reformat RPSL files using lglass by using the lglass.rpsl module:

    $ ./bin/lglass-rpsl < $DATA/inetnum/172.22.0.53_32

lglass.rpsl also supports in-place operation:

    $ ./bin/lglass-rpsl -i $DATA/inetnum/172.22.0.53_32

This opens the file, reads the content into memory, seeks to position 0, writes the formatted object and truncates the file.
Simple web interface

lglass also comes with a simple web interface written in Python3 using Bottle and Jinja2. It also provides a binary to run it using wsgiref:

    $ ./bin/lglass-web

Furthermore you can use any WSGI server like Gunicorn by using lglass.web.application:app as WSGI callback. You can provide a path to the configuration file in the environment variable `LGLASS_WEB_CFG`.
