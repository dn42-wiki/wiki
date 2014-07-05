Quagga is probably one of the oldest software router around.  It still works, of course, even though it has an unattractive configuration syntax (unfortunately inspired by Cisco's IOS) and has some small issues with IPv6.  But since it's so old, you will find a lot of configuration examples around.

## Source address selection

Use this in your `zebra.conf`:

    route-map RM_SET_SRC permit 10
      set src 172.22.XX.XX
    ip protocol bgp route-map RM_SET_SRC

Unfortunately, this is not possible with IPv6...