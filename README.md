# Tajir Chain Registry

Canonical chain configuration, deployed contract addresses, and build provenance for
**Tajir Chain** — an OP Stack L2 settling through the AggLayer.

This repository is the reference for anyone running a node, integrating a bridge or
wallet, indexing the chain, or verifying our deployment independently.

| Network | Chain ID | L1 | AggLayer rollup ID | RPC | Explorer |
|---|---|---|---|---|---|
| Mainnet | `3377` | Ethereum (`1`) | 28 | https://rpc.tajirchain.com | https://explorer.tajirchain.com |
| Testnet | `7733` | Sepolia (`11155111`) | 62 | https://rpc.testnet.tajirchain.com | https://explorer.testnet.tajirchain.com |

## Layout

```
mainnet/           testnet/
├── chain.json         chain id, native currency, RPC and explorer endpoints
├── addresses.json     deployed L1 and L2 contract addresses
├── rollup.json        op-node rollup configuration
├── genesis.json       L2 genesis state (op-geth / op-reth)
└── SHA256SUMS         checksums for the files above
abis/                  ABIs for Tajir-authored contracts
PROVENANCE.md          versions and how to reproduce our deployment
```

## Native gas token

TJR is the **native gas token** of Tajir Chain, not an ERC-20 on L2. The canonical
token is an ERC-20 on Ethereum L1:

```
0x9D98C61d1136cfA2ac263Be355350C97Ca41c110   TJR, 18 decimals, fixed supply 650,000,000
```

It is locked on L1 by the AggLayer bridge and is native on L2. `gasTokenAddress()` on
the L2 bridge returns that address, with `gasTokenNetwork() = 0` (Ethereum).

## Running a node

Point `op-node` at `<network>/rollup.json` and initialise your execution client with
`<network>/genesis.json`. Verify checksums against `SHA256SUMS` first.

Confirm the genesis you built matches ours by comparing block 0:

```bash
cast block 0 --field hash --rpc-url https://rpc.tajirchain.com
# 0x3f17c7c28185abe05e1bab0b0638ac2898bb0cdf9c58822b1c006609be1fe653
```

## Contracts

Tajir-authored contracts (token, staking, governance, fee distribution) live in
[`tajir-contracts`](https://github.com/Tajir-Chain/tajir-contracts). Everything else in
this deployment is unmodified upstream OP Stack and AggLayer — see `PROVENANCE.md`.

## Corrections

Open an issue or a pull request. Every address here is verifiable on-chain; if something
does not match, we want to know.

## License

CC0 1.0 (public domain) — see `LICENSE`. This is chain data; use it freely, no
attribution required.
