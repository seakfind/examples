# VLESS-TCP-XTLS-Vision

Forked from https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS-Vision

## [XTLS Vision](https://github.com/XTLS/Xray-core/discussions/1295)安装指南

1. Installation guide

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

2. Download configuratiojn

```
curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS-Vision/config_server.json
```

3. Upload certificate and private key

- Rename the certificate file to `fullchain.cer` and the private key file to `private.key`, log into your VPS using WinSCP, upload them to the `/etc/ssl/private/` directory and execute the following command.

```
chown -R nobody:nogroup /etc/ssl/private/
```

- [Insufficient privileges when using certificates](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates-zh-Hans-CN)
- [Apply for SSL certificate with acme](https://github.com/chika0801/Xray-install#1%E7%94%A8acme%E7%94%B3%E8%AF%B7-ssl-%E8%AF%81%E4%B9%A6)

4. Starting program

```
systemctl start xray
```

```
systemctl status xray
```

- Configure `/usr/local/etc/xray/config.json`
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
端口(prot) `16387`
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
