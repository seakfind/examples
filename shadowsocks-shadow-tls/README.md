# Shadowsocks + Shadow-TLS

Shadow-TLS presents to the outside observer a real TLS handshake with a trusted site. This obscures the fact that data for the true destination is being transferred behind the scenes. The concept is illustrated in a diagram in the [docs](https://github.com/ihciah/shadow-tls/blob/master/docs/protocol-en.md), _quod vide_. 

Shadow-TLS is available for macOS and Linux _but not for Windows_.

In the scenario in this example:

* The Shadowsocks client, running on an Ubuntu PC, listens on localhost port `10808`. SS forwards encrypted traffic to localhost port `8001`.
* The Shadow-TLS client is listening on localhost port `8001`. It talks to Shadow-TLS on the server on port `443`. Their communication appears to include a real TLS handshake with the trusted site.
* The Shadow-TLS server listens on port `443`. It relays the TLS handshake with the trusted site. Behind the scense, it forwards the true data to SS on port `8002` on the same server.
* The SS server listens on server internal port `8002`. SS sends out the traffic to its final destination.

## Server

Install Shadowsocks on your Debian or Ubuntu server:

```
apt update && apt upgrade -y

apt install shadowsocks-libev -y
```

Generate a 96-bit (12-byte) password for Shadowsocks, expressed as 16 base-64 characters:

```
openssl rand -base64 12
```

Our example Shadowsocks password will be:

```
z3LJ8yYOEKB7tPFy
```

Remember that there will be two passwords in this scenario. The Shadow-TLS password will be different from the Shadowsocks password.

Configure Shadowsocks:

```
vi /etc/shadowsocks-libev/config.json 
```

Insert a configuration based on the example **`config_server.json`**.

Save the file.

Restart Shadowsocks with this configuration:

```
systemctl restart shadowsocks-libev
```

Check that Shadowsocks is `active (running)`:

```
systemctl status shadowsocks-libev
```

Check that Shadowsocks is listening on port `8002` of the server:

```
ss -tulpn | grep 8002
```

Open port `443/tcp` on the server.

In a browser, visit the [Shadow-TLS releases page](https://github.com/ihciah/shadow-tls/releases). Determine the most recent version. Right now it is `v0.2.3`.

Install the latest Shadow-TLS binary on your server. For `v0.2.3` the commands are:

```
wget https://github.com/ihciah/shadow-tls/releases/download/v0.2.3/shadow-tls-x86_64-unknown-linux-musl

cp shadow-tls-x86_64-unknown-linux-musl /usr/bin

chmod +x /usr/bin/shadow-tls-x86_64-unknown-linux-musl
```

Generate a 96-bit (12-byte) password for Shadow-TLS, expressed as 16 base-64 characters. This will be different from your Shadowsocks password:

```
openssl rand -base64 12
```

Our example Shadow-TLS password is:

```
tmZhXnRPOKD1DG7V
```

Create a systemd service file 

```
vi /usr/lib/systemd/system/shadow-tls.service
```

Modeled the contents on the example **`shadow-tls.service`**

Save the file.

Start the Shadow-TLS service running:

```
systemctl daemon-reload

systemctl enable shadow-tls

systemctl start shadow-tls
```

Check that Shadow-TLS is `active (running)`:

```
systemctl status shadow-tls
```

Check that Shadow-TLS is listening on port `443` of the server:

```
ss -tulpn | grep 443
```

Finish your SSH session with the server:

```
exit
```

## Client

Install Shadowsocks-Libev:

```
sudo apt update && sudo apt upgrade -y

sudo apt install shadowsocks-libev -y
```

Stop the SS server from running, as you do not need it:

```
sudo systemctl stop shadowsocks-libev

sudo systemctl disable shadowsocks-libev
```


Create a new configuration file for the client:

```
sudo vi /etc/shadowsocks-libev/client.json
```

Insert a configuration modeled on the example **`config_client.json`**.

Save the file.

Run Shadowsocks-Libev local client like this:

```
sudo systemctl enable shadowsocks-libev-local@client

sudo systemctl start shadowsocks-libev-local@client
```

Check that Shadowsocks-Libev is `active (running)` like this:

```
sudo systemctl status shadowsocks-libev-local@config
```

Check that Shadow-Libev is listening on localhost port `10808`:

```
sudo ss -tulpn | grep 10808
```

Visit the [Shadow-TLS releases page](https://github.com/ihciah/shadow-tls/releases). Determine the most recent version. Right now it is `v0.2.3`.

Download the most recent `shadow-tls-x86_64-unknown-linux-musl`.

Start the Shadow-TLS client running in a terminal window.

* Replace `tmZhXnRPOKD1DG7V` in the command below by your actual Shadow-TLS password
* Replace `123.123.123.123` by your actual server IP address
* Replace `cloud.tencent.com` by the SNI of a trusted server

```
cd ~/Downloads

chmod +x shadow-tls-x86_64-unknown-linux-musl

./shadow-tls-x86_64-unknown-linux-musl client --listen 127.0.0.1:8001 --password tmZhXnRPOKD1DG7V --server 123.123.123.123:443 --sni cloud.tencent.com
```

Leave the terminal window open.

Open Firefox. From the hamburger menu, select **Settings** &gt; **General**, and under **Network Settings** click **Settings**.

* Select **Manual proxy configuration**
* Enter SOCKS Host `127.0.0.1`
* Enter Port `10808`
* Select **SOCKS v5**
* Check the box for **Proxy DNS when using SOCKS v5**

Click **OK**.
