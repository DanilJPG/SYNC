## kj nodes
#### Snapshot
```
curl -Ls https://snapshots.kjnodes.com/lava-testnet/genesis.json > $HOME/.lava/config/genesis.json

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ulava\"|" -e 's|^pruning *=.*|pruning = "custom"|' -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' -e 's|^pruning-interval *=.*|pruning-interval = "19"|' $HOME/.lava/config/app.toml

STATE_SYNC_RPC=https://lava-testnet.rpc.kjnodes.com:443
STATE_SYNC_PEER=d5519e378247dfb61dfe90652d1fe3e2b3005a5b@lava-testnet.rpc.kjnodes.com:44656
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 2000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i -e "s|^enable *=.*|enable = true|" -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" -e "s|^persistent_peers *=.*|persistent_peers = \"$STATE_SYNC_PEER\"|" $HOME/.lava/config/config.toml

sudo systemctl start lavad.service
sudo journalctl -f -u lavad.service -o cat
```
