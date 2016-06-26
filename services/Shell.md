# Shell

Providers:

| Person        | Hostname                             | Net              | Description | Contact       |
|:------------- |:------------------------------------ |:---------------- |:----------- |:------------- |
| aix           | entropy.aix.ovh & entropy.aix.dn42   | clearnet & dn42  | See below   | aix @ hackint |
| mortzu        | shell.mortzu.dn42                    | dn42 only        | -           | -             |

## Entropy shellbox
The Entropy shellbox runs a [Grsecurity](https://grsecurity.net/) secured kernel, along with various other features such as some sysctl tweaks. It has an internal mail system which anyone can use to contact a shell user (`[user]@entropy.aix.ovh`). Mail is also accepted to shell users from external mail servers. Additionally, it has all of the [BlackArch tools](http://www.blackarch.org/tools.html) installed and available for everyone to use.
To further enhance security, the ownership of various SUID executables and logs has been restricted to members of certain groups, which are nicely explained by the [MOTD](https://entropy.aix.dn42/shell/motd) ([altlink](https://entropy.aix.ovh/shell/motd)).

By default, users will be part of only `tpe` and `audit` groups (as well as their own) but may request to be added to other groups. Please note that only `execve()` and `chdir()` calls are logged as a result of a user's membership of the `audit` group.

Lastly, in the interests of full disclosure, here are the [details of the box](http://pastie.org/pastes/10890047/text).