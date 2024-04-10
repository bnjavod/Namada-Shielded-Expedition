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

## Post-genesis Validator Setup

#### Recover key from registered account
```
namadaw derive --alias $WALLET --alias-force
```

#### find key address
```
namadaw find --alias $WALLET
```

#### check balance well
```
namadac balance --owner $WALLET
```

#### find validator key
```
VALIDATOR_ADDR=$(namadac find-validator --tm-address=$(curl -s localhost:26657/status | jq -r .result.validator_info.address) | grep "Found validator address" | cut -d '"' -f 2)
echo $VALIDATOR_ADDR
```

#### initialize validator
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

#### bond tokens to your validator
```
namadac bond \
  --validator $VALIDATOR_ALIAS \
  --source $WALLET \
  --signing-keys $WALLET_PK \
  --amount $SELF_BOND_AMOUNT \
  --memo $WALLET_PK
```

## Validator management

#### validator consensus state
```
namadac validator-state --validator $VALIDATOR_ADDR
```

#### unjail validator
```
namadac unjail-validator --validator $VALIDATOR_ADDR
```

#### claim rewards
```
namadac claim-rewards --validator $VALIDATOR_ADDR
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
