# VLESS-gRPC-TLS

Forked from [https://github.com/chika0801/Xray-examples/blob/main/VLESS-gRPC-TLS](https://github.com/chika0801/Xray-examples/blob/main/VLESS-gRPC-TLS)

## Server configuration method

1. Install Xray

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

2. Download configuration

```
curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/seakfind/examples/main/VLESS-gRPC-XTLS/config_server.json
```

3. Obtain SSL certificate and private key

You first need to buy a domain name, then add a subdomain and point the subdomain to the IP of your VPS. Wait 5-10 minutes for DNS resolution to take effect. You can check if the returned IP is correct by pinging your subdomain. After confirming that the DNS resolution has taken effect, execute the following commands (execute each command in sequence).

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
acme.sh --issue -d chika.example.com --standalone --keylength ec-256
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

4. Start the Xray service

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

## v2rayN configuration method

![VLESS-gRPC-TLS](https://user-images.githubusercontent.com/88967758/180817492-7c165cd0-2d65-4901-85c5-c07f1b39e8c7.jpg)
