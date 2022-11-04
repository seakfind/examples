# VLESS-TCP-XTLS-Vision

Forked from https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS-Vision

## Domain

Get a paid domain name. A cheap source is [Namesilo](https://www.namesilo.com).

## SSL cert

Open port `80/tcp` in your server firewall.

Install the packages for a free Let's Encrypt SSL certificate:

```
apt install certbot -y
```

Request the certificate and key, replacing `vps.example.com` in the example by your actual server hostname:

```
certbot certonly --standalone --agree-tos --register-unsafely-without-email -d vps.example.com
```

Set everything up for renewal:

```
certbot renew --dry-run
```

## Xray server

We will request the latest version (`1.6.2` at this time) so that we get the most recent prerelease features. We want to run as `root` to access the SSL certificate private key.

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root --version 1.6.2
```

Generate a secret UUID:

```
uuidgen
```

Example of results: `b709e1e9-6f08-45b1-b8e5-b5814539f0cd`.

Generate a random port number:

```
echo $(($RANDOM + 10000))
```

Example of results: `12345`.

Open port `12345/tcp` in your server firewall.

Edit the Xray server configuration file:

```
vi /usr/local/etc/xray/config.json
```

Delete the dummy line and insert a configuration similar to the example **`config_server.json`**.

Restart Xray:

```
systemctl restart xray
```

Exit your SSH session with the server:

```
exit
```

## Client

Download and unzip the corresponding version of the client. Right now it will come from [https://github.com/XTLS/Xray-core/releases/tag/v1.6.2](https://github.com/XTLS/Xray-core/releases/tag/v1.6.2).

Configure a client configuration file `config.json` in the same folder as the binary. Use **`config_client.json`** as a template.

Start the proxy client running:

```
./xray -c config.json
```

## Firefox

From the Firefox hamburger menu, go to **Settings** > **General** > **Network Settings** > **Settings**.

Configure Firefox to use the SOCKS5 proxy on localhost port 10808.

* **Manual proxy configuration**
* SOCKS Host `127.0.0.1`
* Port `10808`
* **SOCKS v5**
* Check the box for **Proxy DNS when using SOCKS5**

Click **OK**.

Check that everything is working.
