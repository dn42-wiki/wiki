# GeoFeed

A **GeoFeed** is a standardized mechanism for publishing geographic location information about IP address prefixes, helping associate IP blocks with real-world locations for better geolocation accuracy, latency mapping, and network topology documentation.

## Standards

| RFC | Description |
|-----|-------------|
| [RFC 8805](https://www.rfc-editor.org/rfc/rfc8805) | Defines the CSV-based GeoFeed file format |
| [RFC 9632](https://www.rfc-editor.org/rfc/rfc9632) | Specifies referencing GeoFeeds from RPSL objects via `geofeed:` attribute |
| [RFC 9877](https://www.rfc-editor.org/rfc/rfc9877) | RDAP extension for automated GeoFeed discovery |

## File Format

A UTF-8 CSV file with the following fields:

```csv
# prefix,country_code,region_code,city,postal
172.20.0.0/24,FR,FR-ARA,Lyon,69123
fd42:1234::/48,FR,,,
```

Fields follow [ISO 3166](https://en.wikipedia.org/wiki/ISO_3166) conventions. Unused fields may be left empty.

## Publishing on DN42

1. **Host the file** — Serve your GeoFeed over HTTP/HTTPS. **Hosting within DN42 is strongly recommended** so that the file remains accessible to all DN42 participants regardless of public internet connectivity, and to keep DN42 infrastructure self-contained.
2. **Reference in the registry** — Add a `geofeed:` attribute to your `inetnum` or `inet6num` object:

```rpsl
inetnum:    172.20.0.0/24
geofeed:    http://example.dn42/geofeed.csv
```

> ⚠️ Hosting your GeoFeed on the public internet means it may be unreachable to peers who access DN42 exclusively through the internal network. Always prefer a DN42-reachable URL.

## Best Practices

- Ensure the URL remains reachable and update the registry if it changes
- Maintain consistent node names and clear comments for easier automation and identification
- Keep locations accurate
- Store your GeoFeed in version control (e.g., Git) for easier management

## Further Reading

- [RIPE 82 GeoFeed presentation](https://ripe82.ripe.net/presentations/84-RIPE82_geofeed.pdf)