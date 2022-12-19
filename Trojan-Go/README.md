# Trojan-Go

Forked from [https://www.losem.tk/948](https://www.losem.tk/948).

YouTube video tutorial [https://www.youtube.com/watch?v=UXuXC3TOlyU](https://www.youtube.com/watch?v=UXuXC3TOlyU).

## 0. Starting point

You have a virtual private server (VPS), a domain name, and a DNS record pointing from fully qualified domain name of server to IP address of server.

## 1. Open firewall

Either disable your firewall completely, or open ports 22/tcp (SSH), 80/tcp (HTTP), and 443/tcp (HTTPS).

## 2. Apply BBR congestion control

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

## 3. Update software

```
apt update && apt upgrade -y
```

## 4. Install prerequisite packages

```
apt install -y wget unzip socat
```

## 5. Create folder for trojan-go

```
mkdir /etc/trojan-go
```

## 6. Enter trojan-go folder

```
cd /etc/trojan-go
```

## 7. Download trojan-go

Determine latest release by visiting [https://github.com/p4gefau1t/trojan-go/releases](https://github.com/p4gefau1t/trojan-go/releases) in a browser.

Download latest release to server, e.g. 

```
wget https://github.com/p4gefau1t/trojan-go/releases/download/v0.10.6/trojan-go-linux-amd64.zip
```

## 8. Unzip trojan-go

```
unzip trojan-go-linux-amd64.zip
```

## 9. Create config.json file

```
curl -Lo config.json https://raw.githubusercontent.com/seakfind/examples/main/Trojan-Go/config_server.json
```

Edit `config.json`.

Replace `your-domain-name.com` by your actual server fully qualified domain name.

Replace `your_awesome_password` by your choice for password.

Replace `/ray123` by your choice for path.

## 10. Exit trojan-go directory

```
cd
```

## 11. Install acme:

```
curl https://get.acme.sh | sh
```

## 12. Create soft link:

```
ln -s /root/.acme.sh/acme.sh /usr/local/bin/acme.sh
```

## 13. Switch CA organization

```
acme.sh --set-default-ca --server letsencrypt
```

## 14. Apply for certificate

Replace `your-domain-name.com` by your actual server fully qualified domain name.

For server with IPv4:

```
acme.sh --issue -d your-domain-name.com --standalone --keylength ec-256
```

For IPv6-only server:

```
acme.sh --issue -d your-domain-name.com --listen-v6 --standalone --keylength ec-256
```

## 15. Install certificate

Replace `your-domain-name.com` by your actual server fully qualified domain name.

```
acme.sh --install-cert -d your-domain-name.com --ecc --key-file /etc/trojan-go/server.key --fullchain-file /etc/trojan-go/server.crt
```

## 16. Install Nginx

```
apt install -y nginx
```

## 17. Modify nginx configuration file

```
curl -Lo /etc/nginx/sites-available/default https://raw.githubusercontent.com/seakfind/examples/main/Trojan-Go/default
```

Edit `/etc/nginx/sites-available/default`.

Replace `your-domain-name.com` by your actual server fully qualified domain name.

Replace `www.bing.com` by your camouflage URL.

## 18. Reload nginx

```
systemctl reload nginx
```

## 19. View nginx status

```
systemctl status nginx
```

## 20. Create trojan-go.service file

```
curl -Lo /etc/systemd/system/trojan-go.service https://raw.githubusercontent.com/seakfind/examples/main/Trojan-Go/trojan-go.service
```

## 21. Reload daemon

```
systemctl daemon-reload
```

## 22. Set boot automatically

```
systemctl enable trojan-go
```

## 23. Start trojan-go

```
systemctl start trojan-go
```

## 24. View trojan-go process

```
systemctl status trojan-go
```

## 25. CDN

Apply CDN proxying (orange cloud in Cloudflare) and SSL/TLS mode Full.

![v2rayn-trojan-go](https://user-images.githubusercontent.com/90979794/208422007-3898ea8d-c5e0-46fd-8acc-c969d840b1fa.png)
