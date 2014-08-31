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
