# Tahoe LAFS

## The idea
Tahoe-LAFS provides a distributed, reliable and crypted file system.

## How?
Some people run Tahoe-LAFS nodes, providing space. With clients files can be published and received to the cloud. Everything will be encrypted on client side and keep redundant in the cloud.

## Benefit
Default you need only 3 of 10 parts of a file to reconstruct it. So a downtime of a tahoe node doesn't means data loss.

Because of the encryption an owner of a node don't know anything about the stored content.

## Usage
To provide storage to the cloud you have to run a node.

## Install and configuration
To run a node you have to install tahoe-lafs at least in version 1.10.2. You can get source code from <https://tahoe-lafs.org/source/tahoe-lafs/releases/allmydata-tahoe-1.10.2.zip>, if the version of the package in the distribution not at least 1.10. Then you have to extract it and install with `python2 setup.py build && sudo python2 setup.py install`.

Before the first start you have to create a node with `bin/tahoe create-node` or a client (doesn't provide storage) with `bin/tahoe create-client`. This will create the folder .tahoe in your home dir. In the file .tahoe/tahoe.cfg you have to enter on `introducer.furl` the link to our introducer node (UPDATED):

```
introducer.furl = pb://shvdnad4bqey27ff7ngtschexamvdmmr@tahoe-lafs.e-utp.dn42:44412/kmvmrcforeeet7isgq7ftuymywqp3obb
helper.furl = pb://ru7miwm74bfkd6ytchfoq4wgvo3vikq3@fido.e-utp.dn42:44412/iiiopiclr2gszw2fmckbx3eob6krxk7x
```

With `bin/tahoe start` you start your local node.

## Client
You can reach the local node via web browser at [http://localhost:3456](http://localhost:3456).

## Further informations
Look at <https://tahoe-lafs.org> for further information.
