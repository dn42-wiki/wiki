# DN42 RPKI 
This page covers guidance and examples on using RPKI within DN42.

## Quick Start

It is recommended to run your own RPKI validator, as this provides you with the most security and control over your routing decisions. However, to get started, or if running your own validator isn’t desirable, a public RPKI RTR server is available. The service supports full RPKI validation for all relevant DN42 and affiliated networks’ prefixes.

### Using Public RPKI Services

DN42’s RPKI RTR service endpoints are hosted by multiple operators. By configuring multiple RTR servers in your BGP daemon, you gain additional resiliency and improved validation coverage.

| Server                   | Port | **IPv4/IPv6** |
| ------------------------ | ---- | ------------- |
| rpki.akae.re             | 8082 | both          |
| rpki.dn42.launchpadx.top | 8082 | both          |
| rpki.dn42.milu.moe       | 8082 | both          |
| rpki.dn42.6700.cc        | 8282 | both          |
| rpki.nia.dn42            | 8082 | both          |

#### FlapAlerted RPKI RTR Servers

These services will publish a ROA pointing to AS0 when a prefix flapping. This can be used to prevent flap from spreading further in the network.

| Server                   | Port | **IPv4/IPv6** | FlapAlerted Instance | Provider |
| ------------------------ | ---- | ------------- | -------------------- | -------- |
| rpki.dn42.launchpadx.top | 8084 | both          | [https://flaps.lpnet0.dn42/](https://flaps.lpnet0.dn42/), [https://dn42-flaps.launchpadx.top/](https://dn42-flaps.launchpadx.top/) | AS4242423702 |
| rpki.nia.dn42            | 8084 | both          | [flap.nia.dn42](https://flap.nia.dn42/), [flap42.strexp.net](https://flap42.strexp.net/) | AS4242421331 |
| rpki.nia.dn42            | 8083 | both          | Multiple Sources (2-Votes Policy) (see [flap-data.nia.dn42](https://flap-data.nia.dn42/)) | AS4242421331 |

To configure the service, connect your BGP software’s RPKI client to one or more of these RTR servers.

#### Example Configuration (Bird 2)

```conf
protocol rpki roa_dn42_1 {
        roa4 { table dn42_roa; };
        roa6 { table dn42_roa_v6; };
        remote "rpki1.example.com";
        port 8082;
        refresh 600;
        retry 300;
        expire 7200;
}

protocol rpki roa_dn42_2 {
        roa4 { table dn42_roa; };
        roa6 { table dn42_roa_v6; };
        remote "rpki2.example.com";
        port 8082;
        refresh 600;
        retry 300;
        expire 7200;
}
```

### Running Your Own RPKI Server

#### With Docker

```bash
docker run --name dn42rpki -p 8082:8282 --restart=always -d rpki/stayrtr -verify=false -checktime=false -cache=https://dn42.burble.com/roa/dn42_roa_46.json
```

#### With Docker Compose

```conf
services:
  stayrtr:
    image: rpki/stayrtr:latest
    ports:
      - "8082:8282"
    command: >
      -cache https://dn42.burble.com/roa/dn42_roa_46.json
```

### Using Kioubit's DN42 Registry Wizard

[DN42 Registry Wizard](https://github.com/Kioubit/dn42_registry_wizard) is a comprehensive tool for DN42 registry interactions. **Unlike other solutions, it can parse the registry and host an RTR server all-in-one** without requiring separate components.

#### All-in-One RTR Server

```sh
# Clone the DN42 registry
git clone https://git.dn42.dev/dn42/registry.git

# Start RTR server directly from registry
./registry_wizard <path to registry> rtr 

# Setup a cronjob to continously update the registry and notify registry_wizard
git fetch --all
git reset --hard origin/master
kill -SIGUSR1 "$(pidof 'registry_wizard')"
```

```
Usage: registry_wizard <registry_root> rtr [OPTIONS]

Options:
  -p, --port <port>        Port to listen on [default: 9323]
      --refresh <refresh>  RTR refresh timing [default: 3600]
      --expire <expire>    RTR expire timing [default: 7200]
      --retry <retry>      RTR retry timing [default: 600]
  -h, --help               Print help
```