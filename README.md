

<h1 align="center"> Push Chain </h1>


<img width="1920" height="1415" alt="528046664-ccc04c2e-7190-470c-ab1c-5d7fe3e99f92" src="https://github.com/user-attachments/assets/462ca356-342b-4bb4-8b41-10428eaa0e53" />




> Unlock the Potential of Intent-Based, Secure Cross-Chain Interactions



 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>
 * [Push Chain Website](https://push.org/knowledge/)<br>
 * [Blockchain Explorer](https://explorer.corenodehq.xyz/Push-testnet)<br>
 * [Discord](https://discord.com/invite/pushchain)<br>
 * [Twitter](https://x.com/PushChain)<br>
 * [Faucet](https://faucet.push.org/)<br>


 ### Public RPC & Explorer

 https://push-testnet-api.corenodehq.xyz

 https://push-testnet-rpc.corenodehq.xyz

 https://push-testnet-evm.corenodehq.xyz

 https://explorer.corenodehq.xyz/Push-testnet

## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	6|
| RAM	| 16+ GB |
| Storage	| 400 GB SSD |

### 🚧Gerekli kurulumlar
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### 🚧 Go kurulumu
```
cd $HOME
VER="1.23.0"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### 🚧 Dosyaları çekelim ve kuralım
```
cd $HOME
mkdir -p $HOME/.pchain/cosmovisor/genesis/bin
rm -rf ~/bin
wget -O push.tar.gz https://github.com/pushchain/push-chain-node/releases/download/v0.0.38/push-chain_0.0.38_linux_amd64.tar.gz
tar -xvf push.tar.gz
chmod +x $HOME/bin/pchaind
mv $HOME/bin/pchaind $HOME/.pchain/cosmovisor/genesis/bin/pchaind
```
```
mkdir -p $HOME/.pchain/cosmovisor/upgrades/evm-v0-5-0/bin
rm -rf ~/bin
wget -O push.tar.gz https://github.com/pushchain/push-chain-node/releases/download/v0.0.39/push-chain_0.0.39_linux_amd64.tar.gz
tar -xvf push.tar.gz
chmod +x $HOME/bin/pchaind
mv $HOME/bin/pchaind $HOME/.pchain/cosmovisor/upgrades/evm-v0-5-0/bin/pchaind
```
```
sudo ln -s $HOME/.pchain/cosmovisor/genesis $HOME/.pchain/cosmovisor/current -f
sudo ln -s $HOME/.pchain/cosmovisor/current/bin/pchaind /usr/local/bin/pchaind -f
```
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```
### 🚧 Servis oluşturalım
```
sudo tee /etc/systemd/system/pchaind.service > /dev/null << EOF
[Unit]
Description=pchain node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.pchain"
Environment="DAEMON_NAME=pchaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.pchain/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable pchaind
```
### 🚧 İnit
```
echo "export PUSH_PORT="30"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
NOT: Node adınızı yazınız.
```
pchaind config set client chain-id push_42101-1
pchaind config set client keyring-backend os
pchaind config set client node tcp://localhost:${PUSH_PORT}657
pchaind init "Node-Adi" --chain-id push_42101-1
```
### 🚧 Genesis addrbook
```
wget -O $HOME/.pchain/config/genesis.json https://snapshot.corenodehq.xyz/push/genesis.json
wget -O $HOME/.pchain/config/addrbook.json  https://snapshot.corenodehq.xyz/push/addrbook.json
```
### 🚧 Peer
```
SEEDS="a8d3377ef5f091980a425b84380655865c0f2320@push-testnet-seed.itrocket.net:30656"
PEERS="1160dc307b84b47e430cc23a4cb266d4d767e233@push-testnet-peer.itrocket.net:30656,6531c80081c30afe3c4adb57c57721d16a3a405c@148.113.178.57:26656,d7fe39a89a2ab1d9a8207121c6a1f8e11f79ac97@34.9.151.27:26656,d2a1f2a83858d483ed05c5f094a65d1dd463de78@34.57.236.39:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.pchain/config/config.toml
```
### config pruning and Gas
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.pchain/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.pchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.pchain/config/app.toml


sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "1000000000upc"|g' $HOME/.pchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.pchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.pchain/config/config.toml
```
### 🚧 Port ayarı
```
sed -i.bak -e "s%:1317%:${PUSH_PORT}317%g;
s%:8080%:${PUSH_PORT}080%g;
s%:9090%:${PUSH_PORT}090%g;
s%:9091%:${PUSH_PORT}091%g;
s%:8545%:${PUSH_PORT}545%g;
s%:8546%:${PUSH_PORT}546%g;
s%:6065%:${PUSH_PORT}065%g" $HOME/.pchain/config/app.toml

sed -i.bak -e "s%:26658%:${PUSH_PORT}658%g;
s%:26657%:${PUSH_PORT}657%g;
s%:6060%:${PUSH_PORT}060%g;
s%:26656%:${PUSH_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${PUSH_PORT}656\"%;
s%:26660%:${PUSH_PORT}660%g" $HOME/.pchain/config/config.toml
```
```
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${PUSH_PORT}657\"|" $HOME/.pchain/config/client.toml
```
### 🚧 Snap
```
sudo systemctl stop pchaind
cp $HOME/.pchain/data/priv_validator_state.json $HOME/.pchain/priv_validator_state.json.backup
rm -rf $HOME/.pchain/data $HOME/state/wasm
curl https://snapshot.corenodehq.xyz/push/push_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.pchain
mv $HOME/.pchain/priv_validator_state.json.backup $HOME/.pchain/data/priv_validator_state.json
sudo systemctl restart pchaind && sudo journalctl -u pchaind -f
```
### 🚧 Başlatalım
```
sudo systemctl restart pchaind
journalctl -fu pchaind -o cat
```


### 🚧 Cüzdan olusturalım
```
pchaind keys add cüzdan-adi
```
### 🚧 Cüzdan import
```
pchaind keys add cüzdan-adi --recover
```
### 🚧 Validator Olusturma
```
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(pchaind comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000000000000000upc\",
    \"moniker\": \"nodeismin\",
    \"identity\": \"keybasecode\",
    \"website\": \"bura\",
    \"security\": \"bura\",
    \"details\": \"details\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
```
### OL :D
```
pchaind tx staking create-validator validator.json \
--from cüzdan-adi \
--chain-id push_42101-1 \
--gas auto --gas-adjustment 1.5 -y --fees 323982000000000upc
```


### Delege 
```
pchaind tx staking delegate valoper-adresi miktar000000000000000000upc \
--from cüzdan-adi \
--chain-id push_42101-1 \
--gas auto --gas-adjustment 1.5 -y --fees 323982000000000upc
```

### Komple Silme
```
sudo systemctl stop pchaind
sudo systemctl disable pchaind
sudo rm -rf /etc/systemd/system/pchaind.service
sudo rm $(which pchaind)
sudo rm -rf $HOME/.pchain
sed -i "/PUSH_/d" $HOME/.bash_profile
```
