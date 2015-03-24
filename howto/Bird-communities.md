Bird is a commonly used BGP daemon.  This page provides configuration and help to run Bird for dn42.

Communities can be used to prioritize traffic based on different flags, in DN42 we are using communities to display latency, bandwidth and encryption. Please note that everyone should be using community 64511.

# BGP community criteria
```
(64511, 1) :: latency \in [0, 2.7ms]
(64511, 2) :: latency \in [2.7ms, 7.3ms]
(64511, 3) :: latency \in [7.3ms, 20ms]
(64511, x) :: latency \in [exp(x-1), exp(x)] ms (for x < 10)
 
(64511, 21) :: bw >= 0.1mbit
(64511, 22) :: bw >= 1mbit
(64511, 23) :: bw >= 10mbit
(64511, 24) :: bw >= 100mbit
(64511, 25) :: bw >= 1000mbit
(64511, 2x) :: bw >= 10^(x-2) mbit
bw = min(up,down) for asymmetric connections
 
(64511, 31) :: not encrypted
(64511, 32) :: encrypted with unsafe vpn solution
(64511, 33) :: encrypted with safe vpn solution (but no PFS)
(64511, 34) :: encrypted with safe vpn solution with PFS 
```

https://gist.github.com/tombii/b4edb9ee6a4f59617d98