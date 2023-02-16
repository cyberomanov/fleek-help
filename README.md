# FLEEK

## Content
1. [Installation](https://github.com/cyberomanov/fleek-help/edit/main/README.md#installation)
2. [Auto-Upgrade](https://github.com/cyberomanov/fleek-help/edit/main/README.md#auto-upgrade)

## Installation

1. Update, upgrade and install requirements:
```shell
sudo apt-get update && \
sudo apt-get upgrade -y && \
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y && \
source "$HOME/.cargo/env" && \
cargo install sccache && \
sudo apt-get install build-essential cmake clang pkg-config libssl-dev protobuf-compiler -y
```
2. Clone repository and build:
```shell
git clone https://github.com/fleek-network/ursa && \
cd ursa && \
make install
```
4. Create a service:
```shell

sudo tee /etc/systemd/system/fleekd.service > /dev/null <<EOF
[Unit]
Description=Fleek Node
After=network.target
[Service]
User=$USER
Type=simple
ExecStart=$(which ursa)
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
5. Restart the service:
```shell
sudo systemctl daemon-reload && \
sudo systemctl enable fleekd && \
sudo systemctl restart fleekd && \
sudo journalctl -u fleekd -f -o cat
```
6. Set an identity:
```shell
IDENTITY="nickname"
```
7. Replace default identity:
```shell
systemctl stop fleekd && \
sed -i.bak -e "s/^identity *=.*/identity = \"${IDENTITY}\"/" $HOME/.ursa/config.toml && \
rm $HOME/.ursa/keystore/*; \
sudo systemctl restart fleekd && \
sudo journalctl -u fleekd -f -o cat
```

8. Backup:
```shell
echo -e "$HOME/.ursa/keystore/"
```

## Auto-Upgrade

1. Create bash script:
```shell
sudo tee ~/fleek_update.sh > /dev/null <<EOF
#!/bin/bash

cd ~/ursa && \
git fetch --all && \
git pull && \
make install && \

cat ~/.ursa/config.toml | grep "identity" && \

sudo systemctl restart fleekd
EOF
```
2. Edit script rights:
```shell
chmod u+x ~/fleek_update.sh
```
3. Open crontab editor:
```shell
crontab -e
```
4. Edit crontab rule:
```shell
# fleek
15 */4 * * * bash ~/fleek_update.sh >> ~/fleek_update.log
```
