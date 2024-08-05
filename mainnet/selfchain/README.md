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
### Install cosmovisor
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```
### Download binary
```
cd $HOME
wget https://github.com/hotcrosscom/Self-Chain-Releases/releases/download/mainnet-v1.0.1/selfchaind-linux-amd64
mv selfchaind-linux-amd64 selfchaind
chmod +x selfchaind
```
```
mkdir -p $HOME/.selfchain/cosmovisor/genesis/bin
mv selfchaind $HOME/.selfchain/cosmovisor/genesis/bin/
```
```
sudo ln -s $HOME/.selfchain/cosmovisor/genesis $HOME/.selfchain/cosmovisor/current -f
sudo ln -s $HOME/.selfchain/cosmovisor/current/bin/selfchaind /usr/local/bin/selfchaind -f
```
### Config Node
* Init Node
```
selfchaind init dwentz --chain-id self-1
```
```
selfchaind config chain-id self-1
selfchaind config keyring-backend file
```
* Download Genesis && AddrBook
```
curl -Ls https://raw.githubusercontent.com/molla202/Self-Chain-Mainnet/main/genesis.json > $HOME/.selfchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/molla202/Self-Chain-Mainnet/main/addrbook.json > $HOME/.selfchain/config/addrbook.json
```
* Custom port
```
PORT=19
```
```
selfchaind config node tcp://localhost:${PORT}657
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.selfchain/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.selfchain/config/app.toml
```
* Add Peers or Seed
```
SEEDS=
PEERS="f238d6a52578975198ceac2b0c2b004d49d5613f@88.198.5.77:31656,7a9038d1efd34c7f3baea17d8822262a981568b1@217.182.136.79:30156,b844793daeffaedfcdbd5b08688cd10e1859d678@37.120.245.116:26656,b307b56b94bd3a02fcad5b6904464a391e13cf48@128.199.33.181:26656,5bfe7ec3ce0fbbf6d724dc85edef31c23b0a5e5e@94.130.138.48:41656,8401cbf633c496e464a2d016b333f61ff34e9ee9@167.71.233.135:26656,2f547f93392d7351c74a0d8cae1d44f172cf32e5@64.227.156.23:26656,6a3a0db2763d8222d00af55cbbe35824a39c8292@176.9.183.45:34656,6ae10267d8581414b37553655be22297b2f92087@174.138.25.159:26656,861152eda2fbab6555e8188088ea4dea9472a174@38.242.157.6:26656,a950d48fce4a648aacf7327198e6ea3e545f3112@168.119.166.138:26656,e097dc629cbe874b139841dedb06775cc75435ee@65.108.237.188:20656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.selfchain/config/config.toml
```
* Setup minimum GasFee
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uself\"/" $HOME/.selfchain/config/app.toml
```

* Setup pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.selfchain/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/selfchaind.service > /dev/null << EOF
[Unit]
Description=selfchain node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.selfchain"
Environment="DAEMON_NAME=selfchaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.selfchain/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
### Launch Node
```
sudo systemctl daemon-reload 
```
```
sudo systemctl enable selfchaind
```
```
sudo systemctl restart selfchaind
```
### Check log node
```
journalctl -fu selfchaind -o cat
```
### Wallet configuration
* add wallet
```
selfchaind keys add wallet
```
* recover wallet
```
selfchaind keys add wallet --recover
```
* list wallet
```
selfchaind keys list
```
* delete wallet
```
selfchaind keys delete wallet
```
* check balances
```
selfchaind q bank balances $(selfchaind keys show wallet -a)
```
### Validator Management
* create validator
```
selfchaind tx staking create-validator \
--amount="1000000000uself" \
--pubkey=$(selfchaind tendermint show-validator) \
--moniker=NodeName \
--identity="F57A71944DDA8C4B" \
--website="https://dwentz.xyz" \
--details=details node validator \
--chain-id=self-1 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000000 \
--broadcast-mode block \
--gas-adjustment=1.2 \
--gas-prices="0.5uself" \
--gas=auto \
--from=wallet
```
* edit validator
```
selfchaind tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=self-1 \
--gas-adjustment=1.2 \
--gas-prices="0.5uself" \
--gas=auto \
--from=wallet
```
* unjail validator
```
selfchaind tx slashing unjail --from wallet --chain-id self-1 --gas-prices 0.5uself --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
selfchaind query slashing signing-info $(selfchaind tendermint show-validator)
```
### Token management

* withdraw rewards
```
selfchaind tx distribution withdraw-all-rewards --from wallet --chain-id self-1  --gas-adjustment 1.2 --gas-prices 0.05uself --gas auto -y
```
* withdraw rewards with comission
```
selfchaind tx distribution withdraw-rewards $(selfchaind keys show wallet --bech val -a) --commission --from wallet --chain-id self-1  --gas-adjustment 1.2 --gas-prices 0.05uself --gas auto -y
```
* delegate token to your own validator
```
selfchaind tx staking delegate $(selfchaind keys show wallet --bech val -a) 1000000uself --from wallet --chain-id self-1  --gas-adjustment 1.2 --gas-prices 0.05uself --gas auto -y
```
### Node Info
* node id
```
selfchaind status 2>&1 | jq .NodeInfo
```
* validator info
```
selfchaind status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop selfchaind &&
sudo systemctl disable selfchaind &&
sudo rm /etc/systemd/system/selfchaind.service &&
sudo systemctl daemon-reload &&
rm -f $(which selfchaind) &&
rm -rf .selfchain &&
rm -rf selfchain
```
