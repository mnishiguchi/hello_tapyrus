# Tapyrus Dev Environment

A minimal Docker Compose setup to run a single-node Tapyrus Core chain in
**dev** mode, plus an interactive Ruby REPL connected to the node’s RPC.

## What is Tapyrus?

Tapyrus is an open-source, federated blockchain platform developed by [Chaintope](https://www.chaintope.com/).

- **Federated Network**: Multiple “signers” produce blocks with a threshold Schnorr signature scheme.
- **Custom Tokens**: Supports colored-coin style tokens (`OP_COLOR`) alongside native TPC.
- **Dev Mode (“regtest”)**: Spin up a self-contained, single-node chain for rapid development.

Learn more:

- **[Tapyrus Core (GitHub)][tapyrus-core]**
- **[Getting-Started Guide][getting-started]**
- **[Docker Usage Docs][docker-docs]**
- **[tapyrus/tapyrusd on Docker Hub][docker-hub]**

<!-- Reference-style link targets -->

[tapyrus-core]: https://github.com/chaintope/tapyrus-core
[getting-started]: https://github.com/chaintope/tapyrus-core/blob/master/doc/tapyrus/getting_started.md
[docker-docs]: https://github.com/chaintope/tapyrus-core/blob/master/doc/docker_image.md
[docker-hub]: https://hub.docker.com/r/tapyrus/tapyrusd

## Files

```
├── docker-compose.yml
├── tapyrus.conf
├── data/
└── tapyrus-client/
    └── Dockerfile.dev 
```

- **`docker-compose.yml`**
  • `tapyrusd`: Tapyrus Core node in dev mode (imports a fixed genesis block via `GENESIS_BLOCK_WITH_SIG`).
  • `tapyrus-client`: Ruby IRB with `tapyrus`‐gem, auto-connected to RPC.
- **`tapyrus.conf`**
  Dev-mode settings (networkid=1905960821, RPC credentials, keypool, bind, etc.).
- **`data/`**
  Persists blockchain & wallet data between runs.
- **`tapyrus-client/`**
  Builds Ruby image with `tapyrus`‐gem.

## Quick Start

Set up a local Tapyrus dev node and start mining TPC.

### 1. Start the Tapyrus node

In one terminal, run:

```bash
docker compose up tapyrusd
```

This launches the Tapyrus daemon in dev mode, initializes the blockchain from a
fixed genesis block, and opens the JSON-RPC interface.

### 2. Launch the interactive Ruby console

In a second terminal, run:

```bash
docker compose run --rm tapyrus-client
```

This starts an IRB session with the `tapyrus` gem pre-loaded and connected to
your running node. You should see:

```
✓ Tapyrus RPC client is ready ▶︎ $client (tapyrusd:2377)
```

Now you can invoke RPC methods directly from Ruby.

### 3. Generate a wallet address & import the Aggregate WIF

First, create a fresh address to receive your coinbase rewards, then import the
default aggregate key so the node can sign blocks.

```ruby
addr    = $client.getnewaddress
agg_wif = "cUJN5RVzYWFoeY8rUztd47jzXCu1p57Ay8V7pqCzsBD3PEXN7Dd4"  # default dev Aggregate key
$client.importprivkey(agg_wif, "dev_aggkey", false)               # label, skip rescan
```

### 4. Mine 101 blocks to unlock rewards

Mining produces new blocks and awards coinbase outputs to your address. You
must mine at least 101 blocks for the first coinbase output to mature.

```ruby
block_hashes = $client.generatetoaddress(101, addr, agg_wif)
```

Here, 101 blocks = 1 block to mature the coinbase + 100 confirmations for additional security.

### 5. Check your wallet balance

Once mining is complete, list all unspent transaction outputs in your wallet:

```ruby
$client.listunspent
```

A typical response looks like:

```ruby
[
  {
    "txid"          => "abcd1234…",
    "vout"          => 0,
    "address"       => addr,
    "amount"        => 50.0,
    "confirmations" => 101
  }
]
```

You now have 50 TPC ready to spend. Feel free to send transactions, create
colored tokens, or explore other Tapyrus features.

> **Tips**
>
> - `$client.dumpprivkey(addr)` shows the private key for any address.
> - You may see references to `$client.generate(101, agg_wif)`, but this method is deprecated.
> - Always prefer `generatetoaddress + importprivkey` for compatibility and clarity.

## Cleanup

When you’re done developing and want to stop the node:

1. **Exit the Ruby IRB session**
   In your IRB terminal, type `exit` or press `Ctrl-D`. This will automatically stop the `tapyrus-client` container.

2. **Stop all running containers & remove volumes**
   From your project root (where `docker-compose.yml` lives), run:

   ```bash
   docker compose down --volumes --remove-orphans
   ```

   - `--volumes` (or `-v`) deletes any named volumes (e.g. the contents of `./data/`).
   - `--remove-orphans` cleans up any leftover containers not defined in your compose file.

At this point, no Tapyrus containers are running. To restart later (preserving
existing chain data), simply run through the **Quick Start** steps again.
