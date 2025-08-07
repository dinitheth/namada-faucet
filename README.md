# Namada Faucet

A simple faucet server that funds accounts on a Namada testnet. It exposes a REST API and a minimal web UI for requesting tokens.

## Build

Ensure Rust and the protobuf compiler are installed, then build the faucet:

```bash
cargo build
```

## Running

The server expects a Namada key and connection details for a testnet node.

```bash
cargo run -- \
  --port 5000 \
  --difficulty 4 \
  --private-key <FAUCET_SK_HEX> \
  --chain-start 0 \
  --chain-id <CHAIN_ID> \
  --rpc http://localhost:26657
```

A Dockerfile is provided:

```bash
docker build -t namada-faucet .
docker run -p 5000:5000 \
  -e PORT=5000 \
  -e DIFFICULTY=4 \
  -e PRIVATE_KEY=<FAUCET_SK_HEX> \
  -e CHAIN_START=0 \
  -e CHAIN_ID=<CHAIN_ID> \
  -e RPC=http://namada-node:26657 \
  namada-faucet
```

## Requesting tokens

Open `http://localhost:5000/` in a browser, enter a wallet address and amount, and press **Receive**. The page solves the proof‑of‑work and submits the faucet request automatically.

To use the API directly:

```bash
# Get faucet settings and NAM token address
SETTINGS=$(curl -s http://localhost:5000/api/v1/faucet/setting)
TOKEN=$(echo $SETTINGS | jq -r '.tokens_alias_to_address.NAM')

# Request a challenge
CHALLENGE=$(curl -s http://localhost:5000/api/v1/faucet | jq -r '.challenge')
TAG=$(curl -s http://localhost:5000/api/v1/faucet | jq -r '.tag')

# Solve PoW in your client then submit:
curl -X POST http://localhost:5000/api/v1/faucet \
  -H 'Content-Type: application/json' \
  -d '{"solution":"<SOLUTION>","challenge":"'$CHALLENGE'","tag":"'$TAG'","transfer":{"token":"'$TOKEN'","target":"<ADDRESS>","amount":1}}'
```

The faucet will fund the specified address with the configured amount of NAM tokens, subject to rate limits and withdrawal limits.
