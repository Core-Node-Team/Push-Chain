### 🚧 Snap
```
sudo systemctl stop pchaind
cp $HOME/.pchain/data/priv_validator_state.json $HOME/.pchain/priv_validator_state.json.backup
rm -rf $HOME/.pchain/data $HOME/state/wasm
curl https://snapshot.corenodehq.xyz/push/push_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.pchain
mv $HOME/.pchain/priv_validator_state.json.backup $HOME/.pchain/data/priv_validator_state.json
sudo systemctl restart pchaind && sudo journalctl -u pchaind -f
```
