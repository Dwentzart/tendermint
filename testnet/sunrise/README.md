### Install prequirement
```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### Install go
```
ver="1.21.4"
```
```
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
```
```
sudo rm -rf /usr/local/go
```
```
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
```
```
rm "go$ver.linux-amd64.tar.gz"
```
```
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
go version
```

### Download binary
```
wget https://github.com/sunriselayer/sunrise/releases/download/v0.1.4/sunrised
chmod +x sunrised
mv sunrised /root/go/bin
```
### Config Node

* Initialize the node
```
sunrised init "Dwentz" --chain-id sunrise-test-0.2
```
* Init Config
```
sunrised config set client chain-id sunrise-test-0.2
sunrised config set client keyring-backend test
sunrised config set client node tcp://localhost:34657
```
* Download Genesis && AddrBook
```
wget -O ~/.sunrise/config/genesis.json https://raw.githubusercontent.com/sunriselayer/network/main/sunrise-test-0.2/genesis.json

```
* Custom port
```
PORT=11
```
```
sunrised config node tcp://localhost:${PORT}657
```
```
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.sunrise/config/config.toml
sed -i -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://localhost:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"localhost:9090\"%address = \"localhost:${PORT}090\"%; s%^address = \"localhost:9091\"%address = \"localhost:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.sunrise/config/app.toml

```

* Add Peers or Seed
```
sed -i -e 's|^seeds *=.*|seeds = "0c0e0cf617c1c58297f53f3a82cea86a7c860396@a.sunrise-test-1.cauchye.net:26656,db223ecc4fba0e7135ba782c0fd710580c5213a6@a-node.sunrise-test-1.cauchye.net:26656,82bc2fdbfc735b1406b9da4181036ab9c44b63be@b-node.sunrise-test-1.cauchye.net:26656"|' $HOME/.sunrise/config/config.toml

```
* Setup minimum GasFee
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.25urise"|' $HOME/.sunrise/config/app.toml

```

* Setup pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.sunrise/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/sunrised.service > /dev/null <<EOF
[Unit]
Description=sunrised 
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sunrised) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### Launch Node
```
sudo systemctl daemon-reload 
```
```
sudo systemctl enable sunrised
```
```
sudo systemctl restart sunrised
```
### Check log node
```
journalctl -fu sunrised -o cat
```
### Wallet configuration
* add wallet
```
sunrised keys add wallet
```
* recover wallet
```
sunrised keys add wallet --recover
```
* list wallet
```
sunrised keys list
```
* delete wallet
```
sunrised keys delete wallet
```
* check balances
```
sunrised q bank balances $(sunrised keys show wallet -a)
```
### Validator Management
* create validator
```
sunrised tx staking create-validator \
--amount="1000000000urise" \
--pubkey=$(sunrised tendermint show-validator) \
--moniker=NodeName \
--identity="F57A71944DDA8C4B" \
--website="https://dwentz.xyz" \
--details=details node validator \
--chain-id=sunrise-test-0.2 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000000 \
--broadcast-mode block \
--gas-adjustment=1.2 \
--gas-prices="0.5urise" \
--gas=auto \
--from=wallet
```
* edit validator
```
sunrised tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=sunrise-test-0.2 \
--gas-adjustment=1.2 \
--gas-prices="0.5urise" \
--gas=auto \
--from=wallet
```
* unjail validator
```
sunrised tx slashing unjail --from wallet --chain-id sunrise-test-0.2 --gas-prices 0.5urise --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
sunrised query slashing signing-info $(sunrised tendermint show-validator)
```
### Token management

* withdraw rewards
```
sunrised tx distribution withdraw-all-rewards --from wallet --chain-id sunrise-test-0.2  --gas-adjustment 1.2 --gas-prices 0.5urise --gas auto -y
```
* withdraw rewards with comission
```
sunrised tx distribution withdraw-rewards $(sunrised keys show wallet --bech val -a) --commission --from wallet --chain-id sunrise-test-0.2  --gas-adjustment 1.2 --gas-prices 0.5urise --gas auto -y
```
* delegate token to your own validator
```
sunrised tx staking delegate $(sunrised keys show wallet --bech val -a) 1000000urise --from wallet --chain-id sunrise-test-0.2  --gas-adjustment 1.2 --gas-prices 0.5urise --gas auto -y
```
### Node Info
* sync info
```
sunrised status 2>&1 | jq .sync_info
```
* node id
```
sunrised status 2>&1 | jq .node_info
```
* validator info
```
sunrised status 2>&1 | jq .validator_info
```

### Delete Node
```
sudo systemctl stop sunrised &&
sudo systemctl disable sunrised &&
sudo rm /etc/systemd/system/sunrised.service &&
sudo systemctl daemon-reload &&
rm -f $(which sunrised) 
```
