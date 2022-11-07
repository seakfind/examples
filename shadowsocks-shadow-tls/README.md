# Shadowsocks + Shadow-TLS

Shadow-TLS presents to the outside observer a real TLS handshake with a trusted site. This obscures the fact that data for the true destination is being transferred behind the scenes. The concept is illustrated in a diagram in the [docs](https://github.com/ihciah/shadow-tls/blob/master/docs/protocol-en.md), _quod vide_. Shadow-TLS is available for macOS and Linux but not for Windows. 

In the scenario in this example:

* The Shadowsocks client, running on an Ubuntu PC, listens on localhost port `10808`. SS forwards encrypted traffic to localhost port `8001`.
* The Shadow-TLS client is listening on localhost port `8001`. It talks to Shadow-TLS on the server on port `443`. Their communication appears to include a real TLS handshake with the trusted site.
* The Shadow-TLS server listens on port `443`. It relays the TLS handshake with the trusted site. Behind the scense, it forwards the true data to SS on port `8002` on the same server.
* The SS server listens on server internal port `8002`. SS sends out the traffic to its final destination.
