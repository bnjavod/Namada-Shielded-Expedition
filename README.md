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

#### Public RPC: 
```
https://rpc.namadase-cybernova.icu/
```

#### Indexer Service: 
```
https://indexer.namadase-cybernova.icu/block/last/
```

## Validator Setup

#### Recover key from registered account
```
namadaw derive --alias "${ALIAS}"
```

#### check key address
```
namadaw find --alias "${ALIAS}"
```

#### check balance well
```
namadac balance --owner "${ALIAS}"
```

#### find validator key
```
namadac find-validator --tm-address=$(curl -s localhost:26657/status | jq -r .result.validator_info.address)
```

#### initialize validator
```
namadac  init-validator \
 --alias "${VAL_NAME}" \
 --account-keys "${KEY_NAME}" \
 --signing-keys "${KEY_NAME}" \
 --commission-rate 0.1 \
 --max-commission-rate-change 0.1 \
 --email "${EMAIL}" \
```

#### bond tokens to your validator
```
namadac  bond \
 --source "${KEY_NAME}" \
 --validator "<Validator address>" \
 --amount 1000 \
```

## Validator management

#### validator consensus state
```
namadac validator-state --validator "<Validator address>"
```

#### unjail validator
```
namadac unjail-validator --validator  "<Validator address>"
```

#### claim rewards
```
namadac claim-rewards --validator "<Validator address>"
```

## Run node as Service
```
sudo tee /etc/systemd/system/namadad.service << EOF
[Unit]
Description=Namada Node
After=network.target
[Service]
User=$USER
WorkingDirectory=$HOME/.local/share/namada
Type=simple
ExecStart=/usr/local/bin/namada --base-dir=$HOME/.local/share/namada node ledger run
Environment=NAMADA_CMT_STDOUT=true
Environment=TM_LOG_LEVEL=p2p:none,pex:error
RemainAfterExit=no
Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
