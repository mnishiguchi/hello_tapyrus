# Tapyrus Dev Environment

A minimal Docker Compose setup to run [Tapyrus Core](https://github.com/chaintope/tapyrus-core) in **dev** mode.

## What is Tapyrus?

Tapyrus is an open-source, federated blockchain platform developed by [Chaintope](https://www.chaintope.com/).  

- **Federated Network**: Multiple “signers” collaboratively produce blocks using a threshold Schnorr signature scheme.  
- **Custom Tokens**: Supports colored-coin style tokens (`OP_COLOR`) alongside the native TPC currency.  
- **Dev-Mode (“regtest”)**: Quickly spin up a single-node chain for development and testing.

Learn more in the official docs and repo:

- GitHub: [chaintope/tapyrus-core](https://github.com/chaintope/tapyrus-core)  
- Getting Started Guide: [doc/getting_started.md](https://github.com/chaintope/tapyrus-core/blob/master/doc/tapyrus/getting_started.md)  
- Docker Image Instructions: [doc/docker_image.md](https://github.com/chaintope/tapyrus-core/blob/master/doc/docker_image.md)  
- Docker Hub: [tapyrus/tapyrusd](https://hub.docker.com/r/tapyrus/tapyrusd)

## Files

- `docker-compose.yml`  
  • `genesis` service: one‐time block generator  
  • `tapyrusd` service: Tapyrus node (RPC on port 2377)

- `tapyrus.conf`  
  Dev-mode config (networkid=1905960821, RPC, wallet settings)

- `data/`  
  Holds `genesis.1905960821` and chain data

- `scripts/tapyrus-cli`  
  Shortcut for `tapyrus-cli` inside the node container

## Quickstart

1. **Generate genesis block** (once):

   ```bash
   mkdir data
   docker compose run --rm genesis > data/genesis.1905960821
   ```

2. **Start Tapyrus node**:

   ```bash
   docker compose up -d tapyrusd
   ```

3. **Call RPCs**:

   ```bash
   scripts/tapyrus-cli getblockchaininfo
   scripts/tapyrus-cli getnewaddress
   scripts/tapyrus-cli listunspent
   ```

4. **Shutdown**:

   ```bash
   docker compose down
   ```
