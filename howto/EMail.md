Running email in dn42 is not very complicated.  Your SMTP daemon probably already listens on the wildcard address, so you mostly need to:

* open your firewall to allow TCP/25 from dn42
* setup DNS (MX records, or simply relevant A records)
* configure your mail server if needed

## Exim tips

### Sending emails

By default on Debian, Exim refuses to send mail to other mailservers when they resolve to RFC1918 addresses.  This will manifest by the following error message when trying to send a mail:

    ** foo@bar.dn42: all relevant MX records point to non-existent hosts

This is controlled by the `ignore_target_hosts` variable in the configuration file.

### Receiving emails

Don't forget to add your dn42 domains to the list of local domains, so that you accept incoming emails.  On Debian, it is controlled by `dc_other_hostnames` in `update-exim4.conf.conf`.  For instance:

    dc_other_hostnames='myself.org;myself.dn42;myserver.myself.dn42'


## Postfix

### Sending Mails
If your machine sends/receives Mails in "clearnet" with specific bound IP's you need to create an additional transport in master.cf

    out_dn42  unix -       -       n       -       -       smtp
        -o smtp_bind_address=172.23.67.1
        -o smtp_bind_address6=fd70:96c9:ef25::1
        -o smtp_helo_name=ns1.mhm.dn42
        -o syslog_name=postfix-dn42

and add this transport to /etc/postfix/transport for dn42 (and dont forget to postmap)

    .dn42           out_dn42:

This should to the trick for sending mails via your DN42-IP

### Receiving emails

The Domain mails should be received for has to be added to 'mydestination =' in main.cf