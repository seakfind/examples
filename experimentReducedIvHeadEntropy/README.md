# experimentReducedIvHeadEntropy

According to GFW-Report and others' research, the GFW is likely to block a binary protocol unless it is an HTTP, TLS, or SSH connection, or the first six bytes of data sent from the client to the server can be interpreted as printable characters. This suggests that this kind of censorship can be temporarily evaded by sending only printable characters in the first six bytes of data. v2fly v2ray-core pull request #1552 therefore added the `experimentReducedIvHeadEntropy` option to Shadowsocks outbound. This option requests that V2Ray remap the first six bytes of the initialization vector to printable characters. Only the client configuration needs to be changed to implement this option.

Enabling experiments has security implications. It is possible for anyone on the privileged network path to identify the protocol when this experiment is enabled.
