<div align="center">

# ⛓ Tajir Chain Registry

**The authoritative on-chain configuration, deployed contract addresses, and build provenance for Tajir Chain.**

*An OP Stack Layer 2 — settling through the AggLayer — with TJR as its native gas token.*

---

[![Mainnet Chain ID](https://img.shields.io/badge/Mainnet%20Chain%20ID-3377-6C3BDB?style=for-the-badge&logo=ethereum&logoColor=white)](https://explorer.tajirchain.com)
[![Testnet Chain ID](https://img.shields.io/badge/Testnet%20Chain%20ID-7733-9B59B6?style=for-the-badge&logo=ethereum&logoColor=white)](https://explorer.testnet.tajirchain.com)
[![License: CC0-1.0](https://img.shields.io/badge/License-CC0%201.0-lightgrey.svg?style=for-the-badge)](https://creativecommons.org/publicdomain/zero/1.0/)
[![Built on OP Stack](https://img.shields.io/badge/Built%20on-OP%20Stack-FF0420?style=for-the-badge&logo=optimism&logoColor=white)](https://stack.optimism.io)
[![Settled via AggLayer](https://img.shields.io/badge/Settlement-AggLayer-7B3FE4?style=for-the-badge)](https://agglayer.xyz)

</div>

---

## Overview

This repository is the **single source of truth** for Tajir Chain's canonical configuration. It is intended for:

- 🏗️ **Node operators** — running a full or sequencer node
- 🌉 **Bridge & wallet integrators** — connecting to Tajir Chain
- 🔍 **Indexers & explorers** — ingesting chain metadata
- 🔐 **Security researchers** — independently verifying our deployment
- 📊 **Stakeholders & investors** — auditing contract addresses and supply information

> Every address and hash in this repository is independently verifiable on-chain. Nothing here requires trusting us.

---

## Networks at a Glance

| Property | Mainnet | Testnet |
|---|---|---|
| **Chain ID** | `3377` | `7733` |
| **L1 Settlement** | Ethereum Mainnet (`1`) | Sepolia (`11155111`) |
| **AggLayer Rollup ID** | `28` | `62` |
| **Native Token** | TJR | tTJR *(no value)* |
| **Block Time** | 2 seconds | 2 seconds |
| **Stack** | OP Stack | OP Stack |
| **RPC Endpoint** | `https://rpc.tajirchain.com` | `https://rpc.testnet.tajirchain.com` |
| **Block Explorer** | [explorer.tajirchain.com](https://explorer.tajirchain.com) | [explorer.testnet.tajirchain.com](https://explorer.testnet.tajirchain.com) |

---

## Architecture

Tajir Chain is an **OP Stack L2** that settles through the **AggLayer** rather than posting fraud proofs or ZK proofs directly to Ethereum. The settlement chain of trust is:

```
Tajir Chain (L2, Chain ID 3377)
    │  OP Stack execution + sequencing
    │  2-second block time · 60 M gas limit
    ▼
AggLayer (Rollup ID 28)
    │  Aggregated validity proof
    ▼
Ethereum Mainnet (L1, Chain ID 1)
    │  SystemConfig · OptimismPortal · DisputeGameFactory
    │  DisputeGameFactory · L1CrossDomainMessenger
    ▼
Final settlement & data availability
```

L1 contracts were deployed with **`op-deployer v0.6.0`** against unmodified upstream OP Stack and AggLayer releases. See [`PROVENANCE.md`](./PROVENANCE.md) for exact versions and reproduction instructions.

---

## Repository Layout

```
tajir-chain-registry/
│
├── mainnet/                    Mainnet canonical configuration
│   ├── chain.json              Chain ID, native currency, RPC & explorer endpoints
│   ├── addresses.json          All deployed L1 & L2 contract addresses
│   ├── rollup.json             op-node rollup configuration (effective, from live node)
│   ├── genesis.json            L2 genesis state (op-geth / op-reth compatible)
│   └── SHA256SUMS              SHA-256 checksums for the four files above
│
├── testnet/                    Sepolia-backed testnet configuration (same layout)
│   ├── chain.json
│   ├── addresses.json
│   ├── rollup.json
│   ├── genesis.json
│   └── SHA256SUMS
│
├── abis/
│   └── TajirToken.json         ABI for the TJR ERC-20 token on Ethereum L1
│
├── PROVENANCE.md               Deployed contract versions & reproduction instructions
├── LICENSE                     CC0 1.0 Universal — public domain
└── README.md                   This file
```

---

## Native Gas Token — TJR

TJR is the **native gas token** of Tajir Chain. It is **not** an ERC-20 on L2. The canonical ERC-20 lives on Ethereum L1, where it is locked by the AggLayer bridge and becomes native on L2.

> **Bridge with `AgglayerBridge` only.** It is the canonical bridge for TJR and every
> other asset. The OP Stack `L1StandardBridge` is **not** usable here: it credits an
> ERC-20 on L2, and TJR has no ERC-20 on L2 — so a deposit through it cannot be
> delivered. Its address is deliberately omitted from this registry. The OP Stack
> entries that remain (`SystemConfig`, `OptimismPortal`, `BatchInbox`) are node
> infrastructure, needed to sync a node, not to move assets.

### Token Details

| Property | Value |
|---|---|
| **Name** | Tajir |
| **Symbol** | TJR |
| **Decimals** | 18 |
| **Initial Supply** | 650,000,000 TJR |
| **Supply Model** | No mint function — supply can never increase. Burn is supported, so it may decrease. Read `totalSupply()`; do not hardcode. |
| **L1 Contract (Proxy)** | [`0x9D98C61d1136cfA2ac263Be355350C97Ca41c110`](https://etherscan.io/address/0x9D98C61d1136cfA2ac263Be355350C97Ca41c110) |
| **L1 Contract (Implementation)** | [`0x48bf1A70E9509B59831Dd23921eB9570F1ED7BA8`](https://etherscan.io/address/0x48bf1A70E9509B59831Dd23921eB9570F1ED7BA8) |
| **Proxy Pattern** | ERC-1967 (UUPS) |
| **Compiler** | `0.8.27+commit.40a35a09`, optimizer 200 runs |
| **Verification** | Sourcify (exact match) · Blockscout |

The full supply is minted once inside `initialize()`, which runs in the proxy constructor. `Transfer(0x0 → holder)` in the proxy deploy transaction is the **only** mint event in the token's history.

On the L2 bridge, `gasTokenAddress()` returns `0x9D98C61d1136cfA2ac263Be355350C97Ca41c110` and `gasTokenNetwork()` returns `0` (Ethereum mainnet).

> **ABI:** [`abis/TajirToken.json`](./abis/TajirToken.json)

---

## Contract Addresses — Mainnet

### L1 — Ethereum Mainnet

#### OP Stack Contracts

| Contract | Address |
|---|---|
| SystemConfig | [`0x8dF5680aBdeCb180142c8c0c389F31045B9e8Dd8`](https://etherscan.io/address/0x8dF5680aBdeCb180142c8c0c389F31045B9e8Dd8) |
| OptimismPortal | [`0x55Cd3d1e86987799fbbBb8a32b592276991b1092`](https://etherscan.io/address/0x55Cd3d1e86987799fbbBb8a32b592276991b1092) |
| DisputeGameFactory | [`0x61a9C4b2Db1CC8c506F18a6887E43faD91DF8a5e`](https://etherscan.io/address/0x61a9C4b2Db1CC8c506F18a6887E43faD91DF8a5e) |
| ProxyAdmin | [`0xaC725B03EB24bd576972C1e3d6Fd31eb003A285d`](https://etherscan.io/address/0xaC725B03EB24bd576972C1e3d6Fd31eb003A285d) |
| BatchInbox | [`0x009AFe3a07Dc5Eb0E3d6DCFfBCEc2c3C90eA20E3`](https://etherscan.io/address/0x009AFe3a07Dc5Eb0E3d6DCFfBCEc2c3C90eA20E3) |
| AdminSafe (multisig) | [`0xBA1D51BBB17Fd24Fd6bF2c030F3ca0E3a3B4d22E`](https://etherscan.io/address/0xBA1D51BBB17Fd24Fd6bF2c030F3ca0E3a3B4d22E) |

#### AggLayer Contracts

| Contract | Address |
|---|---|
| AgglayerBridge | [`0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe`](https://etherscan.io/address/0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe) |
| AgglayerManager | [`0x5132A183E9F3CB7C848b0AAC5Ae0c4f0491B7aB2`](https://etherscan.io/address/0x5132A183E9F3CB7C848b0AAC5Ae0c4f0491B7aB2) |
| AgglayerGER | [`0x580bda1e7A0CFAe92Fa7F6c20A3794F169CE3CFb`](https://etherscan.io/address/0x580bda1e7A0CFAe92Fa7F6c20A3794F169CE3CFb) |
| RollupContract (AggchainECDSAMultisig) | [`0xB07134fF28e74a90667Ab5e3F7271a542D6E0358`](https://etherscan.io/address/0xB07134fF28e74a90667Ab5e3F7271a542D6E0358) |
| POL | [`0x455e53CBB86018Ac2B8092FdCd39d8444aFFC3F6`](https://etherscan.io/address/0x455e53CBB86018Ac2B8092FdCd39d8444aFFC3F6) |

### L2 — Tajir Chain

| Contract | Address |
|---|---|
| AgglayerBridge | [`0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe`](https://explorer.tajirchain.com/address/0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe) |
| AgglayerGER | [`0xa40D5f56745a118D0906a34E69aeC8C0Db1cB8fA`](https://explorer.tajirchain.com/address/0xa40D5f56745a118D0906a34E69aeC8C0Db1cB8fA) |
| WETH | [`0x7a7f337d883EE98DF40c4654b3ad4E3aacF304E8`](https://explorer.tajirchain.com/address/0x7a7f337d883EE98DF40c4654b3ad4E3aacF304E8) |
| OP Stack Predeploys | Standard at `0x4200000000000000000000000000000000000000` – `0x42000000000000000000000000000000000000FF` |

> Full address manifests: [`mainnet/addresses.json`](./mainnet/addresses.json) · [`testnet/addresses.json`](./testnet/addresses.json)

---

## Contract Addresses — Testnet (Sepolia)

| Contract | Address |
|---|---|
| TajirTestToken (L1) | [`0x55AEE58B82f2195D9F18e4C18eadB7B0c2ea6E09`](https://sepolia.etherscan.io/address/0x55AEE58B82f2195D9F18e4C18eadB7B0c2ea6E09) |
| SystemConfig | [`0xfce2f0a58726F4a6CEfA6ca7607B0Ea88215F789`](https://sepolia.etherscan.io/address/0xfce2f0a58726F4a6CEfA6ca7607B0Ea88215F789) |
| OptimismPortal | [`0x57Fec7B357CD769e62223a0cA3CC878F7DC92aaa`](https://sepolia.etherscan.io/address/0x57Fec7B357CD769e62223a0cA3CC878F7DC92aaa) |
| DisputeGameFactory | [`0xbdE5ca7e0DE67E182e5C3d9326026d0bBe55Ac91`](https://sepolia.etherscan.io/address/0xbdE5ca7e0DE67E182e5C3d9326026d0bBe55Ac91) |
| AgglayerBridge | [`0x528e26b25a34a4A5d0dbDa1d57D318153d2ED582`](https://sepolia.etherscan.io/address/0x528e26b25a34a4A5d0dbDa1d57D318153d2ED582) |
| AgglayerManager | [`0x32d33D5137a7cFFb54c5Bf8371172bcEc5f310ff`](https://sepolia.etherscan.io/address/0x32d33D5137a7cFFb54c5Bf8371172bcEc5f310ff) |
| AgglayerGateway | [`0xaA8103640A6C92af48A97D720168011E9f3Ec697`](https://sepolia.etherscan.io/address/0xaA8103640A6C92af48A97D720168011E9f3Ec697) |

---

## Running a Node

### Prerequisites

- [`op-geth`](https://github.com/ethereum-optimism/op-geth) or [`op-reth`](https://github.com/paradigmxyz/reth) (execution client)
- [`op-node`](https://github.com/ethereum-optimism/optimism) (consensus / rollup driver)
- An Ethereum L1 RPC endpoint (`$ETH_RPC`)

### Step 1 — Verify file integrity

Before initialising, confirm the canonical files are unmodified:

```bash
# From the repo root
sha256sum -c mainnet/SHA256SUMS
# All four files must return: OK
```

### Step 2 — Initialise the execution client

```bash
# op-geth
op-geth init --datadir ./data mainnet/genesis.json

# op-reth (alternative)
op-reth init --datadir ./data --chain mainnet/genesis.json
```

### Step 3 — Start op-node

```bash
op-node \
  --rollup.config=mainnet/rollup.json \
  --l1=$ETH_RPC \
  --l2=http://localhost:8551 \
  --l2.jwt-secret=./jwt.hex \
  --rpc.addr=0.0.0.0 \
  --rpc.port=9545
```

### Step 4 — Verify genesis block

Confirm the genesis state you built matches the canonical chain:

```bash
# Mainnet — must return the canonical block 0 hash
cast block 0 --field hash --rpc-url https://rpc.tajirchain.com
# Expected: 0x3f17c7c28185abe05e1bab0b0638ac2898bb0cdf9c58822b1c006609be1fe653

# Testnet
cast block 0 --field hash --rpc-url https://rpc.testnet.tajirchain.com
# Expected: 0x7736d0c52f8b325fbfcd976005bfd215da93dfc74936342386ed92cdb17d434d
```

A matching hash confirms your node is correctly initialised from the canonical genesis.

---

## Verifying the Deployment

Every deployed contract can be independently verified against the versions recorded in [`PROVENANCE.md`](./PROVENANCE.md).

### Check a contract version on-chain

```bash
# Example: SystemConfig
cast call 0x8dF5680aBdeCb180142c8c0c389F31045B9e8Dd8 'version()(string)' \
  --rpc-url $ETH_RPC
# Returns: 3.13.1
```

### Deployed contract versions (mainnet)

| Contract | `version()` |
|---|---|
| SystemConfig | `3.13.1` |
| OptimismPortal | `5.2.0` |
| DisputeGameFactory | `1.4.0` |
| AgglayerBridge | `v1.1.0` |
| AgglayerManager | `v1.0.0` |
| Rollup (AggchainECDSAMultisig) | `v1.0.0` |

### Verify TJR token bytecode

The implementation is verified on [Sourcify](https://sourcify.dev) (exact match) and Blockscout:

```bash
cast code 0x48bf1A70E9509B59831Dd23921eB9570F1ED7BA8 --rpc-url $ETH_RPC
```

> **Note:** `UUPSUpgradeable` stores its own address in three immutable slots (`__self`), so 3 × 20 bytes will differ from a plain build artifact by design. Mask those slots before comparing — everything else matches exactly.

---

## Integrating with Tajir Chain

### Adding to MetaMask / EVM Wallets

| Field | Mainnet | Testnet |
|---|---|---|
| Network Name | `Tajir Chain` | `Tajir Chain Testnet` |
| RPC URL | `https://rpc.tajirchain.com` | `https://rpc.testnet.tajirchain.com` |
| Chain ID | `3377` | `7733` |
| Currency Symbol | `TJR` | `tTJR` |
| Block Explorer | `https://explorer.tajirchain.com` | `https://explorer.testnet.tajirchain.com` |

### Programmatic Configuration

Consume [`mainnet/chain.json`](./mainnet/chain.json) or [`testnet/chain.json`](./testnet/chain.json) directly — they follow the [EIP-3091](https://eips.ethereum.org/EIPS/eip-3091) explorer standard and the [Ethereum Lists](https://github.com/ethereum-lists/chains) chain schema.

---

## Contract Source Code

Tajir-authored contracts — including the TJR token, staking, governance, and fee distribution — are maintained in a separate repository:

**[Tajir-Chain/tajir-contracts](https://github.com/Tajir-Chain/tajir-contracts)**

All OP Stack and AggLayer contracts deployed here are **unmodified upstream releases**. See [`PROVENANCE.md`](./PROVENANCE.md) for exact versions and deployment tooling.

---

## Contributing & Corrections

This repository contains publicly verifiable on-chain data. If an address, hash, or configuration does not match what you observe on-chain:

1. **Open an issue** — describe the discrepancy and include the on-chain evidence
2. **Open a pull request** — with the correction and the `cast` command that verifies it

We treat inaccuracies here as critical bugs. Target response time: **24 hours**.

---

## License

<a href="https://creativecommons.org/publicdomain/zero/1.0/">
  <img src="https://licensebuttons.net/p/zero/1.0/88x31.png" alt="CC0 1.0 Universal" />
</a>

**CC0 1.0 Universal — Public Domain Dedication**

The contents of this repository are chain configuration and factual deployment data. To the extent possible under law, **Tajir Chain** has waived all copyright and related or neighboring rights to this work.

You may copy, modify, distribute, and use this data — including for commercial purposes — without asking permission and without attribution required.

📄 [Full legal text](https://creativecommons.org/publicdomain/zero/1.0/legalcode)

> **Disclaimer:** THE DATA IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND. Always verify addresses against the live chain before relying on them in production systems.

---

## Acknowledgements

Tajir Chain is built on the shoulders of open-source giants:

- [OP Stack](https://stack.optimism.io) by [Optimism](https://optimism.io) — the execution and sequencing framework
- [AggLayer](https://agglayer.xyz) by [Polygon](https://polygon.technology) — unified liquidity and validity proof aggregation

---

<div align="center">

**Tajir Chain** &nbsp;·&nbsp; [tajirchain.com](https://tajirchain.com) &nbsp;·&nbsp; [Explorer](https://explorer.tajirchain.com) &nbsp;·&nbsp; [GitHub](https://github.com/Tajir-Chain)

*Built for global commerce. Settled on Ethereum.*

</div>

