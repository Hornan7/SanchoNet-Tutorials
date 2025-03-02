---
Title: SanchoNet - Geek Speak Made Fun
Status: In progress
Authors:
    - Mike Hornan <mike.hornan@able-pool.io>
Usefull links:
    - <https://github.com/IntersectMBO/credential-manager>
    - <https://github.com/IntersectMBO/cardano-node/releases>
    - <https://github.com/cardano-community/guild-operators>
    - <https://github.com/gitmachtl/cardano-signer>
    - <https://github.com/Cardano-Atlantic-Council/Scripts>
Created: 2025-02-28
License: CC-BY-4.0
---
# SanchoNet Tutorial

## Table of Contents

+ [Initial Environment Configuration](#initial-environment-configuration)
+ [Sancho Wallet](#sancho-wallet)
  - [Generate a wallet from the CLI](#generate-a-wallet-from-the-cli)
  - [Generate a wallet from a mnemonic phrase](#generate-a-wallet-from-a-mnemonic-phrase)
  - [Restore a wallet from a mnemonic phrase](#restore-a-wallet-from-a-mnemonic-phrase)
  - [Get SanchoBucks from the Mike or the King](#get-sanchobucks-from-mike-or-the-king)
+ [Stake Pools](#stake-pools)
  - [Create a block producer node](#create-a-block-producer-node)
  - [Create a relay node](#create-a-relay-node)
+ [Delegated Representative](#delegated-representative)
  - [Create a DRep and register it](#create-a-drep-and-register-it)
  - [Create a multi-signature DRep and register it](#create-a-multi-signature-drep-and-register-it)
+ [Constitutional Committee](#constitutional-committee)
  - [Download and install Nix](#download-and-install-nix)
  - [Install the Credential Manager tools](#install-the-credential-manager-tools)
  - [Generate Cardano keys and Openssl certificate signing request](#generate-cardano-keys-and-openssl-certificate-signing-request)
  - [The Head of security role and Certificate authority](#the-head-of-security-role-and-certificate-authority)
  - [Mint the cold credential NFT](#mint-the-cold-credential-nft)
  - [Mint the hot credential NFT](#mint-the-hot-credential-nft)
  - [Authorize the Constitutional Committee hot credential](#authorize-the-constitutional-committee-hot-credential)
  - [Vote on governance actions as a consortium](#vote-on-governance-actions-as-a-consortium)
+ [Build a governance action](#build-a-governance-action)
  - [Motion of no-confidence](#motion-of-no-confidence)
  - [Update Committee and/or threshold](#update-committee-and/or-threshold)
  - [New Constitution or Guardrail Scripts](#new-constitution-or-guardrail-scripts)
  - [Hard-Fork Initiation](#hard-fork-initiation)
  - [Protocol parameter changes](#protocol-parameter-changes)
  - [Treasury withdrawal](#treasury-withdrawal)
  - [Info action](#info-action)
+ [Usefull Scripts](#usefull-scripts)

## Initial Environment Configuration

#### 1. Create a file for your node, database, socket and keys.
```bash
mkdir ~/sancho-src ~/keys ~/sancho-src/db ~/sancho-src/socket
cd sancho-src
```

#### 2. Grab the binary files of the last node version release. (Yes, because on SanchoNet, we test the latest)
```bash
wget https://github.com/IntersectMBO/cardano-node/releases/download/10.2.1/cardano-node-10.2.1-linux.tar.gz
```

#### 3. Extract them and move the binaries to /usr/local/bin so they can be used globaly
```bash
tar -xvf cardano-node-10.2.1-linux.tar.gz
rm cardano-node-10.2.1-linux.tar.gz
cd bin
sudo mv cardano-node /usr/local/bin
sudo mv cardano-cli /usr/local/bin
```

#### 4. Add SanchoNet Node Socket Path and Network ID to Your .bashrc File
This should allow you to write cardano-cli commands and queries without having to specify the socket path and network ID options everytime.
```bash
echo 'export CARDANO_NODE_SOCKET_PATH=${HOME}/sancho-src/socket/node.socket' >> ~/.bashrc
echo 'export CARDANO_NODE_NETWORK_ID=4' >> ~/.bashrc
source ~/.bashrc
```

# Sancho Wallet

## Generate a wallet from the CLI

#### 1. Create your payment key pairs
```bash
cardano-cli conway address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

#### 2. Create your stake key pairs
```bash
cardano-cli conway stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey
```

#### 3. Build your wallet address
```bash
cardano-cli conway address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file payment.addr
```

#### 4. Build your stake address
```bash
cardano-cli conway stake-address build \
--stake-verification-key-file stake.vkey \
--out-file stake.addr
```
#### 5. You are now ready to get money from [Mike](#get-sanchobucks-from-mike-or-the-king) or our beloved [King](#get-sanchobucks-from-mike-or-the-king)

## Generate a wallet from a mnemonic phrase

#### 1. Get the cardano-signer script from Martin Lang(ATADA)'s github repository.
*Martin Lang is a very well known Cardano Developer and a great Stake pool operator. So shout out to him for his great scripts.*
The following command will get the `cardano-signer` binary file, extract it and move it to `/usr/local/bin` so you can use it globaly:
```bash
cd ~/sancho-src
wget https://github.com/gitmachtl/cardano-signer/releases/download/v1.23.0/cardano-signer-1.23.0_linux-x64.tar.gz
tar -xvf https://github.com/gitmachtl/cardano-signer/releases/download/v1.23.0/cardano-signer-1.23.0_linux-x64.tar.gz
sudo mv cardano-signer /usr/local/bin
```

#### 2. Generate payment key pairs and the secret.json file
```bash
cardano-signer keygen \
--path payment \
--json-extended \
--out-skey payment.xskey \
--out-vkey payment.vkey \
--out-file secret.json
```

#### 3. Write down your seed phrase and keep it safe.
*While we don't prioritize security for SanchoNet, it's definitely worth protecting it for Mainnet use. (Yes that involve the use of a cold environment to generate these)*
```bash
cat secret.json | jq .mnemonics
```

#### 4. Generate stake key pairs from your mnemonic phrase
```bash
cardano-signer keygen \
--path stake \
--mnemonics "$(cat secret.json | jq -r .mnemonics)"  \
--json-extended \
--out-skey stake.xskey \
--out-vkey stake.vkey | jq '.'
```
Make sure the mnemonic phrase in the output of this command matches the one you generated for your payment keys.

#### 5. Build your wallet address
```bash
cardano-cli conway address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file payment.addr
```

#### 6. Build your stake address
```bash
cardano-cli conway stake-address build \
--stake-verification-key-file stake.vkey \
--out-file stake.addr
```

#### 7. You are now ready to get money from [Mike](#get-sanchobucks-from-mike-or-the-king) or our beloved [King](#get-sanchobucks-from-mike-or-the-king)

## Restore a wallet from a mnemonic phrase

#### 1. Get the cardano-signer script from Martin Lang(ATADA)'s github repository. (If you don't already have it)
The following command will get the `cardano-signer` binary file, extract it and move it to `/usr/local/bin` so you can use it globaly:
```bash
cd ~/sancho-src
wget https://github.com/gitmachtl/cardano-signer/releases/download/v1.23.0/cardano-signer-1.23.0_linux-x64.tar.gz
tar -xvf https://github.com/gitmachtl/cardano-signer/releases/download/v1.23.0/cardano-signer-1.23.0_linux-x64.tar.gz
sudo mv cardano-signer /usr/local/bin
``` 

#### 2. Restore the payment key pairs and the secret.json file
```bash
cardano-signer keygen \
--path payment \
--mnemonics "WRITE DOWN YOUR SEED PHRASE HERE"  \
--json-extended \
--out-skey payment.xskey \
--out-vkey payment.vkey \
--out-file secret.json
```

#### 3. Double-check the contents of your `secret.json` file to ensure your payment keys have been properly restored.
```bash
cat secret.json | jq '.'
```

#### 4. Restore the stake key pairs from your mnemonic phrase
```bash
cardano-signer keygen \
--path stake \
--mnemonics "$(cat secret.json | jq -r .mnemonics)"  \
--json-extended \
--out-skey stake.xskey \
--out-vkey stake.vkey | jq '.'
```

#### 5. Restore your wallet address
```bash
cardano-cli conway address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file payment.addr
```

#### 6. Restore your stake address
```bash
cardano-cli conway stake-address build \
--stake-verification-key-file stake.vkey \
--out-file stake.addr
```

#### 7. Query the UTXO of Your Payment Address to Verify If Your Wallet Balance Is Restored
```bash
cardano-cli conway query utxo \
--address $(cat payment.addr)
```

## Get SanchoBucks from Mike or the King
Now when you are finally ready to get some SanchoBucks to build on SanchoNet, you can ask our beloved King [Big Joe the Don](https://x.com/bigjoethedon) or [Mike Hornan](https://x.com/Hornan7) directly.
It is highly recommended to join the [ABLE pool Discord](https://discord.gg/tHYrxCtdHm) to hang out with us, or if you want to test or possibly break something. You might be surprised by how willing we are to test anything that could potentially damage the chain.
Then Mike will send SanchoBucks directly to your wallet address. (Yes, he always answers his DMs.)

# Stake pools

## Create a block producer node

#### 1. Create your startnode executable file.
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

#### 2. Create a linux service for your node
```bash
sudo cat > sancho-node.service << EOF
[Unit]
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
WantedBy          = multi-user.target
EOF
sudo mv sancho-node.service /etc/systemd/system
```
#### 3. Enable your Node linux service
```bash
sudo systemctl daemon-reload
sudo systemctl enable sancho-node.service
```

#### 4. Create your pool keys
```bash
cd ~/keys
cardano-cli conway node key-gen \
--cold-verification-key-file cold.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter-file opcert.counter
sudo chmod 400 cold.skey cold.vkey
```

#### 5. Create the KES keys
```bash
cardano-cli conway node key-gen-KES \
--verification-key-file kes.vkey \
--signing-key-file kes.skey
sudo chmod 400 kes.skey kes.vkey
```

#### 6. Create VRF keys
```bash
cardano-cli conway node key-gen-VRF \
--verification-key-file vrf.vkey \
--signing-key-file vrf.skey
sudo chmod 400 vrf.skey vrf.vkey
```

#### 7. Modify your topology file
```
cd ~/sancho-src/share/sanchonet
rm topology.json
echo '{
  "bootstrapPeers": [
    {
      "address": "sancho-testnet.able-pool.io",
      "port": 6002
    }
  ],
  "localRoots": [
    {
      "accessPoints": [],
      "advertise": false,
      "trustable": false,
      "valency": 1
    }
  ],
  "publicRoots": [
    {
      "accessPoints": [],
      "advertise": false
    }
  ],
  "useLedgerAfterSlot": 33695977
}' > topology.json
```

#### 8. Create the operational cert
```bash
kesPeriod=416
cardano-cli conway node issue-op-cert \
--kes-verification-key-file ~/keys/kes.vkey \
--cold-signing-key-file ~/keys/cold.skey \
--operational-certificate-issue-counter-file ~/keys/opcert.counter \
--kes-period ${kesPeriod} \
--out-file ~/keys/opcert.cert
sudo chmod 400 ~/keys/opcert.cert
```
