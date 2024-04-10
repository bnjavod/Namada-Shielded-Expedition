# Operate Namada Infrastructure as a service

## Providing Services on Post-Genesis Node

#### Snapshot:
```
http://rpc.namadase-cybernova.icu:46656/snapshot-shielded-expedition.tar.lz4
```

#### Seed:
```
tcp://47bf9e314826d5d8ec29c90922cf6f1c734e82bd@37.60.238.210:26657
```

#### Peer:
```
tcp://47bf9e314826d5d8ec29c90922cf6f1c734e82bd@37.60.238.210:26657
```

#### Address Book
```
http://rpc.namadase-cybernova.icu:46656/addrbook.json
```

#### Public RPC: 
```
https://rpc.namadase-cybernova.icu/
```

#### Indexer Service: 
```
https://indexer.namadase-cybernova.icu/block/last
```

## Post-genesis Validator Setup

#### Recover key of registered SE account
```
namadaw derive --alias $WALLET --alias-force
```

#### Find key address
```
namadaw find --alias $WALLET
```

#### Check balance well
```
namadac balance --owner $WALLET
```

#### Find validator key
```
VALIDATOR_ADDR=$(namadac find-validator --tm-address=$(curl -s localhost:26657/status | jq -r .result.validator_info.address) | grep "Found validator address" | cut -d '"' -f 2)
echo $VALIDATOR_ADDR
```

#### Initialize post-genesis validator
```
namada client init-validator \
  --alias $VALIDATOR_ALIAS \
  --account-keys $WALLET \
  --signing-keys $WALLET \
  --commission-rate 0.05 \
  --max-commission-rate-change 0.01 \
  --email $EMAIL \
  --memo $WALLET_PK
```

#### Self-bond
```
namadac bond \
  --validator $VALIDATOR_ALIAS \
  --source $WALLET \
  --signing-keys $WALLET_PK \
  --amount $SELF_BOND_AMOUNT \
  --memo $WALLET_PK
```

## Validator Management

#### Check validator consensus state
```
namadac validator-state --validator $VALIDATOR_ADDR
```

#### Unjail validator
```
namadac unjail-validator --validator $VALIDATOR_ADDR
```

#### Claim rewards
```
namadac claim-rewards --validator $VALIDATOR_ADDR
```

#### Check node status
```
curl -s localhost:26657/status | jq .
```

#### Check validator bonded-stake
```
namadac bonded-stake --validator $VALIDATOR_ADDR
```

## Configure Node Service
```
sudo tee /usr/lib/systemd/user/namadase.service > /dev/null <<EOF
[Unit]
Description=Namada SE Daemon Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=always
RestartSec=30
WorkingDirectory=$HOME/.local/share/namada
Environment=NAMADA_CMT_STDOUT=true
Environment=TM_LOG_LEVEL=p2p:none,pex:error
ExecStart=/usr/local/bin/namada --base-dir=$HOME/.local/share/namada node ledger run

[Install]
WantedBy=default.target
EOF
```

## Launch Node service
```
sudo chmod 755 /usr/lib/systemd/user/namadase.service
systemctl --user daemon-reload
systemctl --user enable namadase
systemctl --user start namadase
journalctl --user-unit=namadase.service -f
```
