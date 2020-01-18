#Application programming interfaces (APIs)
This page can be useful if you are trying to automate something or if you are trying to retrieve data programmatically.

##Proving ASN ownership
Through this automated service you prove your ASN ownership to KIOUBIT-MNT who then automatically creates a "ownership verification signature". This signature can be very easily verified by anyone. This removes the hassle from checking every different authentication method in the registry. This is particularly useful for automated setups.

API Documentation: https://dn42.g-load.eu/api/verify/documentation.txt

##Registry REST API

[dn42regsrv](https://git.dn42.us/burble/dn42regsrv) is a REST API for the DN42 registry that provides a bridge between interactive applications and the registry.

As well as the main REST API to the DN42 registry, the server can also generate ROA tables and provides a small web application for exploring registry data.

A public instance of the API and associated explorer web app is available at the following URLs:

https://explorer.burble.com/ (public internet link)  
http://explorer.collector.dn42/ (DN42 link)
