nlgld.dn42 hosts a PGP keyserver (Hockeypuck) on keys.nlgld.dn42.
Sending and receiving keys works exactly as other keyservers would.

Note that emails associated with PGP keys are not verified. Do not use this keyserver as the only source of trust. Instead, refer to the registry, or find another way to verify the person behind the key.


## Sending a PGP key using GPG

```sh
gpg --keyserver keys.nlgld.dn42 --send-keys XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

## Receiving a PGP key using GPG

```sh
gpg --keyserver keys.nlgld.dn42 --recv-keys XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

## Use keys.nlgld.dn42 as a default GPG keyserver

Append the following to `~/.gnupg/gpg.conf`:

```
keyserver keys.nlgld.dn42
```

In the gnupg config file, the keyservers listed are queried in order of appearance. Please place this keyserver low on that list, to decrease the load on it.

## Contact

Please contact `NLGLD-DN42` in case of abuse.