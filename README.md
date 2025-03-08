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
  - [Register your stake address](#register-your-stake-address)
  - [Delegate to a stake pool](#delegate-to-a-stake-pool)
  - [Delegate to a DRep](#delegate-to-a-drep)
+ [Stake Pools](#stake-pools)
  - [Create a block producer node](#create-a-block-producer-node)
  - [Register your stake pool](#register-your-stake-pool)
  - [Create a relay node](#create-a-relay-node)
+ [Delegated Representative](#delegated-representative)
  - [Create a DRep and register it](#create-a-drep-and-register-it)
  - [Create a multi-signature DRep and register it](#create-a-multi-signature-drep-and-register-it)
+ [Constitutional Committee Consortium](#constitutional-committee-consortium)
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

## Register your stake address

#### 1. Create the registration certificate
```bash
cardano-cli conway stake-address registration-certificate \
--stake-verification-key-file stake.vkey \
--key-reg-deposit-amt 2000000 \
--out-file registration.cert
```

#### 2. Build the transaction to submit the certificate on-chain
```bash
cardano-cli conway transaction build \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--certificate-file registration.cert \
--out-file tx.raw
```

#### 3. Sign the transaction body file
```bash
cardano-cli conway transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--out-file tx.signed
```

#### 4. Submit the transaction on-chain
```bash
cardano-cli conway transaction submit \
--tx-file tx.signed
```

## Delegate to a stake pool

#### 1. Create a stake delegation certificate
```bash
cardano-cli conway stake-address stake-delegation-certificate \
--stake-verification-key-file stake.vkey \
--stake-pool-id "THE ID OF THE STAKE POOL YOU ARE DELEGATING TO" \
--out-file delegation.cert
```

#### 2. Build the transaction to submit the certificate on-chain
```bash
cardano-cli conway transaction build \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--certificate-file delegation.cert \
--out-file tx.raw
```

#### 3. Sign the transaction body file
```bash
cardano-cli conway transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--out-file tx.signed
```

#### 4. Submit the transaction on-chain
```bash
cardano-cli conway transaction submit \
--tx-file tx.signed
```

## Delegate to a DRep

#### 1. Create a vote delegation certificate
Note that when delegating the voting power of your wallet, you can opt to delegate to `--always-abstain` or `--always-no-confidence`. However, in the example below, we will delegate to a specific DRep ID.
```bash
cardano-cli conway stake-address vote-delegation-certificate \
  --stake-verification-key-file stake.vkey \
  --drep-key-hash "PUT YOUR DREP ID HERE" \
  --out-file vote-deleg.cert
```

#### 2. Build the transaction to submit the certificate on-chain
```bash
cardano-cli conway transaction build \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--certificate-file vote-deleg.cert \
--out-file tx.raw
```

#### 3. Sign the transaction body file
```bash
cardano-cli conway transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--out-file tx.signed
```

#### 4. Submit the transaction on-chain
```bash
cardano-cli conway transaction submit \
--tx-file tx.signed
```

# Stake pools

## Create a block producer node

#### 1. Create the startnode.sh executable
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

#### 2. Set up a Linux service for your node.
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
#### 3. Activate your Node Linux service.
```bash
sudo systemctl daemon-reload
sudo systemctl enable sancho-node.service
```

#### 4. Generate your pool key pairs and operational certificate counter
```bash
cd ~/keys
cardano-cli conway node key-gen \
--cold-verification-key-file cold.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter-file opcert.counter
sudo chmod 400 cold.skey cold.vkey
```

#### 5. Generate the KES key pairs
```bash
cardano-cli conway node key-gen-KES \
--verification-key-file kes.vkey \
--signing-key-file kes.skey
sudo chmod 400 kes.skey kes.vkey
```

#### 6. Generate the VRF key pairs
```bash
cardano-cli conway node key-gen-VRF \
--verification-key-file vrf.vkey \
--signing-key-file vrf.skey
sudo chmod 400 vrf.skey vrf.vkey
```

#### 7. Update your topology file with the correct bootstrap peer.
```
cd ~/sancho-src/share/sanchonet
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

#### 8. Generate the operational certificate.
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

#### 9. Its now ready to run
```bash
sudo systemctl start sancho-node.service
```

## Register your stake pool

#### 1. Create you pool metadata
Create your pool metadata following the example below, and upload it to a URL that you control (e.g., your GitHub repository).
```json
{
  "name": "Able Stake Pool",
  "description": "Able Pool On Sancho Testnet",
  "ticker": "ABLE",
  "homepage": "https://able-pool.io"
}
```

#### 2. Get the hash of your metadata file
```bash
cardano-cli hash anchor-data \
--url <THE URL LINK TO YOUR METADATA>
```

#### 3. Create the pool registration certificate
```bash
cardano-cli conway stake-pool registration-certificate \
--cold-verification-key-file cold.vkey \
--vrf-verification-key-file vrf.vkey \
--pool-pledge 9000000000 \
--pool-cost 340000000 \
--pool-margin 0.05 \
--pool-reward-account-verification-key-file stake.vkey \
--pool-owner-stake-verification-key-file stake.vkey \
--pool-relay-ipv4 <RELAY NODE PUBLIC IP> \
--pool-relay-port <RELAY NODE PORT> \
--metadata-url <POOL METADATA URL LINK> \
--metadata-hash <THE HASH OF YOUR METADATA FILE> \
--out-file pool-registration.cert
```

#### 4. Build the transaction to submit the certificate on-chain
```bash
cardano-cli conway transaction build \
--witness-override 3 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--certificate-file pool-registration.cert \
--out-file tx.raw
```

#### 5. Sign the transaction body file
```bash
cardano-cli conway transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--signing-key-file cold.skey \
--out-file tx.signed
```

#### 6. Submit the transaction on-chain
```bash
cardano-cli conway transaction submit \
--tx-file tx.signed
```

#### 7. Get your stake pool ID (bech32-encoded)
```bash
cardano-cli conway stake-pool id \ \
--cold-verification-key-file cold.vkey \
--output-format bech32 \
--out-file pool.id
```

#### 8. Verify that your stake pool is registered
```bash
cardano-cli conway query pool-params \
--stake-pool-id $(cat pool.id)
```

## Create a relay node

#### 1. Create the startnode.sh executable
```bash
cd ~
echo '#!/bin/bash

# Configuration variables
TOPOLOGY_FILE="${HOME}/sancho-src/share/sanchonet/topology.json"
CONFIG_FILE="${HOME}/sancho-src/share/sanchonet/config.json"
DATABASE_PATH="${HOME}/sancho-src/db"
SOCKET_PATH="${HOME}/sancho-src/socket/node.socket"
HOST_ADDR="0.0.0.0"
PORT="6002"

cardano-node run \
    --topology "${TOPOLOGY_FILE}" \
    --database-path "${DATABASE_PATH}" \
    --socket-path "${SOCKET_PATH}" \
    --host-addr "${HOST_ADDR}" \
    --port "${PORT}" \
    --config "${CONFIG_FILE}" ' >> startnode.sh
sudo chmod 755 startnode.sh
```

#### 2. Set up a Linux service for your node.
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

#### 3. Activate your Node Linux service.
```bash
sudo systemctl daemon-reload
sudo systemctl enable sancho-node.service
```

#### 4. Update your topology file with the correct bootstrap peer.
```
cd ~/sancho-src/share/sanchonet
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

#### 5. Its now ready to run
```bash
sudo systemctl start sancho-node.service
```

# Delegated Representative

## Create a DRep and register it

#### 1. Generate a DRep key pair
```bash
cardano-cli conway governance drep key-gen \
--verification-key-file drep.vkey \
--signing-key-file drep.skey
```

#### 2. Generate a DRep ID
Note that in the example below, we are generating a bech32-encoded ID. However, if you want to validate a vote in the future by querying the governance state of the ledger, your DRep ID will be displayed in hex format. 
You can always obtain your hex-encoded ID by setting the output option to `--output-hex`.
```bash
cardano-cli conway governance drep id \
--drep-verification-key-file drep.vkey \
--output-bech32 \
--out-file drep.id
```

#### 3. Create a DRep registration certificate
```bash
cardano-cli conway governance drep registration-certificate \
--drep-verification-key-file drep.vkey \
--key-reg-deposit-amt $(cardano-cli conway query gov-state | jq -r .currentPParams.dRepDeposit) \
--out-file drep-register.cert
```

#### 4. Build the transaction to submit the certificate on-chain
```bash
cardano-cli conway transaction build \
--witness-override 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--certificate-file drep-register.cert \
--out-file tx.raw
```

#### 5. Sign the trasaction body file
```bash
cardano-cli conway transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file drep.skey \
--out-file tx.signed
```

#### 6. Submit the transaction on-chain
```bash
cardano-cli conway transaction submit \
--tx-file tx.signed
```

## Create a multi-signature DRep and register it

#### 1. Generate DRep Key Pairs for Each Member of Your Multi-Sig Setup
```bash
cardano-cli conway governance drep key-gen \
--verification-key-file drep.vkey \
--signing-key-file drep.skey
```

#### 2. Obtain the DRep verification key hash for each member's verification key.
```bash
cardano-cli conway governance drep id \
--drep-verification-key-file drep.vkey \
--output-format hex \
--out-file name-drep.hash
```

#### 3. Build the multi-signature script with each member's hash
Note that you can replace the `atLeast` value with `all` and remove the `required` key-value pair if you want to require all signatures to vote on a governance action.
Name the file `drep-script.json` and save it.
```json
{
  "type": "atLeast",
  "required": 2,
  "scripts": [
    {
      "type": "sig",
      "keyHash": "PUT MEMBER'S VERIFICATION KEY HASH HERE"
    },
    {
      "type": "sig",
      "keyHash": "5ab00e8cd1142fcffc5f7a2c2e3549874afd89e26995d7686c2714d4"
    },
    {
      "type": "sig",
      "keyHash": "db5a8cbb0df0359c36541727229993b21371f834202733c9bbabc1fd"
    }
  ]
}
```

#### 4. Generate the Multi-Sig DRep Script Hash as Your ID
```bash
cardano-cli hash script \
--script-file drep-script.json \
--out-file drep-script.hash
```

#### 5. Generate the DRep registration certificate
```bash
cardano-cli conway governance drep registration-certificate \
--drep-script-hash "$(cat drep-script.hash)" \
--key-reg-deposit-amt $(cardano-cli conway query gov-state | jq -r .currentPParams.dRepDeposit) \
--out-file drep-multisig-reg.cert
```

#### 6. Build the transaction to submit the certificate on-chain
Note that the `--witness-override` value should be the number of members plus one, which accounts for both the members' signatures and the payment wallet signature for the transaction.
```bash
cardano-cli conway transaction build \
--tx-in $(cardano-cli conway query utxo --address $(cat payment.addr) --output-json | jq -r 'keys[0]') \
--change-address $(cat payment.addr) \
--witness-override 4 \
--certificate-file drep-multisig-reg.cert \
--certificate-script-file drep-script.json \
--out-file tx.raw
```

#### 7. Witness the transaction body file
Share the transaction body file `tx.raw` with all members so they can sign it using the following command:
```bash
cardano-cli conway transaction witness \
--tx-body-file tx.raw \
--signing-key-file drep.skey \
--out-file name-drep.witness
```

#### 8. As the orchestrator of the transaction, also sign it using your payment credential.
```bash
cardano-cli conway transaction witness \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--out-file payment.witness
```

#### 9. Assemble the transaction with all the witnesses
```bash
cardano-cli conway transaction assemble \
--tx-body-file tx.raw \
--witness-file  payment.witness \
--witness-file  name1-drep.witness \
--witness-file  name2-drep.witness \
--witness-file  name3-drep.witness \
--out-file tx.signed
```

#### 10. Submit the transaction
```bash
cardano-cli conway transaction submit \
--tx-file tx.signed
```

#### 11. Verify that your multi-signature DRep appears on-chain in the ledger's DRep state.
```bash
cardano-cli conway query drep-state \
--drep-script-hash $(cat drep-script.hash)
```

# Constitutional Committee Consortium

## Download and install Nix

#### 1. Get the single-user installation
```bash
sh <(curl -L https://nixos.org/nix/install) --no-daemon
```

#### 2. Configure your Nix installation
```bash
sudo mkdir -p /etc/nix/
sudo cat > nix.conf << EOF
experimental-features = nix-command flakes fetch-closure
trusted-users = $(whoami)
substituters = https://cache.iog.io https://cache.nixos.org/
trusted-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=
EOF
sudo mv nix.conf /etc/nix/
```

#### 3. Restart your system or device.
```
sudo reboot
```

## Install the Credential Manager tools

#### 1. Clone the credential manager repository
```bash
mkdir ~/repos
cd ~/repos
git clone git@github.com:IntersectMBO/credential-manager.git
```

#### 2. Upgrade the Nix package manager and enter the Nix Shell
Ensure that your system or device has a minimum of 8GB of memory before entering the shell. The first time you access it, the shell may take some time to build.
```bash
cd credential-manager
nix upgrade-nix
nix develop
```

#### 3. Update Cabal and compile the orchestrator-cli
After entering the shell for the first time, you’ll need to compile the `orchestrator-cli`. However, before doing so, make sure to update Haskell Cabal first:
```bash
cabal update
orchestrator-cli --help
```

## Generate Cardano keys and Openssl certificate signing request

#### 1. Generate the Cardano keys that you will use for the Membership, Delegation, or Voting role.
Replace the `NAME` variable with your name and the `ROLE` variable with your assigned role.
```bash
NAME="changeMe"
ROLE="changeMe"

cardano-cli conway address key-gen \
--verification-key-file ${ROLE}.vkey \
--signing-key-file ${ROLE}.skey
```

#### 2. Convert these keys into their OpenSSL equivalent format.
```bash
cat ${ROLE}.skey | jq -r ".cborHex" | cut -c 5- | (echo -n "302e020100300506032b657004220420" && cat) | xxd -r -p | base64 \
| (echo "-----BEGIN PRIVATE KEY-----" && cat) | (cat && echo "-----END PRIVATE KEY-----") > ${ROLE}-private.pem
```

#### 3. Create your certificate signing request
```bash
openssl req -new -key ${ROLE}-priv.pem -out ${NAME}-${ROLE}.csr
```

#### 4. After completing all prompts, check the CSR file content before sending it to the Head of Security.
```bash
openssl req -in ${NAME}-${ROLE}.csr -text -noout
```

## The Head of security role and Certificate authority
This section will guide you through creating your self-signed certificate (certificate authority) to help you assume the role of Head of Security.
#### 1. Generate Cardano keys and converting them to a PEM file
```bash
cardano-cli address key-gen --signing-key-file ca.skey --verification-key-file ca.vkey
cat ca.skey | jq -r ".cborHex" | cut -c 5- | (echo -n "302e020100300506032b657004220420" && cat) | xxd -r -p | base64 | (echo "-----BEGIN PRIVATE KEY-----" && cat) | (cat && echo "-----END PRIVATE KEY-----") > ca-priv.pem
```

#### 2. Create a self-signed certificate (CA)
You will be asked to provide some basic attribute information for the CA.
```bash
openssl req -x509 -new -key ca-priv.pem -days 3650 -out ca.cert
```

#### 3. Verify your Certificate Authority 
```bash
openssl x509 -in ca.cert -text -noout
```

#### 4. Examine the Consortium member's certificate signing request before approving it.
Assume that `name` and `role` refer to the respective name and role of the consortium member requesting the signature.
```bash
openssl req -in name-role.csr -text -noout
```

#### 5. Sign each Certificate Signing Requests
Of course, you need to provide the CA private key, the CA certificate, and the certificate signing request. The name and role on the signed certificate should correspond to those of the CSR file.
```bash
openssl x509 -days 365 -req -in name-role.csr -CA ca.cert -CAkey ca-priv.pem -out name-role.cert
```

