# VLESS + WSS + Nginx + CDN + uTLS

## A reasonably secure proxy server

This is a reasonably safe arrangement with these security features:

* The server hosts a real Nginx server with a real domain name and real content
* Traffic is transmitted as WebSocket protected by TLS
* The server's IP address is hidden behind a Content Distribution Network
* Traffic TLS parrots a browser TLS Client Hello by using the uTLS library

## Domain

Get a paid domain name. A cheap source is [Namesilo](https://www.namesilo.com).

When you have your domain name, add your domain to Cloudflare Content Distribution Network (CDN). A CDN conceals your server's IP address. Select the Free plan. Cloudflare will tell you the new nameservers to use. At your domain name registrar, change the nameservers to be Cloudflare nameservers. You may need to wait up to 24 hours for this change to take effect. 

Once added, go to the **DNS** page in Cloudflare. Create a DNS `A` record pointing from the hostname of your server (also known as its fully qualified domain name) to the IP address of your server. Leave proxying off (i.e., Proxy status `DNS only`) for now.

## Web server

Open ports `80/tcp` and `443/tcp` on your server firewall.

Install Nginx on the server:

```
apt update && apt upgrade -y

apt install nginx -y
```

Edit the Nginx default site configuration file:

```
vi /etc/nginx/sites-available/default
```

Change the `server_name` to be the hostname (also known as fully qualified domain name) of your server. For example:

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
        server_name webgl.example.com;
        location / {
                try_files $uri $uri/ =404;
        }
}
```

Save the file with this change. Restart your web server:

```
systemctl restart nginx

systemctl status nginx
```

Add some HTML pages under `/var/www/html` for camouflage purposes:

```
rm /var/www/html/*

apt install git -y

git clone -b gh-pages https://github.com/PavelDoGreat/WebGL-Fluid-Simulation /var/www/html
```

## SSL cert

Install the packages for a free Let's Encrypt SSL certificate:

```
apt install certbot python3-certbot-nginx -y
```

Request the certificate and key, replacing `webgl.example.com` in the example by your actual server hostname:

```
certbot --nginx --agree-tos --register-unsafely-without-email -d webgl.example.com
```

Set everything up for renewal:

```
certbot renew --dry-run
```

## Xray server

Right now, uTLS is in a release, but Vision is in an experimental prerelease. Therefore we request specifically version `1.6.2` at this time.

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root --version 1.6.2
```

Generate a secret UUID:

```
uuidgen
```

Example of results: `b28d21bf-037a-4a19-a3e6-45a409bbd21f`.

Generate a secret path:

```
tr -dc a-z0-9 </dev/urandom | head -c 8 ; echo ''
```

Example of results: `uig3pkig`.

Edit Nginx configuration file:

```
vi /etc/nginx/sites-available/default
```

Add a `location` block for the secret path. The final Nginx configuration file should be similar to the example **`nginx.conf`**.

Edit the Xray server configuration file:

```
vi /usr/local/etc/xray/config.json
```

Delete the dummy line and insert a configuration similar to the example **`config_server.json`**.

Restart Xray and Nginx:

```
systemctl restart xray

systemctl restart nginx
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

## Add CDN

When everything is working, turn proxying on in the server's **DNS** entry in Cloudflare.

Also make sure that **SSL/TLS** mode is set to **Full** or **Full (strict)**.
