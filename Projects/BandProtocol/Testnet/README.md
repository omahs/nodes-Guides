# BAND TESTNET guide

![band](https://github.com/obajay/nodes-Guides/assets/44331529/b42abd97-f3d7-42e0-ba0f-f4ddeb44f5cb)


[WebSite](https://www.bandprotocol.com/)\
[GitHub](https://github.com/bandprotocol/chain)
=
[EXPLORER](https://explorer.stavr.tech/Band-Testnet/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   4| 8GB  | 200GB    |


# 1) Auto_install script
```python
wget -O bandt https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/BandProtocol/Testnet/bandt && chmod +x bandt && ./bandt
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## GO 1.19
```python
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Build 14.09.23
```python
cd $HOME
git clone https://github.com/bandprotocol/chain band
cd band
git checkout v2.5.4
make install


```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`bandd version --long | head`
- version: 2.5.4
- commit: e6548bbf4793829bb8e711e5ed89ba4afc710ded

```python
bandd init STAVR_guide --chain-id band-laozi-testnet6
bandd config chain-id band-laozi-testnet6
```    

## Create/recover wallet
```python
bandd keys add <walletname>
  OR
bandd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.band/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/BandProtocol/Testnet/genesis.json"
```
`sha256sum $HOME/.band/config/genesis.json`
+ c776e2a5fac6ce3214845db52da6beef95fada49717eac55b0b81ad3767db7de

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025uband\"/;" ~/.band/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.band/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.band/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.band/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 10/g' $HOME/.band/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 40/g' $HOME/.band/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.band/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.band/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.band/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.band/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.band/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.band/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/BandProtocol/Testnet/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/bandd.service > /dev/null <<EOF
[Unit]
Description=band test
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bandd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Band Testnet
```python
RPC=https://band.rpc.t.stavr.tech:443
peers=7f03c7f4a41300348afce4b51774ab3fab8ae3c2@band-t.peer.stavr.tech:11016
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.band/config/config.toml
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.band/config/config.toml
sudo systemctl stop bandd && bandd tendermint unsafe-reset-all --keep-addr-book
curl -o - -L http://band-t.files.stavr.tech:1103/files-bandt.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.band --strip-components 2
sudo systemctl restart bandd && journalctl -u bandd -f -o cat
```
# SnapShot Testnet (~2GB) updated every 12 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop bandd
cp $HOME/.band/data/priv_validator_state.json $HOME/.band/priv_validator_state.json.backup
rm -rf $HOME/.band/data
curl -o - -L http://band-t.snapshot.stavr.tech:1025/bandt/bandt-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.band --strip-components 2
curl -o - -L http://band-t.files.stavr.tech:1103/files-bandt.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.band --strip-components 2
mv $HOME/.band/priv_validator_state.json.backup $HOME/.band/data/priv_validator_state.json
wget -O $HOME/.band/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/BandProtocol/Testnet/addrbook.json"
sudo systemctl restart bandd && journalctl -u bandd -f -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable bandd
sudo systemctl restart bandd && sudo journalctl -u bandd -f -o cat
```

### Create validator
```python
bandd tx staking create-validator \
  --amount=1000000uband \
  --pubkey=$(bandd tendermint show-validator) \
  --moniker="STAVR_Guide" \
  --identity="" \
  --details="" \
  --website="" \
  --chain-id="band-laozi-testnet6" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.2" \
  --min-self-delegation="1" \
  --fees 500uband \
  --from=YourWallet -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/BandProtocol)
=

## Delete node
```python
sudo systemctl stop bandd
sudo systemctl disable bandd
rm /etc/systemd/system/bandd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf band
rm -rf .band
rm -rf $(which bandd)
```
#
### Sync Info
```python
bandd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
bandd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u bandd -f -o cat
```
### Check Balance
```python
bandd query bank balances bandd...addressjkl1yjgn7z09ua9vms259j
```
