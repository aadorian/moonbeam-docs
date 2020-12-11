---
title: Chainlink Node
description: How to setup a Chainlink node for the Moonbeam Network
---

# Run a Chainlink Node

## Overview

As an open permissionless network, anyone may choose to operate an Oracle providing data to smart contracts running on Moonbeam.

This article provides an overview on setting up a Chainlink Oracle on Moonbase Alpha. The provided examples are for demonstration purposes only, passwords MUST be managed securely and never stored in plaintext.

## Advanced users

- Chainlink documentation: [https://docs.chain.link/docs/running-a-chainlink-node](https://docs.chain.link/docs/running-a-chainlink-node)
- Moonbase Alpha WSS: wss://wss.testnet.moonbeam.network
- LINK Token on Moonbase Alpha: 0x2B13000735C0fC878673732Ee9bF8Ba5d76F7EC9
- Faucet instructions: https://docs.moonbeam.network/getting-started/testnet/faucet/

## Step by Step Instructions

### Overview

- Setup a Chainlink node connected to Moonbeam Alpha
- Fund node
- Deploy an Oracle contract
- Create Job on the Chainlink node
- Bond node and Oracle
- Test using a client contract

### Requirements

- Docker for running Postgres DB and ChainLink node containers
- [Metamask setup for Moonbeam](integrations/metamask/) with a funded account
- Access to Remix - [Moonbeam Tutorial](/integrations/remix/).

Note: In this guide we assume an Ubuntu 18.04-based environment. Call-outs for MacOs are included.

Note: The commands included in this guide are intended for a development setup, do not use this for a production environment.

### Node Setup

#### Create a new directory for files

For example `mkdir -p ~/.chainlink-moonbeam` then `cd ~/.chainlink-moonbeam`

#### Create a Postgres DB with Docker

```
docker run -d --name chainlink_postgres_db \
    --volume chainlink_postgres_data:/var/lib/postgresql/data \
    -e 'POSTGRES_PASSWORD={YOU_PASSWORD_HERE}' \
    -e 'POSTGRES_USER=chainlink' \
    --network host \
    -t postgres:11
```

MacOs users may replace `--network host \` with `-p 5432:5432`.

#### Create an Environment file for Chainlink in newly created directory

File is read on creation of the ChainLink container.

Reminder, do not store any production passwords in a plaintext file. This is for example purposes only.

```
echo "ROOT=/chainlink
LOG_LEVEL=debug
ETH_CHAIN_ID=1287
MIN_OUTGOING_CONFIRMATIONS=2
LINK_CONTRACT_ADDRESS={LINK TOKEN CONTRACT ADDRESS}
CHAINLINK_TLS_PORT=0
SECURE_COOKIES=false
GAS_UPDATER_ENABLED=false
ALLOW_ORIGINS=*
ETH_URL=wss://wss.testnet.moonbeam.network
DATABASE_URL=postgresql://chainlink:{YOUR_PASSWORD_HERE}@localhost:5432/chainlink?sslmode=disable
MINIMUM_CONTRACT_PAYMENT=0" > ~/.chainlink-moonbeam/.env
```

MacOs users may replace `localhost` with `host.docker.internal`.

#### Create a `.api` file in the new directory

Stores the user and password used to access the node’s API, the node’s operator UI and the chainlink command line.

Reminder, do not store any production passwords in a plaintext file. This is for example purposes only.

```
echo "{AN_EMAIL_ADDRESS}" >  ~/.chainlink-moonbeam/.api
echo "{ANOTHER_PASSWORD}"   >> ~/.chainlink-moonbeam/.api
```

#### Create a `.password` file in the new directory

Wallet password for the node address.

Reminder, do not store any production passwords in a plaintext file. This is for example purposes only.

```
echo "{THIRD_PASSWORD}" > ~/.chainlink-moonbeam/.password
```

#### Launch the containers

```
docker run -d --name chainlink_oracle_node \
    --volume $(pwd):/chainlink \
    --env-file=.env \
    --network host \
    -t smartcontract/chainlink:0.9.2 \
        local n \
        -p /chainlink/.password \
        -a /chainlink/.api
```

MacOs users may replace `--network host \` with `-p 6688:6688`

Verify everything is running with `docker ps` and verify the logs are progressing with `docker logs --tail 50 {container_id}`

### Contract Setup

#### Fund the node

Log onto the ChainLink node's UI (http://localhost:6688/) using the credentials from the `.api` file. Go to the 'Configuration Page` and copy the node address.

<screenshot>

Use the [Moonbeam Faucet](https://docs.moonbeam.network/getting-started/testnet/faucet/) to fund the node so it can write to the chain.

#### Create Oracle Contract

The Oracle contract is the piece of the communication that connects the chain to the node. It’s in charge of forwarding the requests that come from the chain to the node, so it resolves them and submits the result in the caller contract.

The source code of the contract is very simple on our side, because we will be just importing it from Chainlink’s GitHub repository:

In Remix create a new contract

```
pragma solidity ^0.6.6;

import "https://github.com/smartcontractkit/chainlink/evm-contracts/src/v0.6/Oracle.sol";
```

Compile and deploy this contract, write down the contract address!

#### Create Job on the Oracle node

Referring to Chainlink’s official documentation, this is the definition of a job:

> Job specifications, or specs, contain the sequential tasks that the node must perform to produce a final result. A spec contains at least one initiator and one task, which are discussed in detail below. Specs are defined using standard JSON so that they are human-readable and can be easily parsed by the Chainlink node.

Seeing an Oracle as an API service, a job here would be one of the functions that we can call and that will return a result. In order to create our first job, go to the Jobs sections of your node (http://localhost:6688/jobs), click on New Job and paste the following JSON. This will create a job that will request the current ETH price in USD.

```
{
  "initiators": [
    {
      "type": "runlog",
      "params": { "address": "YOUR_ORACLE_CONTRACT_ADDRESS" }
    }
  ],
  "tasks": [
    {
      "type": "httpget",
      "params": { "get": "https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD" }
    },
    {
      "type": "jsonparse",
      "params": { "path": [ "USD" ] }
    },
    {
      "type": "multiply",
      "params": { "times": 100 }
    },
    { "type": "ethuint256" },
    { "type": "ethtx" }
  ]
}
```

#### Bonding Node and Oracle

The last step to have a fully configured Chainlink Oracle is bonding the contract to the node. A node can listen to the requests sent to a certain Oracle contract, but only authorized (aka. bonded) nodes can fulfill the request with a result.

In order to set this authorization, the `setFulfillmentPermission function` from the Oracle contract must be called by the owner of the contract using two parameters:

- The address of the node that we want to bond to the contract.
- A boolean indicating the status of the bond (true, in this case).

Return to the Oracle contract deployed earlier in Remix and enter these values and execute the operation.

#### Test the Oracle

To verify the Oracle is up and answering requests, follow our [Using an Oracle](ntegrations/oracles/chainlink/) Tutorial.
