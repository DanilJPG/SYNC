## 22.01.2023

## [Noders]TEAM
#### State Sync
```
sudo systemctl stop nibid

peers="2ec6cb2a83c178fb490a992a3bd6a5c142c3fc61@nibiru.statesync.nodersteam.com:26656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.nibid/config/config.toml

SNAP_RPC=http://nibiru.statesync.nodersteam.com:26657

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.nibid/config/config.toml

sudo systemctl stop nibid
nibid tendermint unsafe-reset-all --home /root/.nibid --keep-addr-book

sudo systemctl restart nibid && journalctl -u nibid -f -o cat

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.nibid/config/config.toml
sudo systemctl restart nibid && journalctl -u nibid -f -o cat
```

## kjnodes
#### Snaphot
```
sudo systemctl stop nibid
cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup
rm -rf $HOME/.nibid/data

curl -L https://snapshots.kjnodes.com/nibiru-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.nibid
mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json

sudo systemctl start nibid && sudo journalctl -u nibid -f --no-hostname -o cat
```

## Nodeist
#### State Sync
```
systemctl stop nibid
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book

SNAP_RPC="https://rpc-nibiru.nodeist.net:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.nibid/config/config.toml

systemctl restart nibid && journalctl -fu nibid -o cat
```

#### Snapshot
```
sudo apt update
sudo apt install snapd -y
sudo snap install lz4

sudo systemctl stop nibid

nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book

curl -L https://snap.nodeist.net/t/nibiru/nibiru.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nibid --strip-components 2

sudo systemctl start nibid && journalctl -u nibid -f --no-hostname -o cat
```
