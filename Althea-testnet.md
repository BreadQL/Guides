## Official Links
## Links
- [Website](http://althea.net)
- [Twitter](https://twitter.com/AltheaNetwork)

# Install Node Guide Althea
### Setting up variables

Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
ALTHEA_PORT=2027
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export ALTHEA_CHAIN_ID=althea_7357-1" >> $HOME/.bash_profile
echo "export ALTHEA_PORT=${ALTHEA_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony tar htop net-tools clang pkg-config libssl-dev ncdu -y
```

### Install GO
```bash
if ! [ -x "$(command -v go)" ]; then
  ver="1.19.5"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

### Download binaries
```bash
git clone https://github.com/althea-net/althea-chain
cd althea-chain
git checkout v0.3.2
make install
```

### Config app
```bash
althea config chain-id $ALTHEA_CHAIN_ID
althea config keyring-backend test
althea config node tcp://localhost:${ALTHEA_PORT}657
```

## Init app
```bash
althea init $NODENAME --chain-id $ALTHEA_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -O $HOME/.althea/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Althea/genesis.json"
wget -O $HOME/.althea/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Althea/addrbook.json"
```

## Set seeds and peers
```bash
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.althea/config/config.toml
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.althea/config/config.toml
peers="733e9d5f995c2866df9f2e1254551940f060a70c@51.159.159.112:26656,11e8f38e3c5601e4ab2333d5a5bbb108a39b8e1c@159.69.110.238:26656,a81cf8f7f330e2e09bec93c866214f7b3b336849@65.109.87.88:26356,83147260a704b75283ca6da218516ee0eaa82956@170.64.156.36:26656,617433cdf5411fc9241d0f77239f751a14669368@146.190.156.221:26656,856ac01afa0163c27b69e1b25464427310120924@85.25.134.23:26656,d320b861277a338daefec6e620daafe07fc5ee19@65.108.199.36:20036,8203297aacaea1d889fcf36240484c9efc217bbd@116.202.156.106:26656,c6e1ed7117cd56036cc51835945d155e9c474c01@167.235.144.3:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.althea/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.althea/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.althea/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.althea/config/config.toml
```

### Set minimum gas price
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0ualthea\"/;" ~/.althea/config/app.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ALTHEA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${ALTHEA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ALTHEA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ALTHEA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ALTHEA_PORT}660\"%" $HOME/.althea/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${ALTHEA_PORT}317\"%; s%^address = \":8080\"%address = \":${ALTHEA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${ALTHEA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${ALTHEA_PORT}091\"%" $HOME/.althea/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:2${ALTHEA_PORT}7\"%" $HOME/.althea/config/client.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.althea/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.althea/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.althea/config/config.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.althea/config/config.toml
```

### Reset chain data
```bash
althea tendermint unsafe-reset-all --home $HOME/.althea
```

### Create service
```bash
sudo tee /etc/systemd/system/althea.service > /dev/null <<EOF
[Unit]
Description=althea
After=network-online.target

[Service]
User=$USER
ExecStart=$(which althea) start --home $HOME/.althea
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Register and start service
```bash
sudo systemctl daemon-reload && 
sudo systemctl enable althea &&
sudo systemctl restart althea && sudo journalctl -u althea -f -o cat
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
althea keys add $WALLET
```

To recover existing keys use
```bash
althea keys add $WALLET --recover
```

List of wallets
```bash
althea keys list
```
### Save wallet info
Add wallet and valoper address into variables
```bash
ALTHEA_WALLET_ADDRESS=$(althea keys show $WALLET -a)
ALTHEA_VALOPER_ADDRESS=$(althea keys show $WALLET --bech val -a)
echo 'export ALTHEA_WALLET_ADDRESS='${ALTHEA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export ALTHEA_VALOPER_ADDRESS='${ALTHEA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### To check your wallet balance:
```bash
althea query bank balances $ALTHEA_WALLET_ADDRESS
```
### Create validator
After your node is synced, create validator
```bash
althea tx staking create-validator \
--amount=1000000000000000000ualthea \
--pubkey=$(althea tendermint show-validator) \
--moniker=$NODENAME \
--identity="" \
--website="" \
--details="" \
--chain-id=$ALTHEA_CHAIN_ID \
--commission-rate=0.1 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--gas=350000 \
--from=$WALLET -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu althea -o cat
```

Start service
```bash
sudo systemctl start althea
```

Stop service
```bash
sudo systemctl stop althea
```

Restart service
```bash
sudo systemctl restart althea
```

### Node info
Synchronization info
```bash
althea status 2>&1 | jq .SyncInfo
```

Validator info
```bash
althea status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
althea status 2>&1 | jq .NodeInfo
```

Show node id
```bash
althea tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
althea keys list
```

Recover wallet
```bash
althea keys add $WALLET --recover
```

Delete wallet
```bash
althea keys delete $WALLET
```

Get wallet balance
```bash
althea query bank balances $ALTHEA_WALLET_ADDRESS
```

Transfer funds
```bash
althea tx bank send $ALTHEA_WALLET_ADDRESS <TO_DEFUND_WALLET_ADDRESS> 1000000ualthea
```

### Voting
```bash
althea tx gov vote 1 yes --from $WALLET --chain-id=$ALTHEA_CHAIN_ID -y
```

### Staking, Delegation and Rewards
Delegate stake
```bash
althea tx staking delegate $ALTHEA_VALOPER_ADDRESS 10000000ualthea --from=$WALLET --chain-id=$ALTHEA_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```bash
althea tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000utlore --from=$WALLET --chain-id=$ALTHEA_CHAIN_ID --gas=auto
```

Withdraw all rewards
```bash
althea tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$ALTHEA_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```bash
althea tx distribution withdraw-rewards $ALTHEA_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$ALTHEA_CHAIN_ID
```

### Validator management
Edit validator
```bash
althea tx staking edit-validator \
  --moniker=$NODENAME \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$ALTHEA_CHAIN_ID \
  --from=$WALLET -y
```

Unjail validator
```bash
althea tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$ALTHEA_CHAIN_ID -y
```

### Delete node
```bash
sudo systemctl stop althea && \
sudo systemctl disable althea && \
rm /etc/systemd/system/althea.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
sudo rm -rf $HOME/.althea*
sudo rm -rf $HOME/althea-chain
rm -rf $(which althea)
sed -i '/ALTHEA_/d' ~/.bash_profile
```
