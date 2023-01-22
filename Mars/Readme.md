## 22.01.2023

## STAVR
#### State Symc
```
systemctl stop marsd
SNAP_RPC=http://mars.rpc.t.stavr.tech:190

peers="b42f07453d051f65978c22b8047feb9d2e634aff@mars.peer.stavr.tech:181"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.mars/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.mars/config/config.toml
marsd tendermint unsafe-reset-all --home /root/.mars --keep-addr-book
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"1500\"/" $HOME/.mars/config/app.toml
curl -o - -L http://mars.wasm.stavr.tech:1014/wasm-mars.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mars/data --strip-components 3

systemctl restart marsd && journalctl -u marsd -f -o cat
```

#### Snapshot
```
cd $HOME
apt install lz4

sudo systemctl stop marsd
cp $HOME/.mars/data/priv_validator_state.json $HOME/.mars/priv_validator_state.json.backup
rm -rf $HOME/.mars/data

curl -o - -L http://mars.snapshot.stavr.tech:1012/mars/mars-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mars --strip-components 2
curl -o - -L http://mars.wasm.stavr.tech:1014/wasm-mars.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.mars/data --strip-components 3
mv $HOME/.mars/priv_validator_state.json.backup $HOME/.mars/data/priv_validator_state.json
wget -O $HOME/.mars/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Mars/addrbook.json"

sudo systemctl restart marsd && journalctl -u marsd -f -o cat
```


