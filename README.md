

![image](https://github.com/molla202/Arkeo-Network/assets/91562185/02c57bf5-613c-4637-98ea-461fdc85e76f)

<h1 align="center"> Arkeo Network </h1>

 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>
 * [Arkeo Network Website](https://arkeo.network/)<br>
 * [Blockchain Explorer](https://explorer.nodexcapital.com/arkeo/)<br>

## Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| 500 GB SSD |

👉 Not:  linkten form doldurup coin istiyoruz. herhangi bir ödül yok.
https://forms.gle/aM6sc73qtxenRxf37

### Update ve kütüphane kuruyoruz
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux make build-essential jq make lz4 gcc unzip -y  
```
### Go kuruyoruz
```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
rm go1.19.3.linux-amd64.tar.gz
```
## Nodu kuruyoruz
👉 Not: NODE_MONIKER="molla202"  kısmındaki molla202 değiştiriniz kendi adınızı yazınız

```
#!/bin/bash

NODE_MONIKER="molla202"

wget https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/arkeod
chmod +x arkeod
mv arkeod $HOME/go/bin/

arkeod config keyring-backend test
arkeod config chain-id arkeo
arkeod init "$NODE_MONIKER" --chain-id arkeo

curl -s http://seed.arkeo.network:26657/genesis | jq '.result.genesis' > $HOME/.arkeo/config/genesis.json
curl -s https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/addrbook.json > $HOME/.arkeo/config/addrbook.json

SEEDS="20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22856"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.arkeo/config/config.toml

sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "17"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.arkeo/config/app.toml

sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uarkeo"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.arkeo/config/config.toml

sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Network Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which arkeod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

arkeod tendermint unsafe-reset-all --home $HOME/.arkeo --keep-addr-book


sudo systemctl daemon-reload
sudo systemctl enable arkeod
sudo systemctl start arkeod

sudo journalctl -u arkeod -f --no-hostname -o cat
```

### Validator
```
arkeod tx staking create-validator \
--amount=1000000uarkeo \
--pubkey=$(arkeod tendermint show-validator) \
--moniker="molla202" \
--identity= \
--details="" \
--chain-id=arkeo \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=molla202 \
--gas-prices=0.1uarkeo \
--gas-adjustment=1.5 \
--gas=auto \
-y
```
