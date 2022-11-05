# Shadowsocks 2022 with UDP over TCP

## Server

Install Xray to run as `root`:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root
```

Edit the configuration file created by the installation script:

```
vi /usr/local/etc/xray/config.json
```

Restart Xray with the new configuration file:

```
systemctl restart xray
```

## Client

Download the binary.

Create `config.json` modeled on the example.

Run the client:

```
./xray run -c config.json
```
