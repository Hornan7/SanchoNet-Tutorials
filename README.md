# SanchoNet-Tutorials

## Stake pools

### Create a block producer on SanchoNet

#### 1. Create a source file for your node, database and socket.
```bash
mkdir sancho-src
cd sancho-src
mkdir db socket
```

#### 2. Grab the binary files of the last node version release. (Yes, because on SanchoNet, we test the latest)
```bash
wget https://github.com/IntersectMBO/cardano-node/releases/download/10.2.1/cardano-node-10.2.1-linux.tar.gz
```

#### 3. Extract it and move the binaries to /usr/local/bin so it can be used globaly
```bash
tar -xvf cardano-node-10.2.1-linux.tar.gz
cd bin
sudo mv cardano-node /usr/local/bin
sudo mv cardano-cli /usr/local/bin
```

#### 4. Create your startnode executable file.
```bash
cd ~
echo '#!/bin/bash

# Configuration variables
TOPOLOGY_FILE="${HOME}/sancho-src/share/sanchonet/topology.json"
CONFIG_FILE="${HOME}/sancho-src/share/sanchonet/config.json"
DATABASE_PATH="${HOME}/sancho-src/db"
SOCKET_PATH="${HOME}/sancho-src/socket/node.socket"
KES_PATH="keys/kes.skey"
VRF_PATH="keys/vrf.skey"
OPCERT_PATH="keys/opcert.cert"
HOST_ADDR="0.0.0.0"
PORT="6002"

cardano-node run \
    --topology "${TOPOLOGY_FILE}" \
    --database-path "${DATABASE_PATH}" \
    --socket-path "${SOCKET_PATH}" \
    --host-addr "${HOST_ADDR}" \
    --shelley-kes-key "${KES_PATH}"  \
    --shelley-vrf-key "${VRF_PATH}" \
    --shelley-operational-certificate "${OPCERT_PATH}" \
    --port "${PORT}" \
    --config "${CONFIG_FILE}" ' >> startnode.sh
sudo chmod 755 startnode.sh
```

#### 5. Create a linux service for your node
```bash
sudo echo "[Unit]
Description       = Cardano Node Service
Wants             = network-online.target
After             = network-online.target

[Service]
User=$(whoami)
Type=simple
WorkingDirectory=/home/$(whoami)
ExecStart=/home/$(whoami)/startnode.sh
KillSignal=SIGINT
RestartKillSignal=SIGINT
TimeoutStopSec=300
LimitNOFILE=32768
Restart=always
RestartSec=5
SyslogIdentifier=cardano-node

[Install]
WantedBy          = multi-user.target" > /etc/systemd/system/sancho-node.service
```
#### 6. Enable your Node linux service
```bash
sudo systemctl daemon-reload
sudo systemctl enable cardano-node.service
```
