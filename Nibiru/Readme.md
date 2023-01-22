## 22.01.2023

## NodersTEAM
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
