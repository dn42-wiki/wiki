# The idea
Tahoe-LAFS provides a distributed, reliable and crypted file system.

# How?
Some people runs Tahoe-LAFS nodes, providing space. With clients files can be published and received to the cloud. Everything will be encrypted on client side and keep redundant in the cloud.

# Benefit
Default you need only 3 of 10 parts of a file to reconstruct it. So a downtime of a tahoe node doesn't means data loss.

Because of the encryption an owner of a node don't know anything about the stored content.

## Usage
To provide storage to the cloud you have to run a node.

## Notwendige Software und Einstellungen
To run a node you have to install tahoe-lafs at least in version 1.10. You can get source code from https://tahoe-lafs.org/source/tahoe-lafs/releases/allmydata-tahoe-1.10.0.zip, if the version of the package in the distribution not at least 1.10. Then you have to extract it and install with `python2 setup.py build && sudo python2 setup.py install`.

Vor dem ersten Start wird ein Tahoe-LAFS-Knoten mit `bin/tahoe create-node` oder `mit bin/tahoe create-client` ein reiner Client (ohne eigenen Storage) erzeugt. Dabei wird ein `.tahoe`-Ordner im home-Verzeichnis angelegt. In dessen `tahoe.cfg`-Datei muss dann noch unter `introducer.furl` ein Link zu unserer Wolke eingefügt werden:

```
introducer.furl = pb://rzovwxeiykc6f6usyhqphpyn4nik5nzt@introducer.tahoe-lafs.services.ffhb:44411/introducer
```

Mit `bin/tahoe start` wird der lokale Knoten dann gestartet.

## Client
You can reach the local node via web browser at [http://localhost:3456](http://localhost:3456).

## Further informations
Look at https://tahoe-lafs.org for further informations.