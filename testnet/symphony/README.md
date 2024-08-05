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
cd $HOME
git clone https://github.com/Orchestra-Labs/symphony
cd symphony
git checkout v0.2.1
make install
```
### Config Node
* Init Node
```
symphonyd init Dwentz --chain-id symphony-testnet-2
symphonyd config chain-id symphony-testnet-2
symphonyd config keyring-backend test
```
* Download Genesis && AddrBook
```
wget -O $HOME/.symphonyd/config/genesis.json https://raw.githubusercontent.com/Orchestra-Labs/symphony/main/networks/symphony-testnet-2/genesis.json
wget -O $HOME/.symphonyd/config/addrbook.json https://raw.githubusercontent.com/vinjan23/Testnet.Guide/main/Symphony/addrbook.json
```
* Custom port
```
PORT=10
```
```
symphonyd config node tcp://localhost:${PORT}657
```
```
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.symphonyd/config/config.toml
sed -i -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://localhost:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"localhost:9090\"%address = \"localhost:${PORT}090\"%; s%^address = \"localhost:9091\"%address = \"localhost:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.symphonyd/config/app.toml

```

* Add Peers or Seed
```
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.symphonyd/config/config.toml
peers="016eb93b77457cbc8793ba1ee01f7e2fa2e63a3b@136.243.13.36:29156,8df964c61393d33d11f7c821aba1a72f428c0d24@34.41.129.120:26656,298743e0b4813ada523e26922d335a3fb37ec58a@37.27.195.219:26656,785f5e73e26623214269909c0be2df3f767fbe50@35.225.73.240:26656,22e9b542b7f690922e846f479878ab391e69c4c3@57.129.35.242:26656,9d4ee7dea344cc5ca83215cf7bf69ba4001a6c55@5.9.73.170:29156,77ce4b0a96b3c3d6eb2beb755f9f6f573c1b4912@178.18.251.146:22656,27c6b80a1235d41196aa56459689c28f285efd15@136.243.104.103:24856,adc09b9238bc582916abda954b081220d6f9cbc2@34.172.132.224:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.symphonyd/config/config.toml

```
* Setup minimum GasFee
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0note\"/" $HOME/.symphonyd/config/app.toml

```

* Setup pruning
```
sed -i \
-e 's|^pruning *=.*|pruning = "custom"|' \
-e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
-e 's|^pruning-keep-every *=.*|pruning-keep-every = ""|' \
-e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
$HOME/.symphonyd/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/symphonyd.service > /dev/null <<EOF
[Unit]
Description=symphony
After=network-online.target

[Service]
User=$USER
ExecStart=$(which symphonyd) start
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
sudo systemctl enable symphonyd
```
```
sudo systemctl restart symphonyd
```
### Check log node
```
journalctl -fu symphonyd -o cat
```
### Wallet configuration
* add wallet
```
symphonyd keys add wallet
```
* recover wallet
```
symphonyd keys add wallet --recover
```
* list wallet
```
symphonyd keys list
```
* delete wallet
```
symphonyd keys delete wallet
```
* check balances
```
symphonyd q bank balances $(symphonyd keys show wallet -a)
```
### Validator Management
* create validator
```
symphonyd tx staking create-validator \
--amount="1000000000note" \
--pubkey=$(symphonyd tendermint show-validator) \
--moniker=NodeName \
--identity="F57A71944DDA8C4B" \
--website="https://dwentz.xyz" \
--details=details node validator \
--chain-id=symphony-testnet-2 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000 \
--gas-adjustment=1.2 \
--gas-prices="0.5note" \
--gas=auto \
--from=wallet
```
* edit validator
```
symphonyd tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=symphony-testnet-2 \
--gas-adjustment=1.2 \
--gas-prices="0.5note" \
--gas=auto \
--from=wallet
```
* unjail validator
```
symphonyd tx slashing unjail --from wallet --chain-id symphony-testnet-2 --gas-prices 0.5note --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
symphonyd query slashing signing-info $(symphonyd tendermint show-validator)
```
### Token management

* withdraw rewards
```
symphonyd tx distribution withdraw-all-rewards --from wallet --chain-id symphony-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5note --gas auto -y
```
* withdraw rewards with comission
```
symphonyd tx distribution withdraw-rewards $(symphonyd keys show wallet --bech val -a) --commission --from wallet --chain-id symphony-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5note --gas auto -y
```
* delegate token to your own validator
```
symphonyd tx staking delegate $(symphonyd keys show wallet --bech val -a) 1000000note --from wallet --chain-id symphony-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5note --gas auto -y
```
### Node Info
* node id
```
symphonyd status 2>&1 | jq .NodeInfo
```
* validator info
```
symphonyd status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop symphonyd &&
sudo systemctl disable symphonyd &&
sudo rm /etc/systemd/system/symphonyd.service &&
sudo systemctl daemon-reload &&
rm -f $(which symphonyd) &&
rm -rf .symphonyd
```
