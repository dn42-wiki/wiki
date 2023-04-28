# How Authentication Works

`auth` attributes within registry `mntner` objects define a public key that is used to verify the identity of the maintainer and prove that changes to registry objects are authorised.

When a pull request is submitted to the registry, the submitter signs the git commit hash with their private key. The registry maintainers will then check the signature against the registered public key to authorise the change. 

The signature and verification process varies depending on the type of public key within the `auth` attribute.

## Preferred signing methods

*Tip: Add your GPG or SSH key to your account in gitea, doing so enables gitea to automatically check your signature when you sign the commit*

### When using a GPG/PGP Key
1. **Sign the commit using git** - this is the best option as the signature is recorded directly in the git log 
2. Sign using `gpg --clearsign` and provide the signature in the PR comments - only do this if you absolutely cannot sign the commit in git

### When using an SSH key
1. **Sign the commit using git** - git >= 2.34.0 can now sign commits using ssh keys, this is the best option if you able to do so
2. Use the `sign-my-commit` script in the registry - the script adds your signature in a format that allows for automated checking
3. Manually provide a signature in the PR comments using one of the methods detailed below - only do this if you can't sign using git or with the included script

The sections below provide detailed instructions for each of the auth methods.

---

## Finding the commit hash

`git log` will list all the recent commits and show the commit hash:
```
commit 6e2e9ac540e2e4e3c3a135ad90c8575bb8fa1784 (HEAD -> master)
Author: foo <foo@baz.com>
Date:   Mon Jan 01 01:01:01 2020 +0000

    Change some stuff
```

In this case the full commit hash is `6e2e9ac540e2e4e3c3a135ad90c8575bb8fa1784`

---

## Authentication using a GPG/PGP Key

To verify your key, the registry maintainers need to be able to find your full public key.  
There are three options for doing this. but you only need to do **one** of these:  

 1. **Add your public key to your account in gitea** - this is the best option as gitea will automatically check your signature
 2. Upload your key to a public key server 
 3. Create a `key-cert` object in the registry containing your public key

### `auth` attribute format, when your public key is in gitea or a public keyserver

- Use the following `auth` attribute in your `mntner` object:
```conf
auth:               pgp-fingerprint <fingerprint>
```
Where `<fingerprint>` is your **full 40-digit** key fingerprint, without spaces.

- Ensure that your public key has been uploaded to your account in gitea or a public keyserver, e.g. [SKS](https://sks-keyservers.net/), [OpenPGP](https://keys.openpgp.org/), [keybase](https://keybase.io/).

### `auth` attribute format when creating a `key-cert` object

*Tip: look at the existing key-cert objects for examples of how to add your public key*

- In this case the `auth` attribute must refer to the new key-cert object so use the following in your `mntner` object:
```conf
auth:               PGPKEY-<short fingerprint>
```
Where `<short fingerprint>` is the last **8** digits from your key fingerprint.

- Create a `key-cert` object for your public key, using `PGPKEY-<fprint>` for the filename.

### How to sign your commit

*Tip: There are many public guides available with step by step instructions on how to sign commits, e.g. [github guide](https://help.github.com/en/github/authenticating-to-github/signing-commits)*

- Use `git commit -S` to commit and sign your change.

- If you have already committed your change without signing it, you can sign the existing commit using:
```sh
git commit --amend --no-edit -S
```
If you had already pushed your change to gitea, you must also do a force push (`git push --force`) to update the remote copy.

### Verifying the signature

- Use `git log --show-signature` to show recent commits and signatures
- If you have uploaded your key to gitea, you can also check in the gitea UI that your commit is signed and has been verified successfully 

---

## Authentication using an SSH key

Older versions of git and ssh don't support generic ssh signing so there are multiple ways of providing ssh signatures based on the versions you have and the type of key you are using. The newer signature methods are preferred as they allow automatic verification of your signature.

In preference order:

1. **Sign using git**
2. Sign using the included `sign-my-commit` script

If you cannot get the above to work you may also: 

3. Manually sign using the generic ssh-keygen method
4. Manual sign using specific methods for rsa or ecdsa

### `auth` attribute format when using an ssh key

The generic format for authentication using an SSH key is as follows:
```conf
auth:               ssh-<keytype> <pubkey>
```

Common examples:

```conf
auth:               ssh-ed25519 <pubkey>
```

```conf
auth:               ssh-rsa <pubkey>
```

*Tip: look in the existing registry objects for examples*

### Signing using git

If you have git >= 2.34.0 you can now sign git commits directly with an SSH key.

Brief instructions are below, however there are also more detailed guides available on how to do this, e.g. [here](https://blog.dbrgn.ch/2021/11/16/git-ssh-signatures/) or [here](https://calebhearth.com/sign-git-with-ssh)

#### Configuration

- Set your git signature format to be SSH

```sh
git config --global gpg.format ssh
```

- Tell git which SSH key to use

```sh
git config --global user.signingKey '<ssh public key>'
```

#### How to sign

Once configured, you can now use git to sign your commit as normal:

- Use `git commit -S` to commit and sign your change.

- If you have already committed your change without signing it, you can sign the existing commit using:
```sh
git commit --amend --no-edit -S
```
If you had already pushed your change to gitea, you must also do a force push (`git push --force`) to update the remote copy.

#### Verifying the signature

Verifying an ssh signature is slightly more complicated than with gpg keys, please see the guides linked above.

The easiest way to verify your signature is to ensure your SSH key is uploaded then push your changes to gitea. Gitea will automatically verify your signature for you and show if it was successful in the UI. 

### Sign using the `sign-my-commit` script

The registry includes a script that uses ssh-keygen signatures to sign your changes in a format that allows for automatic verification. It requires ssh-keygen >= v8. 

*Tip: use `./sign-my-commit --help` to see all options*

#### How to sign

```sh
./sign-my-commit --ssh --key <path to your SSH private key> --push <MNTNER>
```

e.g.

```sh
./sign-my-commit --ssh --key /home/foo/.ssh/id_ed25519 --push FOO-MNT
```

#### Verifying the signature

The script can also verify your signature:

```sh
./sign-my-commit --ssh --verify <MNTNER>
```

### Manually signing using ssh-keygen

*Please only manually sign if you cannot use either of the automated methods*

OpenSSH v8 introduced new functionality for creating signatures using SSH keys. If you have an older version, you can compile the latest version of ssh-keygen from the [openssh-portable repo](https://github.com/openssh/openssh-portable).

#### How to sign

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

---

### Manually signing - SSH legacy methods

**Only use the methods below if you cannot use any of the previous signature methods**

Please try and upgrade your ssh-keygen version and use the generic ssh-keygen method first.

---

### Authentication with an SSH RSA key

- Use the following `auth` attribute in your `mntner` object:
```conf
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

### Authentication with an SSH ecdsa key

- Use the following `auth` attribute in your `mntner` object:
```conf
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
