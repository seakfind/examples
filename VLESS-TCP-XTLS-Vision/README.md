# VLESS-TCP-XTLS-Vision

Forked from https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS-Vision

## [XTLS Vision](https://github.com/XTLS/Xray-core/discussions/1295)安装指南

1. 安装程序

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --beta
```

2. 下载配置

```
curl -Lo /usr/local/etc/xray/config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS-Vision/config_server.json
```

3. 上传证书和私钥

- 将证书文件改名为`fullchain.cer`，将私钥文件改名为`private.key`，使用WinSCP登录你的VPS，将它们上传到`/etc/ssl/private/`目录，执行下面的命令。

```
chown -R nobody:nogroup /etc/ssl/private/
```

- [使用证书时权限不足](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates-zh-Hans-CN)
- [用acme申请 SSL 证书](https://github.com/chika0801/Xray-install#1%E7%94%A8acme%E7%94%B3%E8%AF%B7-ssl-%E8%AF%81%E4%B9%A6)

4. 启动程序

```
systemctl start xray
```

```
systemctl status xray
```

- 配置 `/usr/local/etc/xray/config.json`
- 证书 `/etc/ssl/private/fullchain.cer`
- 私钥 `/etc/ssl/private/private.key`
- 查看日志 `journalctl -u xray --output cat -e`
- 实时日志 `journalctl -u xray --output cat -f`

## v2rayN配置指南

1. `服务器` ——> `添加[VLESS服务器]`

![1](https://user-images.githubusercontent.com/88967758/199511522-f3d26687-34df-48c7-bff4-b5d1784ecca5.jpg)

## 自定义配置

1. 下载客户端配置[config_client.json](https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS-Vision/config_client.json)

2. 找到`"address": ""`，在`""`中间输入VPS的IP。找到`"serverName": ""`，在`""`中间输入证书中包含的域名。

3. `服务器` ——> `添加自定义配置服务器` ——> `浏览(B)` ——> `选择客户端配置` ——> `Core类型 Xray` ——> `Socks端口 0`

![1](https://user-images.githubusercontent.com/88967758/199512235-7f7d78a6-e27d-4db8-b6f5-7ef4212f1af9.jpg)

小技巧：只要证书在有效期内，证书中包含的域名不用解析到VPS的IP。一份证书，在多个VPS上使用。

## v2rayNG配置指南

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
