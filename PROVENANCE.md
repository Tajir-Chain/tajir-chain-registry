# Provenance

> This document records what Tajir Chain is built from and provides step-by-step instructions to independently verify every claim — without trusting us.

Tajir-authored contracts (token, staking, governance, fee distribution) are maintained in **[Tajir-Chain/tajir-contracts](https://github.com/Tajir-Chain/tajir-contracts)**.

All other contracts in this deployment are **unmodified upstream releases** of OP Stack and AggLayer.

---

## Deployment Tooling

| Tool | Version | Image |
|---|---|---|
| op-deployer | `v0.6.0` | `us-docker.pkg.dev/oplabs-tools-artifacts/images/op-deployer:v0.6.0` |

---

## Deployed Contract Versions — Mainnet

Each value below is returned by `version()` called on the live L1 contract. These are read directly from the chain — not self-reported.

| Contract | Address | `version()` |
|---|---|---|
| SystemConfig | [`0x8dF5680aBdeCb180142c8c0c389F31045B9e8Dd8`](https://etherscan.io/address/0x8dF5680aBdeCb180142c8c0c389F31045B9e8Dd8) | `3.13.1` |
| OptimismPortal | [`0x55Cd3d1e86987799fbbBb8a32b592276991b1092`](https://etherscan.io/address/0x55Cd3d1e86987799fbbBb8a32b592276991b1092) | `5.2.0` |
| L1CrossDomainMessenger | [`0xB708439da411119FAE785cf7446aee25d5dd6b03`](https://etherscan.io/address/0xB708439da411119FAE785cf7446aee25d5dd6b03) | `2.11.0` |
| L1StandardBridge | [`0x02549AB7db8e96faF0cedd48D842Fe4CA468fE03`](https://etherscan.io/address/0x02549AB7db8e96faF0cedd48D842Fe4CA468fE03) | `2.8.0` |
| DisputeGameFactory | [`0x61a9C4b2Db1CC8c506F18a6887E43faD91DF8a5e`](https://etherscan.io/address/0x61a9C4b2Db1CC8c506F18a6887E43faD91DF8a5e) | `1.4.0` |
| AgglayerBridge | [`0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe`](https://etherscan.io/address/0x2a3DD3EB832aF982ec71669E178424b10Dca2EDe) | `v1.1.0` |
| AgglayerManager | [`0x5132A183E9F3CB7C848b0AAC5Ae0c4f0491B7aB2`](https://etherscan.io/address/0x5132A183E9F3CB7C848b0AAC5Ae0c4f0491B7aB2) | `v1.0.0` |
| Rollup (AggchainECDSAMultisig) | [`0xB07134fF28e74a90667Ab5e3F7271a542D6E0358`](https://etherscan.io/address/0xB07134fF28e74a90667Ab5e3F7271a542D6E0358) | `v1.0.0` |

### Reproduce any row

```bash
# Replace the address with the contract you want to verify
cast call <CONTRACT_ADDRESS> 'version()(string)' --rpc-url $ETH_RPC

# Example — SystemConfig
cast call 0x8dF5680aBdeCb180142c8c0c389F31045B9e8Dd8 'version()(string)' --rpc-url $ETH_RPC
# Returns: 3.13.1
```

---

## TJR Token

### Deployment Record

| Property | Value |
|---|---|
| **Proxy (ERC-1967 / UUPS)** | [`0x9D98C61d1136cfA2ac263Be355350C97Ca41c110`](https://etherscan.io/address/0x9D98C61d1136cfA2ac263Be355350C97Ca41c110) |
| **Implementation (current)** | [`0x48bf1A70E9509B59831Dd23921eB9570F1ED7BA8`](https://etherscan.io/address/0x48bf1A70E9509B59831Dd23921eB9570F1ED7BA8) |
| **Implementation (initial, superseded)** | [`0x590A81b2D58ffcE6a7e385e34eC404355D2BD658`](https://etherscan.io/address/0x590A81b2D58ffcE6a7e385e34eC404355D2BD658) |
| **Implementation deploy tx** | [`0x1c791a632eca6e11465c86482081fb8a66f31667bd97a0ffcc5c578fddb057e2`](https://etherscan.io/tx/0x1c791a632eca6e11465c86482081fb8a66f31667bd97a0ffcc5c578fddb057e2) |
| **Proxy deploy + initialize tx** | [`0x5db1cbe3af8cce70956d3471cfc49ee444bc9347159dbf7d9a27c67e35d3c234`](https://etherscan.io/tx/0x5db1cbe3af8cce70956d3471cfc49ee444bc9347159dbf7d9a27c67e35d3c234) |
| **Compiler** | `0.8.27+commit.40a35a09`, optimizer 200 runs |
| **Source verification** | Sourcify (exact match) · Blockscout — both implementations |

### Upgrade history

The proxy is upgradeable (UUPS). Bind integrations to the **proxy** address; the
implementation changes.

| Date (UTC) | Implementation | Tx |
|---|---|---|
| 2026-08-07 | `0x590A81b2D58ffcE6a7e385e34eC404355D2BD658` (initial) | [`0x5db1cbe3…`](https://etherscan.io/tx/0x5db1cbe3af8cce70956d3471cfc49ee444bc9347159dbf7d9a27c67e35d3c234) |
| 2026-08-27 | `0x48bf1A70E9509B59831Dd23921eB9570F1ED7BA8` (adds burn) | [`0x9e6cfef1…`](https://etherscan.io/tx/0x9e6cfef12e40b852e91c16d9b024ff1907fd50b91e28eab09c92c3b8c6e7ad7a) |

The 2026-08-27 upgrade was executed by the 3-of-5 Safe `0xBA1D51BBB17Fd24Fd6bF2c030F3ca0E3a3B4d22E`
at block 25847146, confirmed by the Safe's `ExecutionSuccess` event (a Safe transaction
mines with receipt status 1 even when the inner call reverts, so the event is the check).

Read the live implementation directly rather than trusting this table:

```bash
cast storage 0x9D98C61d1136cfA2ac263Be355350C97Ca41c110 \
  0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc --rpc-url $ETH_RPC
```

### Minting

The entire initial supply of **650,000,000 TJR** is minted once inside `initialize()`, which executes as part of the proxy constructor. There is no separate mint transaction, and no mint function exists — supply can never increase.

The current implementation adds `burn(uint256)` and `burnFrom(address,uint256)`, so total supply **can decrease**. Integrators must read `totalSupply()` rather than assume a constant.

The only `Transfer(address(0) → holder)` event in the token's entire history is emitted by the proxy deploy + initialize transaction listed above. This can be verified on-chain:

```bash
# Confirm the one-and-only mint event
cast logs \
  --address 0x9D98C61d1136cfA2ac263Be355350C97Ca41c110 \
  --topic0 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef \
  --from-block 0 \
  --rpc-url $ETH_RPC
```

### Comparing Deployed Bytecode to Source

The implementation is verified on [Sourcify](https://sourcify.dev) (exact match) and [Blockscout](https://explorer.tajirchain.com).

```bash
# Fetch the deployed bytecode
cast code 0x48bf1A70E9509B59831Dd23921eB9570F1ED7BA8 --rpc-url $ETH_RPC
```

> **Important:** `cast code` on the implementation will **not** byte-match a plain `deployedBytecode` build artifact. `UUPSUpgradeable` stores its own address in three immutable slots (`__self`), so those **3 × 20 bytes differ by construction**. Mask those slots before comparing — everything else matches exactly.

---

## Genesis

`mainnet/genesis.json` and `testnet/genesis.json` are the **exact files used to initialise each chain**. Checksums are in each network's `SHA256SUMS`.

The resulting block 0 hash is public record — the file can be independently verified against the live chain without trusting us:

```bash
# Mainnet — block 0 hash
cast block 0 --field hash --rpc-url https://rpc.tajirchain.com
# Expected: 0x3f17c7c28185abe05e1bab0b0638ac2898bb0cdf9c58822b1c006609be1fe653

# Testnet — block 0 hash
cast block 0 --field hash --rpc-url https://rpc.testnet.tajirchain.com
# Expected: 0x7736d0c52f8b325fbfcd976005bfd215da93dfc74936342386ed92cdb17d434d
```

### Genesis File Provenance

| Property | Detail |
|---|---|
| **File origin** | Resolved artifacts taken from a running node — **not** pre-deploy templates |
| **Timestamp derivation** | `L1 origin block timestamp + 1` (not knowable before the L1 anchor block is chosen) |
| **Timestamp field** | Each `genesis.json` carries the **final, resolved** timestamp |
| **Rollup config** | Each `rollup.json` is `op-node`'s effective `optimism_rollupConfig` |
| **Verified against** | Block 0 on the live chain (both networks) |

---

*All on-chain data in this document was read from mainnet at deployment time. If you find a discrepancy, open an issue in this repository.*

