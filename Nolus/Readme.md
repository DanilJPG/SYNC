## PPNV SPACE
#### State Sync
```
sudo systemctl stop nolusd

cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
nolusd tendermint unsafe-reset-all --home $HOME/.nolus

curl -s https://snapshots4-testnet.nodejumper.io/nolus-testnet/wasm.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nolus

STATE_SYNC_RPC=http://rpc.nolus.ppnv.space:34657
STATE_SYNC_PEER=1a0bb6c35e2663202535d4b849ff06250762d299@rpc.nolus.ppnv.space:35656
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 1000))

SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -e "s|^enable *=.*|enable = true|" $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" \
 $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" \
 $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" \
 $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^persistent_peers *=.*|persistent_peers = \"$STATE_SYNC_PEER\"|" \
 $HOME/.nolus/config/config.toml
 
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
sudo systemctl restart nolusd && journalctl -u nolusd -f --no-hostname -o cat
```


## Nodes Jumper
#### State Sync
```
sudo systemctl stop nolusd

cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book

SNAP_RPC="https://nolus-testnet.nodejumper.io:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="5c236704215735ea722a3ca742a5161c2e871ec6@nolus-testnet.nodejumper.io:29656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.nolus/config/config.toml

sed -i 's|^enable *=.*|enable = true|' $HOME/.nolus/config/config.toml
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $HOME/.nolus/config/config.toml
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $HOME/.nolus/config/config.toml
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $HOME/.nolus/config/config.toml

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json

curl -s https://snapshots4-testnet.nodejumper.io/nolus-testnet/wasm.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nolus

sudo systemctl restart nolusd
sudo journalctl -u nolusd -f --no-hostname -o cat
```

#### Snapshot 
```
sudo apt update
sudo apt install lz4 -y
sudo systemctl stop nolusd
```
```
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book

SNAP_NAME=$(curl -s https://snapshots4-testnet.nodejumper.io/nolus-testnet/ | egrep -o ">nolus-rila.*\.tar.lz4" | tr -d ">")
curl https://snapshots4-testnet.nodejumper.io/nolus-testnet/${SNAP_NAME} | lz4 -dc - | tar -xf - -C $HOME/.nolus

mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json

sudo systemctl restart nolusd
sudo journalctl -u nolusd -f --no-hostname -o cat
```

## Nodeist
#### State Sync
```
systemctl stop nolusd

nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book
SNAP_RPC="https://rpc-nolus.nodeist.net:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.nolus/config/config.toml

rm -r ~/.nolus/data/wasm
wget -O wasmonly.tar.lz4 https://snapshots.nodeist.net/t/nolus/wasmonly.tar.lz4 --inet4-only
lz4 -c -d wasmonly.tar.lz4  | tar -x -C $HOME/.nolus/data
rm wasmonly.tar.lz4

systemctl restart nolusd && journalctl -fu nolusd -o cat
```
#### Snapshot
```
sudo apt update
sudo apt install snapd -y
sudo snap install lz4
```

```
sudo systemctl stop nolusd
```

```
nolusd tendermint unsafe-reset-all --home $HOME/.nolus --keep-addr-book
curl -L https://snap.nodeist.net/t/nolus/nolus.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nolus --strip-components 2
```
```
sudo systemctl start nolusd && journalctl -u nolusd -f --no-hostname -o cat
```
