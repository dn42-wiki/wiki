# DN42 RPKI 
This page covers guidance and examples on using RPKI within DN42.

## Quick Start

It is recommended to run your own RPKI validator, as this provides you with the most security and control over your routing decisions. However, to get started, or if running your own validator isn’t desirable, a public RPKI RTR server is available. The service supports full RPKI validation for all relevant DN42 and affiliated networks’ prefixes.

### Using Public RPKI Services

DN42’s RPKI RTR service endpoints are hosted by multiple operators. By configuring multiple RTR servers in your BGP daemon, you gain additional resiliency and improved validation coverage.

| Server             | Port | **IPv4/IPv6** |
| ------------------ | ---- | ------------- |
| rpki.dn42.milu.moe | 8082 | both          |
| rpki.akae.re       | 8082 | both          |

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
      - "8082:8082"
    command: >
      -bind :8082
      -cache https://dn42.burble.com/roa/dn42_roa_46.json
    stdin_open: true
    tty: true
```

