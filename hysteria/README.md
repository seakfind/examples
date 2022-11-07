# Hysteria

Hysteria, also known as "Hy," is a proxy protocol derived from Google QUIC (Quick UDP Internet Connections). It has a reputation for being faster than other protocols, especially if you have an ISP who throttles your traffic in the interests of "Quality of Service" (QoS).

## Server

Generate a random port number for Hysteria. 

```
echo $(($RANDOM + 10000))
```

We will give port `19732` as our example.

Open port `19732/udp` in your server firewall.

Generate a strong password:

```
openssl rand -base64 24
```

We will use as our example `Nmtwgq5GdLJ1l2GHzOhJXAwbH2YZETm5`.

In a browser, look up the most recent version of Hy on the [GitHub releases page](https://github.com/HyNetwork/hysteria/releases). Right now it is `v1.2.2`.

Install the latest binary on your server:

```
wget https://github.com/HyNetwork/hysteria/releases/download/v1.2.2/hysteria-linux-amd64

cp hysteria-linux-amd64 /usr/bin/

chmod +x /usr/bin/hysteria-linux-amd64
```

Create a configuration file:

```
mkdir /etc/hysteria

vi /etc/hysteria/config.json
```

Insert contents modeled on the example **`config_server.json`**.

Save the file.

Create a systemd service file:

```
vi /usr/lib/systemd/system/hysteria.service
```

(Some distributions store the systemd service files in `/lib` instead of `/usr/lib`).

Insert contents modeled on the example **`hysteria.service`**.

Save the file.

Run Hysteria like this:

```
systemctl daemon-reload

systemctl enable hysteria

systemctl start hysteria
```

Check that Hysteria is `active (running)`:

```
systemctl status hysteria
```

Check that Hysteria is listening on the expected port:

```
ss -tulpn | grep hysteria
```

End your SSH session with the server:

```
exit
```

## Client

Look up the most recent version on the [GitHub releases page](https://github.com/HyNetwork/hysteria/releases). Right now it is `v1.2.2`. Download the latest binary. By default it will go into your `~/Downloads` folder with a name of `hysteria-linux-amd64`.

Set the execution bit:

```
cd ~/Downloads

chmod +x hysteria-linux-amd64
```

Create a client configuration file in the same folder as the binary:

```
vi config.json
```

Insert contents modeled on the example **`config_client.json`**.

Make sure you customize it with your actual server IP address and hostname before you save the file.

Run Hysteria:

```
./hysteria-linux-amd64 client -c config.json
```

You should see `[INFO]` messages to say `Client configuration loaded`, `Connected`, and `SOCKS5 server up and running`.

Leave the terminal window open with Hysteria running in it.

Open Firefox. Under **Settings**, scroll down to **Network Settings** and click the **Settings** button. Configure your Firefox network settings like this:

* Select **Manual proxy configuration**
* Set SOCKS Host `127.0.0.1`
* Set Port `10808`
* Select **SOCKS v5**
* Check **Proxy DNS when using SOCKS v5**
* Check **Enable DNS over HTTPS**

Click **OK** to save these settings.
