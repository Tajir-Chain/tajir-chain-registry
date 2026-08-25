# Provenance

What Tajir Chain is built from, and how to check that claim yourself.

Contracts authored by Tajir Chain live in
[`tajir-contracts`](https://github.com/Tajir-Chain/tajir-contracts).

L1 contracts were deployed with upstream **`op-deployer v0.6.0`**
(`us-docker.pkg.dev/oplabs-tools-artifacts/images/op-deployer:v0.6.0`).

## Deployed contract versions (mainnet, read from chain)

Each of these is the value returned by `version()` on the live L1 contract.

| Contract | `version()` |
|---|---|
| SystemConfig | `3.13.1` |
| OptimismPortal | `5.2.0` |
| L1CrossDomainMessenger | `2.11.0` |
| L1StandardBridge | `2.8.0` |
| DisputeGameFactory | `1.4.0` |
| AgglayerBridge | `v1.1.0` |
| AgglayerManager | `v1.0.0` |
| Rollup (AggchainECDSAMultisig) | `v1.0.0` |

Reproduce any row:

```bash
cast call 0x8dF5680aBdeCb180142c8c0c389F31045B9e8Dd8 'version()(string)' --rpc-url $ETH_RPC
```

## TJR token

| | |
|---|---|
| Proxy | `0x9D98C61d1136cfA2ac263Be355350C97Ca41c110` |
| Implementation | `0x590A81b2D58ffcE6a7e385e34eC404355D2BD658` |
| Implementation deploy tx | `0x1c791a632eca6e11465c86482081fb8a66f31667bd97a0ffcc5c578fddb057e2` |
| Proxy deploy + initialize tx | `0x5db1cbe3af8cce70956d3471cfc49ee444bc9347159dbf7d9a27c67e35d3c234` |
| Compiler | `0.8.27+commit.40a35a09`, optimizer 200 runs |

The full supply is minted once inside `initialize()`, which runs in the proxy
constructor — so there is no separate mint transaction. `Transfer(0x0 → holder)` is an
event of the proxy deploy transaction above, and it is the only `Transfer` in the
token's history to date.

The implementation is verified on Sourcify (exact match) and Blockscout.

### Comparing deployed bytecode to source

`cast code` on the implementation will **not** byte-match a plain `deployedBytecode`
build. `UUPSUpgradeable` stores its own address in three immutable slots (`__self`), so
those 3 × 20 bytes differ by construction. Mask them before comparing; everything else
matches exactly.

## Genesis

`mainnet/genesis.json` and `testnet/genesis.json` are the exact files used to start each
chain. Checksums are in each network's `SHA256SUMS`. The resulting block 0 hash is
public, so the file can be checked against the live chain without trusting us:

```bash
cast block 0 --field hash --rpc-url https://rpc.tajirchain.com
# 0x3f17c7c28185abe05e1bab0b0638ac2898bb0cdf9c58822b1c006609be1fe653

cast block 0 --field hash --rpc-url https://rpc.testnet.tajirchain.com
# 0x7736d0c52f8b325fbfcd976005bfd215da93dfc74936342386ed92cdb17d434d
```

The L2 genesis timestamp is derived at deploy time as `L1 origin block timestamp + 1`,
so it is not knowable before the L1 anchor block is chosen. The files published here are
the resolved artifacts taken from a running node, not pre-deploy templates: each
`genesis.json` carries the final timestamp, and each `rollup.json` is op-node's effective
configuration (`optimism_rollupConfig`). Both were checked against block 0 on the live
chain.
