# VLESS-TCP-XTLS-Vision

Forked from https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS-Vision and translated by Google Translate.

## [XTLS Vision](https://github.com/XTLS/Xray-core/discussions/1295) Installation guide

1. Install

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

2. Download configuration

```
curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/seakfind/examples/main/VLESS-TCP-XTLS-Vision/config_server.json
```

3. Certificate and private key

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

To back up the applied SSL certificate: Use WinSCP to log in to your VPS, enter the `/etc/ssl/private/` directory, and download the certificate file `fullchain.cer` and the private key file `private.key`. To restore, rename the certificate file to `fullchain.cer` and the private key file to `private.key`, log in to your VPS using WinSCP, upload them to the `/etc/ssl/private/` directory and execute the following command: `chown -R nobody:nogroup /etc/ssl/private/`. See [Insufficient privileges when using certificates](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates-zh-Hans-CN).

4. Starting program

```
systemctl start xray
```

```
systemctl status xray
```

- Configuration `/usr/local/etc/xray/config.json`
- Certificate `/etc/ssl/private/fullchain.cer`
- Private key `/etc/ssl/private/private.key`
- View logs `journalctl -u xray --output cat -e`
- Real-time log `journalctl -u xray --output cat -f`

## v2rayN Configuration guide

1. `Servers` ——> `Add [VLESS] server`

![1](https://user-images.githubusercontent.com/88967758/199511522-f3d26687-34df-48c7-bff4-b5d1784ecca5.jpg)

## Custom configuration

1. Download client configuration [config_client.json](https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS-Vision/config_client.json)

2. Find `"address": ""`, enter the IP of the VPS in the middle of `""`. Find `"serverName": ""`, enter the domain name included in the certificate in the middle of `""`.

3. `Server` ——> `Add Custom Configuration Server` ——> `Browse(B)` ——> `Select Client Configuration` ——> `Core Type Xray` ——> `Socks Port 0`

![1](https://user-images.githubusercontent.com/88967758/199512235-7f7d78a6-e27d-4db8-b6f5-7ef4212f1af9.jpg)

Tip: As long as the certificate is valid, the domain name contained in the certificate does not need to be resolved to the VPS IP. One certificate, used on multiple VPS.

## v2rayNG Configuration guide

地址(address) `VPS的IP`
<br/>
端口(port) `16387`
<br/>
用户ID(id) `chika`
<br/>
流控(flow) `xtls-rprx-vision`
<br/>
传输协议(network) `tcp`
<br/>
传输层安全(tls) `tls`
<br/>
SNI `证书中包含的域名`
