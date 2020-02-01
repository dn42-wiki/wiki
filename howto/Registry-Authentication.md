# How Authentication Works

`auth` attributes within registry `mntner` objects define a public key that is used to verify the identity of the maintainer and prove that changes to registry objects are authorised.

When a pull request is submitted to the registry, the submitter signs the git commit hash with their private key. The registry maintainers will then check the signature against the registered public key to authorise the change. 

The signature and verification process varies depending on the type of public key within the `auth` attribute. 

#### Finding the commit hash

`git log` will list all the recent commits and show the commit hash:
```
commit 6e2e9ac540e2e4e3c3a135ad90c8575bb8fa1784 (HEAD -> master)
Author: foo <foo@baz.com>
Date:   Mon Jan 01 01:01:01 2020 +0000

    Change some stuff
```

## Authentication with PGP Key

PGP keys may be uploaded to a public keyserver for verification, or added in the registry.

#### Using a public keyserver

- Use the following `auth` attribute in your `mntner` object:
```
auth:               pgp-fingerprint <fingerprint>
```
Where `<fingerprint>` is your full 40-digit key fingerprint, without spaces.

- Ensure that your public key has been uploaded to a public keyserver, e.g. [SKS](https://sks-keyservers.net/), [OpenPGP](https://keys.openpgp.org/), [keybase](https://keybase.io/).

#### Adding to the registry

- Use the following `auth` attribute in your `mntner` object:
```
auth:               PGPKEY-<fprint>
```
Where `<fprint>` is the last 8 digits from your key fingerprint.

- Create a `key-cert` object for your public key, using `PGPKEY-<fprint>` for the filename. Do browse the registry and check the existing objects for examples.

#### Signing your commits

- Use `git commit -S` to commit and sign your change. See the [github guide](https://help.github.com/en/github/authenticating-to-github/signing-commits).

- If you have already committed your change, you can sign it using.
```
git commit --amend --no-edit -S
```

#### Verifying the signature

- Use `git log --show-signature` to show recent commits and signatures.

## Authentication using an SSH key

The generic format for authentication using an SSH key is as follows:
```
auth:               ssh-<keytype> <pubkey>
```
There are examples below for each specific key type.

#### Generic process for signing with an SSH key

OpenSSH v8 introduced new functionality for creating signatures using SSH keys. If you have an older version, you can compile the latest version of ssh-keygen from the [openssh-portable repo](https://github.com/openssh/openssh-portable).

Use the following to sign the latest `<commit hash>` (that you found using `git log`)
```sh
echo "<commit hash>" | ssh-keygen -Y sign -f <private key file> -n dn42
```

Post the signature in to the 'Conversation' section of your pull request to allow the registry maintainers to verify it. It can help to also include the commit hash that you have signed, to avoid any confusion. 

#### Verifying the signature

The following procedure will verify the signature (using the `<commit hash>`, your `<pubkey>` and the `<signature>` generated in the previous step. 

Create a temporary file containing the signature
```sh
echo "<signature>" > sig.tmp
```
Create a temporary 'allowed users' file
```sh
echo "YOU-MNT ssh-<keytype> <pubkey>" > allowed.tmp
```
Verify the signature
```sh
echo "<commit hash>" | \
    ssh-keygen -Y verify -f allowed.tmp -n dn42 -I YOU-MNT -s sig.tmp
```

### Authentication with an SSH RSA key

- Use the following `auth` attribute in your `mntner` object:
```
auth:               ssh-rsa <pubkey>
```
Where `<pubkey>` is the ssh public key copied from your id_rsa.pub file. 

#### Signing your commits

If you cannot use the generic SSH process described above then RSA signatures can also be created using openssl.

Use the following to sign your `<commit hash>` (that you found using `git log`)
```sh
openssl pkeyutl \
   -sign \
   -inkey ~/.ssh/id_rsa \
   -in <(echo "<commit hash>") | base64
```

Post the signature in to the 'Conversation' section of your pull request to allow the registry maintainers to verify it. It can help to also include the commit hash that you have signed, to avoid any confusion. 

#### Verifying the signature

The following script will verify the signature (using the `<commit hash>`, your rsa `<pubkey>` and the `<signature>` generated in the previous step. 
```sh
openssl pkeyutl \
   -verify \
   -pubin \
   -in <(echo "<commit hash>") \
   -inkey <(ssh-keygen \
               -e \
               -m PKCS8 \
               -f <(echo "ssh-rsa <pubkey>")\
            ) \
   -sigfile <(echo "<signature>" | base64 -d)
```

### Authentication with an SSH ed25519 key

- Use the following `auth` attribute in your `mntner` object:
```
auth:               ssh-ed25519 <pubkey>
```
Where `<pubkey>` is the ssh public key copied from your id_ed25519.pub file. 

#### Signing your commits

There is no alternative process for signing using ed25519 keys, you must use the generic process described above. The process only works with ssh-keygen versions >= v8. 

Use the following to sign your `<commit hash>` (that you found using `git log`)
```sh
echo "<commit hash>" | ssh-keygen -Y sign -f ~/.ssh/id_ed25519 -n dn42
```

Post the signature in to the 'Conversation' section of your pull request to allow the registry maintainers to verify it. It can help to also include the commit hash that you have signed, to avoid any confusion. 

#### Verifying the signature

The following procedure will verify the signature (using the `<commit hash>`, your ed25519 `<pubkey>` and the `<signature>` generated in the previous step. 

Create a temporary file containing the signature
```sh
echo "<signature>" > sig.tmp
```
Create a temporary 'allowed users' file
```sh
echo "YOU-MNT ssh-ed25519 <pubkey>" > allowed.tmp
```
Verify the signature
```sh
echo "<commit hash>" | \
    ssh-keygen -Y verify -f allowed.tmp -n dn42 -I YOU-MNT -s sig.tmp
```

### Authentication with an SSH ecdsa key

- Use the following `auth` attribute in your `mntner` object:
```
auth:               ecdsa-sha2-nistp256 <pubkey>
```
Where `<pubkey>` is the ssh public key copied from your id_ecdsa.pub file. 

#### Signing your commits

If you cannot use the generic SSH process described above then ecdsa signatures can also be created using openssl.

**DO NOT do this on your original ssh key.**  
Make a copy and use the copy as the ssh-keygen command below will overwrite the key file given.

Convert your private ssh key to a file that openssl can read:  
**DO THIS ON A COPY OF YOUR SSH KEY**
```sh
ssh-keygen -p -m pem -f <private key file copy>
```

Sign the commit hash using your ecdsa key, using openssl:
```sh
openssl pkeyutl -sign \
    -inkey <converted key file> \
    -in <(echo "<commit hash>") | base64
```

Post the signature in to the 'Conversation' section of your pull request to allow the registry maintainers to verify it. It can help to also include the commit hash that you have signed, to avoid any confusion. 

#### Verifying the signature

The following script will verify the signature (using the `<commit hash>`, your ecdsa `<pubkey>` and the `<signature>` generated in the previous step. 
```sh
openssl pkeyutl \
   -verify \
   -pubin \
   -in <(echo "<commit hash>") \
   -inkey <(ssh-keygen \
               -e \
               -m PKCS8 \
               -f <(echo "ecdsa-sha2-nistp256 <pubkey>")\
            ) \
   -sigfile <(echo "<signature>" | base64 -d)  
```
