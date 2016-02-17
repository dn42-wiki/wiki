Running email in dn42 is not very complicated.  Your SMTP daemon probably already listens on the wildcard address, so you mostly need to:

* open your firewall to allow TCP/25 from dn42
* setup DNS (MX records, or simply relevant A records)
* configure your mail server if needed

## Redirect

There are forwarding rules for _PERSON_ @ dn42.org to the mail addresses which hav been given in the registry. Please note that the trailing `-DN42` is stripped from the local part.

## Test email

Send an email to `test@evenet.dn42` to check if your mail setup is correct. This host will reply using the following
sieve filter:

```
require ["regex", "variables", "vacation-seconds"];
if header :contains "To" ["test@evenet.dn42"] {
  if header :matches "Subject" "*" {
      set "subject_was" ": ${1}";
  }
  vacation :addresses ["test@evenet.dn42"] :seconds 60 :subject  "Re: ${subject_was}" "Your dn42 email setup works!";
}
```

####Example####

| Handle       | Alias          | Redirection           |
|:------------ |:-------------- |:--------------------- |
| `STV0G-DN42` | stv0g@dn42.org | post@steffenvogel.de` |

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

If you use `smtpd_recipient_restrictions` you can use the following rule to white-list dn42 as sender.
This can circumvent certain rdns configuration failure or in case you use rbl lists:

```
smtpd_recipient_restrictions = permit_mynetworks,
                         permit_sasl_authenticated,
                         check_client_access cidr:/etc/postfix/dn42.cidr,
                         reject_non_fqdn_sender,
                         # ...
                         permit
```

```
#/etc/postfix/dn42.cidr
172.16.0.0/12 OK
10.0.0.0/8 OK
fc00::/7 OK
```

```
$ postmap /etc/postfix/dn42.cidr
```


### Receiving emails

The Domain mails should be received for has to be added to `mydestination =` in main.cf

## New SMTP RFC SMTPUTF8
### EAI
Email Address Internationalization (EAI) as defined in [RFC 6531](http://tools.ietf.org/html/rfc6531) (SMTPUTF8 extension), [RFC 6532](http://tools.ietf.org/html/rfc6532) (Internationalized email headers) and [RFC 6533](http://tools.ietf.org/html/rfc6533) (Internationalized delivery status notifications).
### Postfix
Introduced with Postfix version 3.0, this fully supports UTF-8 email addresses and UTF-8 message header values.
more at the [SMTPUTF8_README](http://www.postfix.org/SMTPUTF8_README.html).
### Exim
Watch Exims EAI Tracker [Bug 1177](http://bugs.exim.org/show_bug.cgi?id=1177)