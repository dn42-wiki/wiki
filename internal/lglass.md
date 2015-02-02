lglass is a Python software package designed for Internet Registries like the DN42. You can generate zone files for DNS and rDNS IPv4/v6, and handle the registry. It is available on GitHub as free software:

    $ git clone git://github.com/fritz0705/lglass.git

[lglass Manual](http://lglass.flonet.dn42/)

## Running your own Whois daemon

lglass provides an event-based whois daemon with internal caching, which was written in Python. It is very simple to run an instance:

    $ ./bin/lglass-whoisd -H $HOST -P $PORT

.

    usage: lglass-whoisd [-h] [-4] [-6] [--host HOST] [--port PORT]
                                 [--cidr] [--no-cidr] [--inverse] [--no-inverse]

    optional arguments:
      -h, --help            show this help message and exit
      -4                    Listen on IPv4
      -6                    Listen on IPv6
      --host HOST, -H HOST  Listen on host
      --port PORT, -p PORT  Listen on port
      --cidr, -c            Perform CIDR matching on queries
      --no-cidr             Do not perform CIDR matching on queries
      --inverse, -i         Perform inverse matching on queries
      --no-inverse          Do not perform inverse matching on queries


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

## Configuration

The configuration file format is JSON and allows configuration of the database chain, the listen parameters, the custom messages and the process management.

| Option   |      Meaning      |
|----------|:-------------|
| listen.host |IP address for listening socket (Default: ::)|
|listen.port|TCP port for listening socket (Default: 4343)|
|listen.protocol|Protocol for listening socket (4 or 6, by default 6)|
|database|Array of database URLs to initialize database chain|
|database.types|Array of object types in database (Default: undefined)<br/>Default chain:<br/>[<br/>  "whois+lglass.database.file+file:.",<br/>  "whois+lglass.database.cidr+cidr:",<br/>  "whois+lglass.database.schema+schema:",<br/>  "whois+lglass.database.cache+cached:"<br/>]|
|messages.preamble|String preamble for whois responses|
|messages.help|String help message for help requests|
|process.user|User to change after initialization|
|process.group|Group to change after initialization|
|process.pidfile|Path to PID file|