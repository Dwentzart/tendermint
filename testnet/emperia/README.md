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
curl -LO https://github.com/empe-io/empe-chain-releases/raw/master/v0.1.0/emped_linux_amd64.tar.gz
tar -xvf emped_linux_amd64.tar.gz
sudo mv emped ~/go/bin
chmod u+x ~/go/bin/emped
```
### Config Node
* Init Node
```
emped init Dwentz --chain-id empe-testnet-2
emped config chain-id empe-testnet-2
emped config keyring-backend test
```
* Download Genesis && AddrBook
```
wget -O $HOME/.empe-chain/config/genesis.json https://raw.githubusercontent.com/empe-io/empe-chains/master/testnet-2/genesis.json
wget -O $HOME/.empe-chain/config/addrbook.json https://raw.githubusercontent.com/vinjan23/Testnet.Guide/main/Empeiria/addrbook.json
```
* Custom port
```
PORT=11
```
```
emped config node tcp://localhost:${PORT}657
```
```
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.empe-chain/config/config.toml
sed -i -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://localhost:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"localhost:9090\"%address = \"localhost:${PORT}090\"%; s%^address = \"localhost:9091\"%address = \"localhost:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.empe-chain/config/app.toml

```

* Add Peers or Seed
```
peers="7419cc5bafc3c9c49b18ec67e3263344cbcc30f2@49.13.215.45:43656,91fb8e75a4b92589211d47d5a9ce934d32733043@116.202.48.104:26656,771b5350519d6081784f70d9be692e76b635529a@188.245.88.126:43656,06bd2afb77d94d2f3c5fe8abbdabb4499ca95793@65.21.97.150:40656,5406f64d38f433cca31c2f6e96d5619fa92be5b5@168.119.179.250:26656,e941537fbe6d86c9bf58aeb880e3212dc3f82086@84.247.141.94:656,94529b5e044f208d1869980f456a53fcef8fb321@14.167.155.13:43656,829207ca2cf7debb16787a79c9fc1aa94e9b55ea@116.203.238.65:43656,080d9cc12e08fb64dd0f4528d0da4a84d5d9428e@37.27.83.234:26656,9e120e5cb5822fbb224d5c1ae2c7f0149b80fc99@45.67.216.220:43656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.empe-chain/config/config.toml


```
* Setup minimum GasFee
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.05uempe\"/" $HOME/.empe-chain/config/app.toml

```

* Setup pruning
```
sed -i \
-e 's|^pruning *=.*|pruning = "custom"|' \
-e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
-e 's|^pruning-keep-every *=.*|pruning-keep-every = ""|' \
-e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
$HOME/.empe-chain/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/emped.service > /dev/null <<EOF
[Unit]
Description=Emped 
After=network-online.target

[Service]
User=$USER
ExecStart=$(which emped) start
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
sudo systemctl enable emped
```
```
sudo systemctl restart emped
```
### Check log node
```
journalctl -fu emped -o cat
```
### Wallet configuration
* add wallet
```
emped keys add wallet
```
* recover wallet
```
emped keys add wallet --recover
```
* list wallet
```
emped keys list
```
* delete wallet
```
emped keys delete wallet
```
* check balances
```
emped q bank balances $(emped keys show wallet -a)
```
### Validator Management
* create validator
```
emped tx staking create-validator \
--amount="1000000000uempe" \
--pubkey=$(emped tendermint show-validator) \
--moniker=NodeName \
--identity="F57A71944DDA8C4B" \
--website="https://dwentz.xyz" \
--details=details node validator \
--chain-id=empe-testnet-2 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000 \
--gas-adjustment=1.2 \
--gas-prices="0.5uempe" \
--gas=auto \
--from=wallet
```
* edit validator
```
emped tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=empe-testnet-2 \
--gas-adjustment=1.2 \
--gas-prices="0.5uempe" \
--gas=auto \
--from=wallet
```
* unjail validator
```
emped tx slashing unjail --from wallet --chain-id empe-testnet-2 --gas-prices 0.5uempe --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
emped query slashing signing-info $(emped tendermint show-validator)
```
### Token management

* withdraw rewards
```
emped tx distribution withdraw-all-rewards --from wallet --chain-id empe-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5uempe --gas auto -y
```
* withdraw rewards with comission
```
emped tx distribution withdraw-rewards $(emped keys show wallet --bech val -a) --commission --from wallet --chain-id empe-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5uempe --gas auto -y
```
* delegate token to your own validator
```
emped tx staking delegate $(emped keys show wallet --bech val -a) 1000000uempe --from wallet --chain-id empe-testnet-2  --gas-adjustment 1.2 --gas-prices 0.5uempe --gas auto -y
```
### Node Info
* node id
```
emped status 2>&1 | jq .NodeInfo
```
* validator info
```
emped status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop emped &&
sudo systemctl disable emped &&
sudo rm /etc/systemd/system/emped.service &&
sudo systemctl daemon-reload &&
rm -f $(which emped) &&
rm -rf .empe-chain
```
