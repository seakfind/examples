# VLESS-gRPC-TLS

Forked from [https://github.com/chika0801/Xray-examples/blob/main/VLESS-gRPC-TLS](https://github.com/chika0801/Xray-examples/blob/main/VLESS-gRPC-TLS)

## Server configuration method

### 1. Domain

Get a paid domain name. A cheap source is Namesilo. Create a DNS `A` record pointing from the hostname of your server (also known as its fully qualified domain name) to the IP address of your server. 

### 2. Firewall

Open ports `80/tcp` and `443/tcp` on your server firewall.

### 3. Install web server

Install Nginx on the server:

```
apt update && apt upgrade -y

apt install nginx -y
```

### 4. Set server name

Edit `/etc/nginx/sites-available/default`.

```
vi /etc/nginx/sites-available/default
```

Change `server_name` to your actual server name, e.g., `chika.example.com`.

```
        server_name chika.example.com;
```

Save the file. Restart Nginx:

```
systemctl restart nginx
```

### 5. Add web content

Add some HTML pages under `/var/www/html` for camouflage purposes:

```
rm /var/www/html/*

apt install git -y

git clone -b gh-pages https://github.com/PavelDoGreat/WebGL-Fluid-Simulation /var/www/html
```

### 6. Obtain SSL certificate and private key

After confirming that the DNS resolution has taken effect, execute the following commands (execute each command in sequence).

Note: Replace `chika.example.com` with your subdomain.

```
apt install -y socat
```

```
curl https://get.acme.sh | sh
```

```
alias acme.sh=~/.acme.sh/acme.sh
```

```
acme.sh --upgrade --auto-upgrade
```

```
acme.sh --set-default-ca --server letsencrypt
```

```
acme.sh --issue -d chika.example.com --nginx --keylength ec-256
```

```
acme.sh --install-cert -d chika.example.com --ecc \
--fullchain-file /etc/ssl/private/fullchain.cer \
--key-file /etc/ssl/private/private.key
```

```
chown -R nobody:nogroup /etc/ssl/private/
```

SSL certificates are valid for 90 days and are automatically renewed every 60 days. If the rate limit is exceeded, an error will be reported.

### 7. Install Xray

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

### 8. Download configuration

```
curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/seakfind/examples/main/VLESS-gRPC-TLS/config_server.json
```

### 9. Restart the Xray service

```
systemctl restart xray
```

```
systemctl status xray
```

- Configuration `/usr/local/etc/xray/config.json`
- Certificate `/etc/ssl/private/fullchain.cer`
- Private key `/etc/ssl/private/private.key`
- View logs `journalctl -u xray --output cat -e`
- Real-time log `journalctl -u xray --output cat -f`

### 10. Reconfigure web server

```
curl -Lo /etc/nginx/sites-available/default https://raw.githubusercontent.com/seakfind/examples/main/VLESS-gRPC-TLS/nginx.conf
```

### 11. Restart web server

```
systemctl restart nginx
```

```
systemctl status nginx
```

## v2rayN configuration method

![VLESS-gRPC-TLS](https://user-images.githubusercontent.com/88967758/180817492-7c165cd0-2d65-4901-85c5-c07f1b39e8c7.jpg)
