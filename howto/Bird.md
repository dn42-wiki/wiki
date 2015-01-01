Bird is a commonly used BGP daemon.  This page provides configuration and help to run Bird for dn42.

# Simplest configuration possible

TODO

## Source address selection

Bird, like any other routing daemon, inserts routes into the kernel.  If you don't specify a source address on the kernel routes, then the kernel will choose for you (and will often choose an address in a transfer subnet, which is not what you want).

To specify the source address, you use the `krt_prefsrc` attribute, for instance in the export filter of the kernel protocol:

    protocol kernel {
       export filter {
               krt_prefsrc = 2001:db8:42::1;
               accept;
       };
    }

(this also works for IPv4)

# Example advanced configurations

Paste your own config template here.

## External links

http://danrimal.net/doku.php?id=wiki:bgp:bird:sample_configs2