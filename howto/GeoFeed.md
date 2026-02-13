A *GeoFeed* is a standardized mechanism for publishing geographic location information about IP address prefixes. It helps network operators associate IP address blocks with real-world locations, enabling better visualization, monitoring, and geolocation accuracy.

A GeoFeed is especially useful in distributed networks like DN42, where many nodes exist in different physical locations. It can assist in mapping latency, debugging routing, and documenting physical network topology.

## Standards & Protocol References

**[RFC 8805](https://www.rfc-editor.org/rfc/rfc8805) - _Format for Self-Published IP Geolocation Feeds_**  
- Defines the **format** for GeoFeed files (CSV);
- Geographic mappings associate prefixes with location attributes.
- A GeoFeed file consists of comma-separated values in UTF-8 format.

**[RFC 9632](https://www.rfc-editor.org/rfc/rfc9632) - _Finding and Using GeoFeed Data_**  
- Specifies how to reference GeoFeed files from routing databases such as RPSL objects (`inetnum`, `inet6num`);
- Defines how to augment existing routing registry objects with attributes pointing to GeoFeed URLs (e.g., using a `geofeed:` attribute);
- Replaces earlier RFC 9092, improving schema and specification details.

**[RFC 9877](https://www.rfc-editor.org/rfc/rfc9877) - _RDAP GeoFeed Extension_**  
- Defines an extension to RDAP (Registration Data Access Protocol) that allows RDAP servers to disclose GeoFeed URLs for IP address resources;
- Useful for automated discovery of GeoFeed data through registry queries.

### Operational Example (RIPE 82)
A concrete operational example of GeoFeed publication and integration within the RIPE Database was presented at RIPE 82 (see presentation below):  
https://ripe82.ripe.net/presentations/84-RIPE82_geofeed.pdf

## GeoFeed file formats

**CSV Example**:
```csv
# prefix,country_code,region_code,city,postal
172.20.0.0/24,FR,FR-ARA,LYON,69123
fd42:1234::/48,FR,,,,
...
```

**JSON Example**:  
```json
[
    {
        "prefix": "172.20.0.0/24",
        "country_code": "FR",
        "region_code": "FR-ARA",
        "city": "LYON",
        "postal": 69123
    },
    {
        "prefix": "fd42:1234::/48",
        "country_code": "",
        "region_code": "",
        "city": "",
        "postal": 0
    },
    ...
]
```

> GeoFeed files can also be TXT or other simple text formats, but CSV/JSON is recommended for ease of parsing in scripts and automation tools.

## How to publish a GeoFeed on DN42 ?

1. **Host your GeoFeed file**  
    The file must be accessible via HTTP or HTTPS (IPv4 and/or IPv6), either on DN42 or on clearnet.  
2. **Reference in the Registry**  
    Add a `remarks: GeoFeed <url>` attribute in your `inet(6)num` object to point to your hosted GeoFeed.  
    Example DN42 WHOIS snippet:
    ```rpsl
    inetnum: 0.0.0.0/0
    remarks: GeoFeed http://example.dn42/geofeed.csv
    ```

## Best Practices
- **Version control**: Store your GeoFeed in Git to track changes and manage updates.
- **Accuracy**: Keep node locations precise enough for mapping, but avoid exposing sensitive private addresses or exact locations.
- **Accessibility**: Ensure your GeoFeed URL is always reachable; update registry references if the file moves.
- **Integration**: Combine GeoFeed with monitoring dashboards, RIPE Atlas probes (clearnet), or Looking Glass tools to visualize node availability over time.
- **Naming Conventions**: Maintain consistent node names and clear comments for easier automation and identification.