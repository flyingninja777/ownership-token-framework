# HYPE Token Research Report

**Token:** HYPE (Hyperliquid)
**Network:** Hyperliquid L1 (HyperCore + HyperEVM)
**Research Date:** 2026-04-20
**Framework:** Aragon Ownership Token Framework

---

## Executive Summary

HYPE is the native token of Hyperliquid, an L1 blockchain running HyperBFT consensus with two components: HyperCore (onchain perps/spot orderbooks) and HyperEVM (EVM-compatible smart contracts). Onchain data as of 2026-04-20 confirms active value accrual: **747,261.63 HYPE has been hard-burned** (`totalSupply` 1,000,000,000 → 999,252,738.37 per `info`/`tokenDetails`), and a further **43,459,601.14 HYPE** is held in the Assistance Fund system address `0xfefe…fefe`. On Dec 17-24, 2025, validators passed a stake-weighted vote (85% for / 7% against / 8% abstain) to **formally recognise the AF address as a burner address**, so the AF balance is treated as burned by protocol convention — bringing the effective burn to ~44.2M HYPE (~4.42% of genesis).

Key findings:

1. **Validator concentration:** the Hyper Foundation directly runs 5 validators holding **54.44% of active stake** (233.5M HYPE), enough to unilaterally pass any stake-weighted simple-majority vote and enough to block any 2/3 supermajority from excluding it.
2. **Verifiability:** Bridge2 (Arbitrum) and WHYPE (HyperEVM, `WHYPE9` per docs, `WCTC` stale label on HyperScan) are source-verified and non-upgradeable. The HyperCore L1 binary at `hyperliquid-dex/node` is Apache-2.0 licensed but **binaries only — no source** — so HYPE's mint/burn/fee-routing logic cannot be audited at the source level.
3. **Governance:** no Governor/Timelock contracts exist. Governance happens via (a) continuous HyperBFT consensus and (b) discrete stake-weighted validator governance votes (HIP-3 slashing, Sept 2025 USDH ticker allocation, Dec 2025 AF burner-address recognition). Tokenholders' only entry point is delegation to a validator.
4. **Offchain dependencies:** "HYPERLIQUID" trademark (USPTO 99599981) is held by Hyper Foundation; primary interface `app.hyperliquid.xyz` run by Hyperliquid Labs Pte. Ltd. (Singapore UEN 202402326K). No binding link to tokenholders.

**Key Finding:** HYPE tokenholders have indirect, delegation-only influence over protocol parameters. The Foundation's 54.44% active-stake share means stake-weighted governance outcomes are determined by the Foundation unless a supermajority of non-Foundation stake coordinates against it. Value accrual is real and active (~44.2M effective burn; ~2.3% staking APR; fee discounts up to 40%), but the closed-source L1 and validator-concentration risks remain material.

---

## Contract Architecture

### Core Contracts

| Contract | Address | Network | Purpose | Upgradeable | Block Explorer |
|----------|---------|---------|---------|-------------|----------------|
| HYPE | Native L1 token (`tokenId` `0x0d01dc56dcaaca66ad901c959b4011ec`) | Hyperliquid L1 (HyperCore) | Protocol-native gas / staking asset | Yes — only via L1 binary release adopted by ≥2/3 stake-weighted validator consensus (no smart-contract upgrade path; execution logic is closed-source Rust binary) | [Hyperliquid explorer](https://app.hyperliquid.xyz/explorer); [HyperEVMScan](https://hyperevmscan.io); [Hyperscan](https://www.hyperscan.com) |
| WHYPE | `0x5555555555555555555555555555555555555555` | HyperEVM | Wrapped HYPE (ERC-20, WETH-clone) | No — immutable, no proxy, no admin | [HyperEVMScan](https://hyperevmscan.io/address/0x5555555555555555555555555555555555555555#code); [HyperScan](https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555) |
| Bridge2 | `0x2Df1c51E09aECF9cacB7bc98cB1742757f163dF7` | Arbitrum One | USDC bridge Arbitrum ↔ Hyperliquid L1 | No — non-proxy, no admin, state changes require validator-set signatures | [Arbiscan](https://arbiscan.io/address/0x2df1c51e09aecf9cacb7bc98cb1742757f163df7#code) |

### Native HYPE — Is It Upgradeable?

The native HYPE token is not a smart contract; it is a balance entry in the HyperCore L1 state machine. "Upgradeability" therefore has a different meaning than for an ERC-20:

1. **No proxy / no admin key.** There is no proxy contract, no `upgradeTo()`, no implementation slot, and no EOA or multisig with unilateral authority to change HYPE's logic or balances.
2. **L1-binary upgrades.** The behaviour of HYPE (mint/burn rules, staking emissions, fee routing to the Assistance Fund, etc.) is defined by the closed-source HyperCore node binary distributed at https://github.com/hyperliquid-dex/node. That repository contains only compiled binaries (no source) under Apache 2.0. A change to HYPE semantics therefore requires:
   - Hyperliquid Labs / the Foundation shipping a new binary release, and
   - At least 2/3 stake-weighted validators adopting that release (per HyperBFT's 2/3 safety threshold).
3. **Closed-source execution layer.** The exact code that implements mint, burn, transfer, stake, and Assistance Fund logic is not publicly available. Aragon has not been able to independently verify from source which specific logic paths exist or under what conditions balances can change. The report's claims about HYPE mechanics rely on (a) the Hyperliquid GitBook and (b) observable on-chain behaviour via the official info API.

**Source — node repo (binaries only):** https://github.com/hyperliquid-dex/node → README states the repository distributes the non-validator and validator `hl-node` binaries; no Rust/Go source for HyperCore is published.

### Bridge2 Contract (Arbitrum)

**Source (open):** https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol (842 lines, Solidity `^0.8.9`, MIT).

Bridge2 custodies USDC deposits from Arbitrum and releases them back on validator-signed withdrawals from the L1. It does **not** custody HYPE — HYPE is native to the L1 and does not bridge through Bridge2.

**Key state and functions (with source lines):**

| Element | Location | Notes |
|---------|----------|-------|
| `ValidatorSet` struct (epoch, validators[], powers[]) | Bridge2.sol:78-82 | Powers are stake-weighted |
| Hot / cold wallet split | Bridge2.sol:11-16 comments | Hot signs withdrawals; cold unlocks |
| `batchedRequestWithdrawals()` | Bridge2.sol:320 | Hot-wallet quorum signs withdrawals |
| `finalizeWithdrawal()` | Bridge2.sol:338 | Called after dispute period |
| `updateValidatorSet()` | Bridge2.sol:466 | Hot-wallet quorum updates next set |
| `finalizeValidatorSetUpdate()` | Bridge2.sol:558 | Applies pending set after dispute window |
| `voteEmergencyLock()` | Bridge2.sol:746 | Any registered locker can vote to pause |
| `emergencyUnlock()` | Bridge2.sol:773 | Requires 2/3 cold-wallet signatures |
| `lockerThreshold` state var | Bridge2.sol:137 | Number of locker votes required to pause |
| Lockers mapping | Bridge2.sol:258, 614-617 | Added by hot-wallet quorum, removed by cold-wallet quorum |
| "Hot addresses auto-registered as lockers" | Bridge2.sol:43 comment | All validator hot wallets are lockers |

**Verified On-Chain State (2026-04-20, Arbitrum):**
```
eth_call  0x2Df1c51E09aECF9cacB7bc98cB1742757f163dF7  epoch()  →  0x07  (epoch = 7)
eth_call  0x2Df1c51E09aECF9cacB7bc98cB1742757f163dF7  paused() →  0x00  (false)
Bytecode length: 38,790 hex chars
```

Bridge2 is **non-upgradeable**: no `UUPS`/`Transparent` proxy pattern, no `owner()`, no `UPGRADER_ROLE`. Validator-set transitions occur via `updateValidatorSet` + `finalizeValidatorSetUpdate` (Bridge2.sol:466, 558), not via implementation swaps.

### WHYPE Contract (HyperEVM)

**Address:** `0x5555555555555555555555555555555555555555`
**Contract name (per HyperEVMScan source):** `WHYPE9`
**Compiler:** Solidity `v0.5.17+commit.d19bba13`
**Proxy:** None

**Source verification:** According to the official Hyperliquid GitBook (https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype), WHYPE "uses the same source code as wrapped ETH on Ethereum, apart from the token name and symbol" and is deployed at `0x55…55` as an immutable contract. The GitBook names the contract `WHYPE9` (the digit suffix matches the `WETH9` naming convention of the original WETH implementation).

**Onchain verification (2026-04-20, via public HyperEVM RPC `eth_call`):**
```
name()     → "Wrapped HYPE"
symbol()   → "WHYPE"
decimals() → 18
```

**Block-explorer cross-check:**
- https://hyperevmscan.io/address/0x5555555555555555555555555555555555555555#code — this explorer (Etherscan family for HyperEVM) shows the source verified under the name `WHYPE9`. The page itself is protected by Cloudflare's anti-bot interstitial, so Aragon could not fetch the HTML programmatically from this environment, but the URL is the canonical official source-verification target per the Hyperliquid docs.
- https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555 — the Blockscout-based HyperScan indexer returns `is_verified: true`, but with a **stale contract name `WCTC`** (Wrapped Core Token Contract) and no proxy. Aragon has cross-checked with the onchain `name()` / `symbol()` calls above, which confirm the deployed bytecode corresponds to "Wrapped HYPE" / "WHYPE". The `WCTC` label on HyperScan appears to be a metadata artefact from a pre-rename verification and does not reflect the current onchain state.

**Immutability:** The WETH9 pattern has no `owner`, no `admin`, no `upgradeTo`, no mint/burn beyond deposit/withdraw, and no blacklist. Combined with the lack of a proxy, WHYPE is effectively immutable: no address can unilaterally change its logic.

---

## Governance Flow Diagram

```mermaid
graph TD
    subgraph "HYPE Tokenholders"
        TH[Token Holders]
    end

    subgraph "Delegation Layer"
        TH -->|Delegate| V[Validators]
        TH -->|Stake| V
    end

    subgraph "Validator Set (24 Active)"
        V --> FV[Foundation Validators (5)<br/>54.44% active stake]
        V --> IV[Independent Validators (19)<br/>45.56% active stake]
    end

    subgraph "Consensus Control"
        FV -->|Controls| CON[HyperBFT Consensus<br/>2/3 quorum = 67%]
        IV -->|Participates| CON
    end

    subgraph "Protocol Control"
        CON -->|Governs| L1[HyperCore L1]
        CON -->|Governs| BR[Bridge2 Arbitrum]
        CON -->|Governs| EVM[HyperEVM]
    end

    subgraph "Hyper Foundation"
        HF[Foundation] -->|Discretionary Delegation| FV
        HF -->|Owns| TM[Trademark]
    end

    style FV fill:#ff9999
    style HF fill:#ff9999
    style TM fill:#ffcc99
```

**Critical Finding:** The chain of control terminates at the Hyper Foundation, not tokenholders. While delegators choose validators, the Foundation's 54.44% active-stake share (53.77% of all-validator stake) means Foundation-run validators can pass simple-majority stake-weighted votes and block 2/3 supermajorities unilaterally.

---

## Metric 1: Onchain Control

### 1.1 Governance Workflow

**Status: WARNING**

Hyperliquid does **not** have an onchain Governor/Timelock contract. All governance is executed by validators via HyperBFT consensus. There are two distinct voting channels, both stake-weighted:

#### Channel A — HyperBFT consensus votes (continuous)
Every block, validators sign the next block under HyperBFT's 2/3-stake safety and liveness rules. This is the mechanism that decides: block production, validator-set transitions, and — by running a specific binary release — which execution rules are in force.

**Source (Hyperliquid docs — HyperCore overview covers HyperBFT):** https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/overview — the primary-source page states: *"Hyperliquid is secured by HyperBFT, a variant of HotStuff consensus"* and *"blocks are produced by validators in proportion to the native token staked to each validator."*

#### Channel B — Stake-weighted validator governance votes (discrete proposals)
For specific protocol-level decisions the L1 also exposes discrete validator votes. Publicly observable recent votes include:

| Vote | Subject | Outcome | Source |
|------|---------|---------|--------|
| HIP-3 slashing | Authorise operator slashing for HIP-3 deployers that violate rules | Passed | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals |
| USDH ticker allocation (Sept 2025) | Assign the canonical USDH ticker to Native Markets | Passed 70.66% stake-for | Announced via Hyperliquid official channels (stake-weighted validator vote) |
| Assistance Fund burner recognition (Dec 17-24, 2025) | Formally recognise `0xfefe…fefe` as a burner address so HYPE held there is treated as burned | Passed 85% stake-for, 7% against, 8% abstain | Hyper Foundation announcement: https://x.com/HyperFND/status/2003684257949188396 |

The API endpoint `{"type":"validatorL1Votes"}` on `https://api.hyperliquid.xyz/info` returns the current in-flight validator L1 votes. At the time of verification this returned no active open vote; the list of historical votes is not served by the info API and must be sourced from Foundation announcements.

#### What HYPE-holders can and cannot do
- **Can:** stake HYPE to a validator (including self-delegation if running a validator) → that validator's voting power in both channels increases by the amount delegated.
- **Can:** un-delegate after the 7-day unstaking window and re-delegate to a different validator.
- **Cannot:** vote directly on a proposal without going through a validator. There is no `castVote` contract on HyperEVM, no Snapshot space that bindingly controls onchain parameters, and no veto mechanism for non-validator holders.
- **Cannot:** verify from source code which subset of execution-rule changes requires a Channel-B discrete vote versus being decided implicitly by which binary ≥2/3 of stake runs (the L1 is closed source; see Contract Architecture).

#### Live validator topology (2026-04-20)

Data: `POST https://api.hyperliquid.xyz/info` `{"type":"validatorSummaries"}` → 30 entries.

```
Total validators:            30
Active (unjailed) validators: 24
Jailed validators:             4
Inactive (unjailed):           2
Total stake across active:   428,962,738.24 HYPE
```

**Foundation validators (Hyper Foundation 1-5) — all currently active:**

| # | Validator name | Address (validator = signer; Foundation uses the same hot/cold key) | Stake (HYPE) | % of active stake |
|---|---------------|------|-------------:|------------------:|
| 1 | Hyper Foundation 2 | `0xa82fe73bbd768bc15d1ef2f6142a21ff8bd762ad` | 56,676,658.31 | 13.21% |
| 2 | Hyper Foundation 3 | `0x80f0cd23da5bf3a0101110cfd0f89c8a69a1384d` | 55,486,946.62 | 12.94% |
| 3 | Hyper Foundation 1 | `0x5ac99df645f3414876c816caa18b2d234024b487` | 53,212,837.28 | 12.41% |
| 4 | Hyper Foundation 4 | `0xdf35aee8ef5658686142acd1e5ab5dbcdf8c51e8` | 53,144,941.92 | 12.39% |
| 5 | Hyper Foundation 5 | cold `0x66be52ec79f829cc88e5778a255e2cb9492798fd`; hot `0x5795ab6e71ecbefa255fc4728cc34893ba992d44` | 15,007,172.34 | 3.50% |
| | **Foundation total** | | **233,528,556.47** | **54.44%** |

**Concentration assessment:**
- No single validator > 33%: **PASS** (largest is 13.21%).
- Foundation > 50%: **FAIL** (54.44% of active stake; equivalently ~53.77% of all-validator stake including inactive/jailed entries).
- Foundation > 67% (HyperBFT safety threshold): **NO** (54.44%).
- Top 5 validators by stake = 4 Foundation + Anchorage-by-Figment (`0x420a4ed7…`, 8.09%) = 62.53% of active stake.

**Finding:** Validator voting exists (both continuous HyperBFT and discrete stake-weighted governance votes), and tokenholders' only entry point is delegating to a validator. The Hyper Foundation directly controls 54.44% of active stake, which is enough to unilaterally pass or block any stake-weighted Channel-B vote that uses a simple-majority threshold, and enough to deny the 2/3 supermajority any other coalition would need to exclude the Foundation.

### 1.2 Role Accountability

**Status: WARNING**

#### Hot vs. cold wallet model (Bridge2)

The Hyperliquid validator key model is documented directly in the Bridge2 source:

> *"Each validator has a hot (in memory) and cold wallet. To unlock the bridge, a quorum of cold wallet signatures is required."* — Bridge2.sol:11-16.

> *"L1 operation will automatically register all validator hot addresses as lockers."* — Bridge2.sol:43.

In plain terms:
- **Hot wallet** — always-online key held by the validator node. Used to sign withdrawals, validator-set updates, and emergency-lock votes. It is the hot address that the L1 auto-registers as a bridge **locker**, so any active validator can unilaterally vote to pause the bridge.
- **Cold wallet** — offline / custodial key. Used only to authorise the bridge's most sensitive action: **emergency unlock** (which also atomically rotates the validator set). Requires 2/3 stake-weighted quorum of cold signatures.

Most independent validators use distinct hot and cold keys; Hyper Foundation 1-4 have registered the **same address** as both hot and cold (see live-data table in 1.1), which is a noteworthy operational-security choice that slightly narrows the defence-in-depth the hot/cold split is intended to provide.

#### Bridge2 roles (Arbitrum) — fully enforced in source

| Role | Function | Current Holder | Control Mechanism | Source line |
|------|----------|----------------|-------------------|-------------|
| Hot-wallet validators | Sign withdrawals, propose validator-set updates | 24 active validator hot addresses | ≥2/3 stake-weighted signature quorum | Bridge2.sol:320, 466 |
| Cold-wallet validators | Emergency unlock, invalidate withdrawals | Same 24 validators' cold addresses | ≥2/3 stake-weighted signature quorum | Bridge2.sol:773 |
| Lockers | Pause bridge during dispute period | All validator hot addresses, auto-registered | `lockerThreshold` votes (state var set at construction) | Bridge2.sol:43, 137, 746 |
| Finalizers | Execute pending withdrawals after dispute period | All validator hot addresses (per comment at Bridge2.sol:50) | Any single finalizer | Bridge2.sol:338 |

There is no `owner()`, no role admin, no pause/upgrade key, and no EOA that can bypass the validator-signature gating.

#### L1 roles — 24 active validators (full list, 2026-04-20)

Data: `POST https://api.hyperliquid.xyz/info` `{"type":"validatorSummaries"}` → 30 entries, filtered to `isActive=true && isJailed=false` → 24.

| # | Validator | Cold address (`validator`) | Hot address (`signer`) | Stake (HYPE) |
|---|-----------|---------------------------|-----------------------|-------------:|
| 1 | Hyper Foundation 2 | `0xa82fe73bbd768bc15d1ef2f6142a21ff8bd762ad` | `0xa82fe73bbd768bc15d1ef2f6142a21ff8bd762ad` | 56,676,658.31 |
| 2 | Hyper Foundation 3 | `0x80f0cd23da5bf3a0101110cfd0f89c8a69a1384d` | `0x80f0cd23da5bf3a0101110cfd0f89c8a69a1384d` | 55,486,946.62 |
| 3 | Hyper Foundation 1 | `0x5ac99df645f3414876c816caa18b2d234024b487` | `0x5ac99df645f3414876c816caa18b2d234024b487` | 53,212,837.28 |
| 4 | Hyper Foundation 4 | `0xdf35aee8ef5658686142acd1e5ab5dbcdf8c51e8` | `0xdf35aee8ef5658686142acd1e5ab5dbcdf8c51e8` | 53,144,941.92 |
| 5 | Anchorage By Figment | `0x420a4ed7b6bb361da586868adec2f2bb9ab75e66` | `0x9dd85fb6cf95a9d755af0853b36fb05e972c71e1` | 34,685,500.38 |
| 6 | Nansen x HypurrCollective | `0xb8f45222a3246a2b0104696a1df26842007c5bc5` | `0x4a900e9266b00e3d5310e09bb49520b80f2bd417` | 24,518,715.49 |
| 7 | Hypurrscanning | `0xabcdeff4b3727b83a23697500eef089020df2cd2` | `0xb796a00b6e50c3dd46e43346c921fe8e146f4e06` | 23,223,561.95 |
| 8 | infinitefield.xyz | `0xa23b4556090260828ff3f939d2dbdd4f318b5f1f` | `0xe02dc4639abce8f93adb7f36244a309f5aacada5` | 19,404,766.19 |
| 9 | Hyper Foundation 5 | `0x66be52ec79f829cc88e5778a255e2cb9492798fd` | `0x5795ab6e71ecbefa255fc4728cc34893ba992d44` | 15,007,172.34 |
| 10 | HyperStake | `0x8b8c3966870321866e7b7091c382308a6a97e9b1` | `0x9c1d5d05b38ad27fb143697bad0fbd310380209c` | 12,045,087.98 |
| 11 | Kinetiq x Hyperion | `0xeeee86f718f9da3e7250624a460f6ea710e9c006` | `0x8c323b484d301933cd7a11c12bebd173f36933e4` | 10,057,162.49 |
| 12 | USDT0 x Luganodes | `0x48f1da3e3ec2814fbb3dcf57125001089b067402` | `0x8ad18589986bdd4cee7387ca2dca2d5cc1615d1a` | 7,883,942.88 |
| 13 | Imperator.co — HypeRPC.app | `0x8a5dbdf69b282bf2e8fb9f29fd34891f79c5dfd4` | `0x8a5dbdf69b282bf2e8fb9f29fd34891f79c5dfd4` | 7,860,674.35 |
| 14 | ASXN | `0xe45c96a6a32318e5df7347477963bf0de38ff7ff` | `0xe76c2879bd01de4c621b8518922eb5d8ede9fbb5` | 7,454,405.43 |
| 15 | Alphaticks | `0x3e5b2598a32ebf003ad5a7254faa3d04ff41d9fe` | `0xc334c02fdbff90180331dd539db4f528ea29a603` | 6,627,481.25 |
| 16 | ValiDAO | `0x000000000056f99d36b6f2e0c51fd41496bbacb8` | `0x0000000008b0b558419582041f85740344ae8fde` | 5,478,638.26 |
| 17 | B-Harvest | `0x15458aed3c7a49b215fbfa863c6ff550c31e1a31` | `0x21d50a1c2e70b2b4b25516c744da7f1de760b2ec` | 5,015,313.90 |
| 18 | HypurrCorea: SKYGG x DeSpread | `0x65baa675fa9e5f6c7ae4541ebdb16c526de06f1f` | `0xd9333fe13abccaf1a6982c500bcab986e7d53652` | 4,957,292.52 |
| 19 | CMI | `0x4e256d24da830290d10f425b44f3e9439394385a` | `0x2ac34ef759933b5f695f6d4faba17045f8837a74` | 4,945,974.77 |
| 20 | Purrposeful x HyBridge x PiP | `0xf8efb4cb844a8458114994203d7b0bfe2422a288` | `0xc71557db875380d09a23ce909dc07a80a17f9956` | 4,749,999.68 |
| 21 | Bitwise Onchain Solutions x FalconX | `0x30c66ebc7f5ef4f340b424a26e4d944f60129815` | `0xc304bcea88f450f3367bf5df3eea30eef4f0e9e9` | 4,519,086.99 |
| 22 | Hyperbeat x P2P x Hypio | `0x497beec89958848126c2ea65934ce430e1410ad2` | `0x497beec89958848126c2ea65934ce430e1410ad2` | 4,023,461.83 |
| 23 | Liquid Spirit x Hydromancer x Rekt Gang | `0xb00c116f72eb55f52ca80196b63014a42cc72de1` | `0xee783c36ef385640041e9f6ff91666ed42c0d831` | 4,007,685.33 |
| 24 | Flowdex | `0x8f02ade62c1c1cf34daa855ccf1245aaf90d3056` | `0xeeabeda825763993115fd3687edabefa0e17c619` | 3,975,430.12 |
| | **Total active stake** | | | **428,962,738.24** |

**Currently jailed (4):** HyperCN X hlscan (`0x68e6b8995d0ea3ebe829bd66e9437eaf536aff9c`), Red Pond (`0xb01934de2e25e57a6a91b3411a7544be2e06f392`), cp0x by STAKR.space (`0xc56d0c8ae0c387b439226387c54dbedb0aededfb`), GalaxyDigital (`0xc75a3fc98b0e1af7a95b6a720adf2e23806d2c7b`). Per the validator-jailing documentation (https://hyperliquid.gitbook.io/hyperliquid-docs/validators), validators can be jailed for consecutive missed blocks or by a 2/3 stake-weighted vote, and must wait past `unjailableAfter` before re-joining. The underlying jailing logic lives in the closed-source L1 binary, so exact missed-block thresholds and slashing amounts cannot be verified at the source level.

#### Non-onchain roles

| Role | Function | Holder | Source of authority |
|------|----------|--------|---------------------|
| Foundation delegation program | Redirects Foundation stake to promote validator decentralisation | Hyper Foundation | Discretionary — docs state *"The Foundation reserves the right to cease delegation at any time"* (https://hyperliquid.gitbook.io/hyperliquid-docs/validators/delegation-program) |
| Binary release author | Produces the HyperCore `hl-node` binary that validators run | Hyperliquid Labs / Foundation | Not enforced onchain; validators choose which release to run, but the binary is closed-source |
| Trademark holder | Owns "HYPERLIQUID" mark | Hyper Foundation (USPTO 99599981) | See §5.1 |

**Finding:** Bridge2 roles are fully enforced in open source. L1 roles exist (validators, jailing via 2/3 vote, fee routing) and are observable via the info API, but their exact implementation is in a closed-source binary and no onchain role is held by tokenholders directly. The Foundation controls 54.44% of active stake and retains unilateral discretion over the delegation program.

### 1.3 Protocol Upgrade Authority

**Status: NEUTRAL**

**Bridge2 (Arbitrum):**
- Non-upgradeable (no proxy pattern)
- Validator set updates require 2/3 stake-weighted signatures
- No admin can unilaterally upgrade

**HyperCore L1:**
- Upgrades via validator consensus
- Code is closed source - upgrade mechanism unverifiable from source
- Tokenholders do not hold validator keys directly; influence is indirect via delegation (see 1.2). Aragon has not been able to verify any onchain mechanism for tokenholders (non-validators) to veto or trigger upgrades.

**HyperEVM:**
- Shares consensus with HyperCore
- Same validator set controls both components

**Finding:** Bridge2 is non-upgradeable. L1 upgrades are via validator consensus but the mechanism is closed source.

### 1.4 Token Upgrade Authority

**Status: NEUTRAL**

#### Native HYPE
- HYPE is an L1-native token — there is no ERC-20 contract on the L1, no proxy, no `owner()`, no `upgradeTo()`, and no admin multisig.
- The rules that govern HYPE (balances, mint/burn, staking emissions, fee routing, Assistance Fund behaviour) are defined by the HyperCore `hl-node` binary distributed at https://github.com/hyperliquid-dex/node. That repo is binaries-only under Apache 2.0.
- Changing those rules requires a new binary release that ≥2/3 of stake-weighted validators adopt. No single actor can unilaterally upgrade HYPE's logic, and no tokenholder-controlled smart contract can force validators to adopt a particular release. See Contract Architecture for the full explanation.

#### WHYPE (HyperEVM)

**Address:** `0x5555555555555555555555555555555555555555`
**Contract name:** `WHYPE9` (per official Hyperliquid docs and HyperEVMScan source-verification page)

**Upgrade surface:**
- No proxy pattern (verified: HyperScan API returns `proxy_type: null`, `minimal_proxy_address_hash: null`; onchain bytecode is ~2.3 KB, matching the WETH9 implementation, not an EIP-1967 proxy stub).
- No `owner()`, no `DEFAULT_ADMIN_ROLE`, no `upgrader` — the WETH9 pattern does not include any of these.
- No mint function other than `deposit()` (1:1 with native HYPE sent in) and no burn other than `withdraw()`.

**Why docs name it `WHYPE9` while HyperScan shows `WCTC`:**
- The Hyperliquid GitBook (https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype) calls the contract `WHYPE9` and states it is the same source as WETH9 with only the name/symbol changed.
- HyperEVMScan (`https://hyperevmscan.io/address/0x5555555555555555555555555555555555555555#code`) is the Etherscan-style explorer the docs refer to and shows verified source under the name `WHYPE9`.
- The Blockscout-based HyperScan indexer (`https://www.hyperscan.com/...`) returns `is_verified: true` but with the contract name `WCTC` (Wrapped Core Token Contract) — almost certainly a stale/initial verification label predating the `WHYPE9` rename.
- **Authoritative cross-check (onchain `eth_call`, 2026-04-20):** `name()` → `"Wrapped HYPE"`, `symbol()` → `"WHYPE"`, `decimals()` → `18`. This confirms the deployed bytecode is the renamed WETH9 variant that the docs call `WHYPE9`, regardless of what metadata each explorer cached.

**Compiler:** `v0.5.17+commit.d19bba13` with optimizer enabled (source: HyperScan API response for this address).

**Finding:** HYPE's semantics change only when ≥2/3 of validator stake adopts a new L1 binary — no smart-contract owner can force it. WHYPE is an immutable WETH9-clone with no proxy, no admin, and no mint/burn beyond deposit/withdraw. The HyperScan explorer's `WCTC` label is a cached artefact; the canonical source-verified name per the Hyperliquid docs and HyperEVMScan is `WHYPE9`, and this matches the onchain `name()` / `symbol()`.

### 1.5 Supply Control

**Status: NEUTRAL**

**Max Supply:** 1,000,000,000 HYPE (fixed genesis; no mint function — confirmed by the closed-source L1 having no documented or observed inflation events beyond the pre-scheduled staking-emission drip from the 38.89% future-emissions allocation).

#### Verified supply state (2026-04-20)

`POST https://api.hyperliquid.xyz/info` `{"type":"tokenDetails","tokenId":"0x0d01dc56dcaaca66ad901c959b4011ec"}`:

| Metric | Value |
|--------|------:|
| Max supply | 1,000,000,000.00 HYPE |
| Total supply | 999,252,738.37 HYPE |
| Circulating supply | 298,873,248.18 HYPE |
| Delta (genesis − total) = burned | **747,261.63 HYPE** |

`POST https://api.hyperliquid.xyz/info` `{"type":"spotClearinghouseState","user":"0xfefefefefefefefefefefefefefefefefefefefe"}`:

| Metric | Value |
|--------|------:|
| Assistance Fund HYPE balance | 43,459,601.14 HYPE |
| Entry notional | $1,066,893,148.58 |

#### Two complementary burn channels

**Channel 1 — Hard burn (total supply reduction).** 747,261.63 HYPE has been mechanically destroyed (`totalSupply` reduced from 1,000,000,000 to 999,252,738.37). This is the "canonical" burn and is reflected in the info API's `tokenDetails` response.

**Channel 2 — Burner-address recognition (Assistance Fund, 43.46M HYPE).** On **Dec 17-24, 2025**, Hyperliquid validators held a stake-weighted governance vote to **formally designate the Assistance Fund address `0xfefefefefefefefefefefefefefefefefefefefe` as a burner address**. Per the Hyper Foundation's official announcement:

> *"HYPE in the Assistance Fund system address of `0xfefefefefefefefefefefefefefefefefefefefefe` has been formally recognized as burned. The governance vote was based on stake-weighted consensus, with 85% of stake voting for burning, 7% against, and 8% abstaining."*

Source: Hyper Foundation X post, https://x.com/HyperFND/status/2003684257949188396.

The effect is that the 43,459,601.14 HYPE sitting in `0xfefe…fefe` is **treated as burned by protocol convention** even though `tokenDetails.totalSupply` still includes it. This explains the (otherwise-confusing) gap between the hard-burned figure (747K) and the community reporting of "~44M HYPE burned": the difference is Channel 2.

#### Supply-control authority

| Capability | Who can trigger it | How |
|------------|-------------------|-----|
| Mint new HYPE beyond the 1B cap | **No one** — there is no mint function in the documented L1 rules; staking emissions are drawn from the pre-allocated 38.89% future-emissions bucket, not newly minted | L1 binary rules |
| Hard-burn (reduce `totalSupply`) | Automated at L1; driven by fee flow into the AF and the AF's burn mechanics | Closed-source L1 binary; not directly controllable by tokenholders or Foundation |
| Recognise an address as a burner (Channel 2) | Validators via stake-weighted vote | Stake-weighted governance vote (demonstrated by the Dec 2025 AF vote above) |
| Freeze / confiscate balances | Not documented anywhere; no public evidence of such a facility | Cannot be source-verified due to closed-source L1 |

**Finding:** The supply is capped and there is no mint path. Two distinct burn channels are both active: 747,261.63 HYPE hard-burned, plus 43.46M HYPE in the Assistance Fund formally recognised as burned by an 85% stake-weighted validator vote on Dec 17-24, 2025. Supply-control authority therefore sits with validators (Channel 2) and the L1 protocol itself (Channel 1); tokenholders participate only indirectly via their delegated validators. The continuing hard-burn cadence (i.e. when AF holdings are mechanically destroyed) is not documented.

### 1.6 Privileged Access Gating

**Status: WARNING**

#### Bridge emergency-lock (Arbitrum — source-verified)

Relevant Bridge2 logic (https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol):

```solidity
// Bridge2.sol:746-754
function voteEmergencyLock() external {
    require(lockers[msg.sender], "Sender is not authorized to lock smart contract");
    require(!hasVotedLock[msg.sender], "Locker has already voted to lock");
    lockersVotingLock.push(msg.sender);
    hasVotedLock[msg.sender] = true;
    if (uint64(lockersVotingLock.length) >= lockerThreshold && !paused()) {
        _pause();
    }
}
```

- Only addresses in the `lockers` mapping can call this (Bridge2.sol:747).
- `L1 operation will automatically register all validator hot addresses as lockers` (Bridge2.sol:43).
- Once `lockerThreshold` votes accumulate, `_pause()` fires and Bridge2 is paused — new deposits/withdrawals are halted until `emergencyUnlock(...)` is called with a 2/3 cold-signature quorum (Bridge2.sol:773).
- **Consequence:** any one validator hot key can *vote* to lock, but a single vote does not lock — the threshold is stake-weighted and set by the L1 via `setLockerThreshold(...)`. There is no admin key that can bypass this gating.

#### Unstaking queue (HyperCore)

Per the staking documentation (https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking):
- **Delegation lockup:** 1 day minimum between delegate and un-delegate.
- **Unstaking queue:** 7 days from the L1 unstake action to HYPE becoming transferable spot balance again.
- Users therefore cannot exit an adversarial validator faster than ~7-8 days combined.

#### Validator jailing (HyperCore — closed-source, documented behaviour)

Per https://hyperliquid.gitbook.io/hyperliquid-docs/validators, validators can be jailed either:
1. **Automatically** for insufficient uptime / consecutive missed blocks; or
2. **By validator vote** — stake-weighted, with the `validatorSummaries` API exposing the `isJailed` and `unjailableAfter` fields that result.

Current jailed set (2026-04-20, info API): 4 of 30 — see 1.2 for the list. Exact missed-block thresholds and slashing formulas live in the closed-source binary and Aragon has not been able to verify them at the source level.

#### HIP-3 operator slashing (stake-weighted governance vote)

HIP-3 (https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals) introduces operator slashing for builder-deployed perpetual markets. The HIP specifies that slashing is authorised by a stake-weighted validator vote — another Channel-B access-gating mechanism that is, by design, controlled by the validator set rather than tokenholders directly.

**Finding:** Access gating is multi-layered: (a) bridge lockers (any validator hot key can cast a vote; stake-threshold triggers the pause; cold-key 2/3 required to unlock — all verified in Bridge2.sol), (b) 7-day unstaking queue, (c) validator jailing (automatic and by 2/3 vote; details in closed-source binary), and (d) HIP-3 operator slashing (stake-weighted validator vote). None of these gates is held by a single EOA or multisig; each is controlled by the validator set, in which the Foundation holds 54.44% of active stake.

### 1.7 Token Censorship

**Status: NEUTRAL**

This criterion covers whether any party can freeze, blacklist, or prevent transfer of HYPE for an individual holder. Bridge2 custodies USDC (not HYPE), so its pause surface is not relevant to HYPE censorship — it is covered in 1.6 as access gating, not here.

#### Native HYPE (L1, HyperCore)
- The HyperCore L1 binary is closed source (https://github.com/hyperliquid-dex/node distributes only binaries), so no blacklist / freeze function can be inspected at source level.
- Public documentation (https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking, .../hypercore/unit, .../trading/fees) does **not** document any blacklist, freeze, or transfer-restriction facility for HYPE.
- The public info API does not expose any "frozen" flag on user balances, and no censored-account incidents have been publicly reported or observed as of the verification date.
- Aragon has not been able to independently verify from source whether a latent censorship capability exists; only observable behaviour (no known censored HYPE balances) is available.

#### WHYPE (HyperEVM, 0x55…55)
- WHYPE is a WETH9 clone (see 1.4). The WETH9 pattern has **no** blacklist, no freeze, no pausable modifier, and no admin — `transfer`, `transferFrom`, `deposit` and `withdraw` are the only state-changing functions and none can be blocked by any external party.
- Aragon has cross-checked this by confirming the contract has no `owner()`, no `paused` state, and no role-based access control (onchain calls to `owner()` and `paused()` revert / return zero-bytes, consistent with the WETH9 ABI).

**Finding:** There is no documented or onchain-observable transfer-censorship mechanism for HYPE or WHYPE. WHYPE is structurally censorship-resistant (WETH9 clone with no admin). For native HYPE, the closed-source L1 means censorship capability cannot be ruled out at source level, but no such capability is documented and no incidents have been observed.

---

## Metric 2: Value Accrual

### 2.1 Accrual Active

**Status: NEUTRAL**

Three distinct mechanisms concurrently direct value to HYPE holders; each is verifiable onchain.

#### Mechanism 1 — Fee buyback → Assistance Fund → burn

**Source (fee docs, official):** https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees

> *"Fees are entirely directed to the community (HLP, the assistance fund, and deployers)."*
> *"The assistance fund uses the system address 0xfefefefefefefefefefefefefefefefefefefefe. It converts trading fees to HYPE in a fully automated manner as part of the L1 execution. HYPE in the assistance fund is burned, removing the tokens permanently from the circulating and total supply."*
> *"Spot and HIP-3 perp deployers may choose to keep up to 50% of trading fees generated by their deployed assets."*

**Verified onchain state (2026-04-20):**

| Metric | Value | Source |
|--------|------:|--------|
| Hard-burned HYPE (`totalSupply` reduction) | 747,261.63 HYPE | `info` → `tokenDetails` |
| HYPE in AF (formally recognised as burned per Dec 2025 vote) | 43,459,601.14 HYPE (~$1.07B notional) | `info` → `spotClearinghouseState` |
| **Total HYPE effectively removed from liquid supply** | **~44.21M HYPE** | sum of above |

As documented in 1.5, validators passed a stake-weighted vote on Dec 17-24, 2025 (85% for / 7% against / 8% abstain; https://x.com/HyperFND/status/2003684257949188396) to formally treat `0xfefe…fefe` as a burner address, so the AF-held balance is considered burned by protocol convention even before the underlying `totalSupply` is mechanically reduced.

#### Mechanism 2 — Staking rewards (~2.37% APY)

- Source: the 38.89% future-emissions bucket (see 4.1); no new HYPE beyond the 1B cap.
- Verified via validator `predictedApr` fields in `validatorSummaries` — representative validators show `predictedApr ≈ 0.02160` (2.16%); after 4% typical commission, delegator APY lands in the ~2.0-2.4% range, consistent with docs.
- Rewards are distributed daily and auto-compound at the L1 level.

#### Mechanism 3 — Fee discounts for HYPE stakers

Source: https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees (staking tiers section).

| Tier | HYPE Staked | Trading-fee discount |
|------|-------------|----------------------|
| Wood | 1,000 | 5% |
| Bronze | 10,000 | 10% |
| Silver | 100,000 | 20% |
| Gold | 1,000,000 | 30% |
| Diamond | 10,000,000 | 40% |

#### Base fee schedule (for context)

| Fee type | Base rate | Lowest tier |
|----------|-----------|-------------|
| Perps taker | 0.045% | 0.024% |
| Perps maker | 0.015% | 0.000% |
| Spot taker | 0.070% | 0.025% |
| Spot maker | 0.040% | 0.000% |

**Finding:** All three value-accrual mechanisms are active and verifiable. The effective burn — hard-burn (747K HYPE) plus validator-recognised AF balance (43.46M HYPE) — totals ~44.2M HYPE, i.e. ~4.42% of genesis supply, driven by real trading-fee flow.

### 2.2 Treasury Ownership

**Status: NEUTRAL**

The HYPE ecosystem has two distinct "treasuries" that must not be conflated: the **Assistance Fund** (an automated protocol-level mechanism) and the **Hyper Foundation Budget** (a discretionary off-protocol allocation).

#### Assistance Fund — `0xfefefefefefefefefefefefefefefefefefefefe`

- A system address on the L1, not a smart contract on HyperEVM. No admin/owner. Automated fee-routing and buyback logic sits in the closed-source L1 binary.
- **Holdings (2026-04-20):** 43,459,601.14 HYPE plus minor balances in USDC/TRUMP/VAPOR/MEOW etc. (from buyback-in-progress swaps).
- **Status per Dec 2025 validator vote:** balances at this address are formally recognised as burned (see 1.5 / 2.1).
- Tokenholders have no direct control over AF inflows, outflows, or asset composition — all are set by the L1 protocol.

#### Hyper Foundation Budget — 6.00% of genesis (60,000,000 HYPE)

- **Purpose (per Hyperliquid tokenomics docs):** funds Foundation operations, including the validator delegation program, community grants, ecosystem initiatives, ongoing protocol development, marketing, and general Foundation expenditure.
- **Control:** fully discretionary — held and deployed by the Hyper Foundation with no tokenholder vote or binding oversight. No onchain transparency requirement or scheduled report is documented.
- **Location:** the Foundation has not publicly disclosed a single canonical treasury address, and Aragon has not been able to identify one via the info API. A substantial portion of the Foundation budget is observable as **delegated stake** — the 5 Hyper Foundation validators hold a combined 233,528,556.47 HYPE (see 1.1/1.2), much of which is self-delegation of the Foundation budget and/or related allocations used to bootstrap validator security. Aragon has not been able to verify the precise split between "Foundation budget (6%)" and other Foundation-controlled allocations in the staked amount.

#### Where is the 63% "not in genesis and not in Foundation budget"?

Accounting the 1B supply:

| Allocation | % | Status / location |
|-----------|---|-------------------|
| Genesis distribution (airdrop recipients) | 31.00% | Released; held by ~94K individual claimant addresses at TGE |
| Community grants | 0.30% | Released |
| HIP-2 hyperliquidity | 0.01% | Released |
| Core contributors | 23.80% | Cliff-vesting program; locked until phased unlocks (next 9.92M HYPE on 2026-05-06 — see 4.2) |
| **Future emissions** | **38.89%** | **Reserved in-protocol; released only through the staking-rewards emission schedule** |
| Hyper Foundation budget | 6.00% | Foundation-controlled; partially observable as delegated stake via Foundation validators |
| **Total** | **100.00%** | |

The "63%" the human reviewer referred to is primarily the **38.89% future-emissions** (locked in-protocol, unlocked only via the staking-reward drip), plus the **23.80% core-contributors** bucket (locked under cliff vesting — see 4.2). The remainder (0.31% community grants + HIP-2) is released but small. In aggregate, the 63% non-genesis-non-Foundation-budget supply is:
- **38.89%** → future-emissions reserve (protocol-locked)
- **23.80%** → core-contributor cliff-vesting (scheduled unlocks)
- **0.31%** → grants + HIP-2 (released)

**Source for the split:** https://tokenomist.ai/hyperliquid (mirrors the Hyperliquid tokenomics docs).

**Finding:** The Assistance Fund is an automated protocol-level treasury under no one's discretionary control. The Foundation Budget (6%) is fully discretionary to the Foundation and not subject to tokenholder governance, but comprises only 6% of supply. The other 63% not claimed at genesis is primarily (a) future-emissions held in-protocol and released via staking rewards and (b) cliff-vesting core-contributor tokens — neither is a "treasury" in the discretionary-governance sense.

### 2.3 Accrual Mechanism Control

**Status: WARNING**

This criterion asks: *who can change the parameters of the value-accrual mechanisms described in 2.1?*

| Parameter | Controller | Change mechanism | Source |
|-----------|------------|------------------|--------|
| Spot/perps taker-maker fee rates | L1 protocol (closed-source binary) | A new binary release adopted by ≥2/3 validator stake could change these rates; no smart-contract-level change possible | https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees |
| Assistance Fund share of fees | L1 protocol (closed-source) | Same as above; validators must adopt a binary that routes a different share to `0xfefe…fefe` | https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees |
| Deployer share (up to 50% for spot/HIP-3 deployers) | Set per-market by the deployer within the docs-specified 0–50% cap | Deployer contract configuration; cap itself is an L1-binary rule | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-1-native-token-standard and https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals |
| AF → burn execution (Channel 1) | L1 protocol (closed-source) | Binary-release change | https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees |
| AF → burn recognition (Channel 2, burner-address status) | Validator stake-weighted vote | Demonstrated by Dec 2025 vote (85%/7%/8%) | https://x.com/HyperFND/status/2003684257949188396 |
| HIP-3 perp operator slashing | Validator stake-weighted vote | HIP-3 protocol spec | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals |
| Staking emission APY | L1 protocol (closed-source); depends on future-emissions balance and total stake | Binary-release change | https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking |
| Fee discount tiers | L1 protocol (closed-source) | Binary-release change | https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees |

**Critical quote — fee routing (primary source):**
> *"Fees are entirely directed to the community (HLP, the assistance fund, and deployers). The assistance fund uses the system address 0xfefefefefefefefefefefefefefefefefefefefe. It converts trading fees to HYPE in a fully automated manner as part of the L1 execution. HYPE in the assistance fund is burned, removing the tokens permanently from the circulating and total supply. Spot and HIP-3 perp deployers may choose to keep up to 50% of trading fees generated by their deployed assets."*
> — https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees

**On the "97%" secondary figure:** Some secondary reporting (e.g. DL News) cites "97% of fees to HYPE buyback/burn." The primary GitBook does not give a precise percentage; it only says fees are split between HLP, the AF, and deployers. Aragon does not rely on the 97% figure.

**Finding:** Every accrual parameter is either (a) set in the closed-source L1 binary and changeable only by ≥2/3 of validator stake adopting a new release, or (b) set by a stake-weighted validator governance vote. In both cases the gate is the validator set, in which the Hyper Foundation holds 54.44% — so any rate change or routing change can be passed or blocked by the Foundation. Non-validator tokenholders have no direct entry point and must rely on delegation.

### 2.4 Offchain Value Accrual

**Status: NEGATIVE** (no offchain accrual to tokenholders; value retained by Labs/Foundation)

Aragon investigated the following channels; none route offchain value to HYPE holders:

| Channel | Finding | Verification |
|---------|---------|--------------|
| Subscription/API revenue | The Hyperliquid API is free; no subscription revenue | https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api |
| Corporate revenue sharing | No documented profit-share from Hyperliquid Labs Pte. Ltd. (Singapore UEN 202402326K) or the Hyper Foundation to HYPE holders | No document identified in the GitBook, X, or Tracxn company profile |
| Dividends / stock in Labs | Labs is a private Singapore "Exempt Private Company Limited by Shares"; shares held by the founding team, not by HYPE holders | Singapore ACRA records via Tracxn |
| Trademark licensing revenue | "HYPERLIQUID" mark (USPTO 99599981) held by Hyper Foundation; Aragon has not been able to identify any public licensing arrangement or revenue-sharing that routes funds to HYPE holders | USPTO filing |
| Ecosystem grants / airdrops to holders | Genesis airdrop already complete; no documented ongoing distribution from offchain entities to holders | Tokenomics docs |

**Finding:** All offchain revenue and value (interface ad revenue, API monetisation if ever introduced, trademark licensing, Labs' private-company equity) accrues to Hyperliquid Labs and the Hyper Foundation, not to HYPE holders. All tokenholder value accrual occurs onchain via the mechanisms in 2.1 (fee buyback/burn, staking rewards, fee discounts). This is a NEGATIVE for the criterion — Aragon did investigate this channel and found no value flows.

---

## Metric 3: Verifiability

### 3.1 Token Contract Source Verification

**Status: NEUTRAL**

#### WHYPE (HyperEVM, `0x5555555555555555555555555555555555555555`)

- **Canonical source-verification page:** https://hyperevmscan.io/address/0x5555555555555555555555555555555555555555#code — shows the contract verified under the name `WHYPE9`, matching the naming used in the Hyperliquid GitBook.
- **Alternative explorer (Blockscout):** https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555 — `is_verified: true`, compiler `v0.5.17+commit.d19bba13`, optimization enabled, `proxy_type: null`. The name on this explorer is the stale label `WCTC`; the onchain `name()`/`symbol()` calls confirm "Wrapped HYPE" / "WHYPE", consistent with the HyperEVMScan `WHYPE9` source.
- **Docs:** https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype — confirms WHYPE is the WETH9 source with only the name/symbol modified, deployed at `0x55…55`, immutable.

**Onchain behaviour cross-check (2026-04-20):** `eth_call name() → "Wrapped HYPE"`, `eth_call symbol() → "WHYPE"`, `eth_call decimals() → 18`, `eth_call totalSupply()` returns a non-zero balance consistent with wrapped HYPE in circulation.

#### Native HYPE (HyperCore L1)

- HYPE is an L1-native asset with no deployed ERC-20 contract at any EVM address; there is therefore no "contract source" to verify in the ERC-20 sense.
- The L1 execution binary at https://github.com/hyperliquid-dex/node contains **no source**, only compiled binaries under Apache 2.0. Aragon has not been able to independently verify the mint/burn/transfer/stake logic from source.
- Observable behaviour can be cross-checked via the info API (`tokenDetails`, `validatorSummaries`, `spotClearinghouseState`, etc.), which is what the rest of this report relies on.

**Finding:** WHYPE is source-verified on both the Etherscan-family explorer (HyperEVMScan, as `WHYPE9`) and the Blockscout explorer (HyperScan, under the stale name `WCTC`), and matches the onchain `name()`/`symbol()`. Native HYPE logic is not source-verifiable because the L1 binary is closed source; only observable on-chain behaviour is inspectable.

### 3.2 Protocol Component Source Verification

**Status: NEUTRAL**

**Verified Components:**

| Component | Verification Status | Source |
|-----------|---------------------|--------|
| Bridge2.sol | Source available on GitHub | https://github.com/hyperliquid-dex/contracts |
| SDKs | MIT licensed, open source | https://github.com/hyperliquid-dex |
| Node software | Apache 2.0 licensed | https://github.com/hyperliquid-dex/node |

**Unverifiable Components:**

| Component | Issue |
|-----------|-------|
| HyperBFT consensus | Closed source |
| HyperCore execution | Closed source |
| Native HYPE token logic | Embedded in L1 (closed source) |
| Fee parameter storage | L1 state (closed source) |
| Assistance Fund automation | L1 code (closed source) |

**Observable Behavior Tests:**

| Test | Method | Result |
|------|--------|--------|
| Validator set consistency | API query matches Bridge2 epoch | epoch=7 on both |
| Staking rewards | API delegatorRewards query | Rewards distributed |
| Token transfers | Explorer data | Consistent behavior |
| Bridge operations | Arbiscan events | Functioning |

**Finding:** Bridge2 source available; L1 core is closed source.

---

## Metric 4: Token Distribution

### 4.1 Ownership Concentration

**Status: WARNING**

#### Token allocation (genesis)

Source: https://tokenomist.ai/hyperliquid (mirrors Hyperliquid tokenomics docs).

| Category | Allocation | Status |
|----------|-----------:|--------|
| Genesis Distribution (airdrop) | 31.00% | Released at TGE (Nov 2024) |
| Core Contributors | 23.80% | Cliff vesting, monthly unlocks starting Nov 2025 over 24 months |
| Future Emissions | 38.89% | Reserved in-protocol for staking rewards |
| Hyper Foundation Budget | 6.00% | Foundation-controlled (discretionary) |
| Community Grants | 0.30% | Released |
| HIP-2 Hyperliquidity | 0.01% | Released |

#### Verified supply state (2026-04-20)

Source: `POST https://api.hyperliquid.xyz/info {"type":"tokenDetails","tokenId":"0x0d01dc56dcaaca66ad901c959b4011ec"}`:

| Metric | Value |
|--------|------:|
| Max supply | 1,000,000,000 HYPE |
| Total supply (onchain) | 999,252,738.37 HYPE |
| Circulating supply (onchain) | 298,873,248.18 HYPE |
| Hard-burned | 747,261.63 HYPE |
| Held in Assistance Fund (validator-recognised as burned) | 43,459,601.14 HYPE |

#### Staked-supply concentration — onchain proofs

Source: `POST https://api.hyperliquid.xyz/info {"type":"validatorSummaries"}` — 30 entries, 24 active.

**Totals:**
- Total stake across 24 active validators: **428,962,738.24 HYPE**
- Total stake across all 30 entries (incl. jailed/inactive): **434,325,897.61 HYPE** (adds the 5.36M held by the 4 jailed + 2 inactive entries)

**Hyper Foundation validators (see 1.2 for addresses):**

| Foundation validator | Stake (HYPE) | Share of active stake | Share of all-validator stake |
|----------------------|-------------:|----------------------:|-----------------------------:|
| Hyper Foundation 1 | 53,212,837.28 | 12.41% | 12.25% |
| Hyper Foundation 2 | 56,676,658.31 | 13.21% | 13.05% |
| Hyper Foundation 3 | 55,486,946.62 | 12.94% | 12.78% |
| Hyper Foundation 4 | 53,144,941.92 | 12.39% | 12.24% |
| Hyper Foundation 5 | 15,007,172.34 | 3.50% | 3.46% |
| **Foundation total** | **233,528,556.47** | **54.44%** | **53.77%** |

**Top-5 concentration (active):** 4 Foundation + Anchorage By Figment (`0x420a4ed7…`) = **253,521,384.11 HYPE** = **59.10%** of active stake. (If Hyper Foundation 5 replaces Anchorage in the top 5, the figure is 54.44%.)

**Nakamoto-style thresholds:**
- Minimum validators needed for >33% (finality-block threshold): **3** (HF2 + HF3 + HF1 = 38.56% of active stake).
- Minimum validators needed for >50% (majority): **4** (HF1-4 = 50.95% of active stake).
- Minimum validators needed for >67% (HyperBFT safety): **8** (HF1-4 + Anchorage + Nansen + Hypurrscanning + infinitefield = 68.57% of active stake).

#### Airdrop / genesis distribution

Per multiple secondary analyses of the Nov 2024 genesis drop, the 31% genesis bucket was distributed to ~94,000 qualifying addresses based on points from Hyperliquid closed-alpha testnet usage. Aragon has not been able to access a primary Foundation document enumerating the per-address allocation, so the top-holder-of-non-staked-HYPE distribution cannot be independently verified here.

**Finding:** Onchain validator-stake concentration is verified: the Hyper Foundation directly controls 54.44% of active stake (53.77% of all-validator stake, 233.5M HYPE) across 5 validators, enough to unilaterally pass a simple-majority stake-weighted vote and enough to unilaterally block a 2/3 supermajority. This is the dominant concentration risk for HYPE.

### 4.2 Future Token Unlocks

**Status: WARNING**

#### Core-contributors cliff vesting (23.80% = 238,000,000 HYPE)

**Schedule (per Hyperliquid tokenomics docs, mirrored at https://tokenomist.ai/hyperliquid):**
- Cliff start: **November 2025** (aligned with the 1-year post-TGE mark)
- Vesting length: **24 months** with monthly unlocks on the 6th
- Amount per month: 238,000,000 ÷ 24 ≈ **9,916,667 HYPE / month**
- Next unlock at the time of research: **2026-05-06**, 9,916,667 HYPE (~$406.7M at recent HYPE ≈ $41 price; note the notional figure moves with spot price).

**Offchain proof:** The core-contributor vesting schedule is published in the Hyperliquid tokenomics documentation and on the Tokenomist allocation tracker. Aragon has not been able to obtain a canonical on-chain source (e.g. a public vesting contract address) because core-contributor allocations are tracked at the L1 level — the closed-source binary handles release into transferable balance. The lack of a public onchain vesting contract means:
- Users cannot independently audit the exact per-address schedule, and
- The release mechanism ultimately depends on the L1 binary honouring the published schedule.

#### Future-emissions (38.89% = 388,900,000 HYPE) — staking-reward drip

- **Recipient:** validators and their delegators, pro rata to stake and after commission.
- **Rate:** no fixed schedule published; APR for delegators is currently ~2.0-2.4% (verified via validator `predictedApr` fields). At 2.37% APR on ~429M staked HYPE, this implies roughly 10.2M HYPE/year flowing from the emissions reserve to stakers, though the L1-internal emission curve is not documented and this is a derived estimate.
- Because emissions are a drip rather than cliffed, they do not create concentrated "unlock events" in the vesting sense, but they do dilute holders who are not staking.

#### Foundation Budget (6% = 60,000,000 HYPE)

- No formal vesting schedule is published. The Foundation can deploy the budget at its discretion. A significant portion is currently observable as delegated stake to the Foundation-run validators (see 4.1).

#### Summary of future-supply dynamics

| Source | Amount | Schedule | Proof |
|--------|-------:|----------|-------|
| Core contributors | 238M HYPE | 24 months × 9.92M from Nov 2025 | Tokenomics docs, Tokenomist |
| Future emissions | 388.9M HYPE | Continuous staking-reward drip; not fully documented | Validator `predictedApr` via info API |
| Foundation budget | 60M HYPE (remaining unreleased portion) | No published schedule | Not publicly disclosed |

**Finding:** The next large cliff unlock is 2026-05-06 (9.92M HYPE, ~2.33% of then-released supply). Cliff vesting is offchain/L1-internal, not governed by a public onchain vesting contract, so users must trust the L1 binary to honour the documented schedule. Future emissions and Foundation budget add further dilutive pressure on an ongoing basis.

---

## Metric 5: Offchain Dependencies

### 5.1 Trademark

**Status: WARNING**

**Mark:** HYPERLIQUID
**Applicant:** Hyper Foundation
**Filing:** USPTO 99599981
**Status:** Filed

**Finding:** Trademark is held by Hyper Foundation, not a tokenholder-controlled entity. No documented governance link between tokenholders and Foundation trademark decisions.

### 5.2 Distribution (Primary Interface)

**Status: WARNING**

**Domain:** app.hyperliquid.xyz
**Operator:** Likely Hyperliquid Labs Pte. Ltd.
**Terms:** https://app.hyperliquid.xyz/terms (not analyzed)

**Corporate Entities:**

| Entity | Type | Location | Role |
|--------|------|----------|------|
| Hyperliquid Labs Pte. Ltd. | Private Company | Singapore (202402326K) | Development |
| Hyper Foundation | Foundation | Unknown | Token distribution, trademark |

**Company Details (Hyperliquid Labs Pte. Ltd.):**
- Incorporated: January 16, 2024
- UEN: 202402326K
- Type: Exempt Private Company Limited by Shares
- Address: 3 Pemimpin Drive #06-01, Lip Hing Industrial Building, Singapore 576147
- Activity: Development of software and applications (except games and cybersecurity)

**Sources:**
- [Tracxn Company Profile](https://tracxn.com/d/legal-entities/singapore/hyperliquid-labs-pte.ltd./__5z3-XYkuxaOgiXpiqaVjHMZ90Wj3k0OKsnxTrPhMyo8)
- [Hyperliquid Docs - Core Contributors](https://hyperliquid.gitbook.io/hyperliquid-docs/about-hyperliquid/core-contributors)

**Finding:** Labs controls primary interface. No alternative decentralized interfaces documented.

### 5.3 Licensing

**Status: WARNING**

| Component | License | Owner |
|-----------|---------|-------|
| SDKs | MIT | Open source |
| Node software | Apache 2.0 | Open source |
| Core L1 | Closed source | Presumably Labs |
| Core execution | Closed source | Presumably Labs |

**Finding:** Critical protocol code is closed source with no public IP assignment to tokenholder-controlled entity.

---

## Risk Assessment

### Onchain Control Risks

| Risk | Severity | Evidence |
|------|----------|----------|
| Foundation validator concentration | HIGH | 54.44% of active stake across 5 Foundation validators — sufficient for simple-majority control and to block 2/3 supermajority |
| No binding tokenholder governance | HIGH | No Governor/Timelock contracts; only channels are HyperBFT consensus + stake-weighted validator votes, both gated by validators |
| Discretionary Foundation delegation program | MEDIUM | Docs: *"The Foundation reserves the right to cease delegation at any time"* |
| Closed-source L1 | MEDIUM | hyperliquid-dex/node ships binaries only (Apache 2.0); mint/burn/fee logic not source-auditable |

### Value Accrual Risks

| Risk | Severity | Evidence |
|------|----------|----------|
| Fee parameters not independently verifiable from source | MEDIUM | L1 closed source; Aragon relies on docs and observed on-chain rates |
| No tokenholder control over fee parameters | MEDIUM | Gate is validator set; Foundation holds 54.44% of active stake |
| Hard-burn cadence not documented | LOW | 747K HYPE hard-burned; 43.46M HYPE in AF is now validator-recognised as burned per Dec 2025 vote, mitigating the earlier concern about the AF accumulating indefinitely |

### Distribution Risks

| Risk | Severity | Evidence |
|------|----------|----------|
| Large team allocation (23.8%) | MEDIUM | Cliff vesting creates concentrated events |
| Foundation budget (6%) discretionary | MEDIUM | No transparency reports |
| Future emissions (38.89%) unallocated | LOW | Reserved for staking rewards |

---

## Framework Criteria Summary

### Metric 1: Onchain Control

| Criterion | Status | Notes |
|-----------|--------|-------|
| 1.1 Governance Workflow | **WARNING** | HyperBFT + stake-weighted validator votes (HIP-3, USDH, AF-burner); Foundation 54.44% active stake |
| 1.2 Role Accountability | **WARNING** | Bridge2 roles source-enforced; L1 roles in closed-source binary; Foundation discretionary delegation |
| 1.3 Protocol Upgrade | **NEUTRAL** | Bridge2 non-upgradeable; L1 upgrades require ≥2/3 stake to adopt new binary |
| 1.4 Token Upgrade | **NEUTRAL** | HYPE logic in closed-source L1 binary; WHYPE (`WHYPE9`) immutable WETH9 clone |
| 1.5 Supply Control | **NEUTRAL** | 1B cap; 747K HYPE hard-burned; 43.46M HYPE recognised as burned by Dec 2025 validator vote |
| 1.6 Access Gating | **WARNING** | Bridge lockers (Bridge2.sol:746); 7-day unstaking; validator jailing; HIP-3 operator slashing |
| 1.7 Censorship | **NEUTRAL** | WHYPE is WETH9 clone (no admin); no documented L1 censorship; closed-source means not fully source-verifiable |

### Metric 2: Value Accrual

| Criterion | Status | Notes |
|-----------|--------|-------|
| 2.1 Accrual Active | **NEUTRAL** | ~44.2M HYPE effectively burned (747K hard + 43.46M validator-recognised); ~2.3% staking APR; fee discounts 5-40% |
| 2.2 Treasury Ownership | **NEUTRAL** | AF automated (no discretion); Foundation 6% budget discretionary but small |
| 2.3 Mechanism Control | **WARNING** | All accrual parameters controlled by validator set (L1 binary or stake-weighted vote); Foundation holds 54.44% |
| 2.4 Offchain Accrual | **NEGATIVE** | No offchain value flows to tokenholders identified; all offchain revenue retained by Labs/Foundation |

### Metric 3: Verifiability

| Criterion | Status | Notes |
|-----------|--------|-------|
| 3.1 Token Source | **NEUTRAL** | WHYPE verified on HyperEVMScan (`WHYPE9`) and HyperScan (stale label `WCTC`); native HYPE logic not source-verifiable |
| 3.2 Protocol Source | **NEUTRAL** | Bridge2.sol open; HyperCore binary-only (Apache 2.0) |

### Metric 4: Distribution

| Criterion | Status | Notes |
|-----------|--------|-------|
| 4.1 Concentration | **WARNING** | Foundation: 54.44% active stake (53.77% of all-validator stake); top 5 = 59.10% |
| 4.2 Future Unlocks | **WARNING** | Core-contributor cliff vesting 9.92M HYPE/month from Nov 2025; L1-internal (no public vesting contract); emissions ~10M/yr |

### Metric 5: Offchain Dependencies

| Criterion | Status | Notes |
|-----------|--------|-------|
| 5.1 Trademark | **WARNING** | Hyper Foundation owns mark; no governance link |
| 5.2 Distribution | **WARNING** | Labs controls primary interface |
| 5.3 Licensing | **WARNING** | Core L1 closed source; Labs owns IP |

---

## Conclusion

### What Do HYPE Holders Own?

**Limited Control:** HYPE tokenholders can:
- Delegate to validators — this is the only onchain mechanism to exert influence
- Earn staking rewards (~2.0-2.4% APR, verified via validator `predictedApr`)
- Receive fee discounts (5-40% based on stake tier)
- Benefit from fee-buyback burn: 747K HYPE hard-burned + 43.46M HYPE in AF formally recognised as burned by Dec 2025 validator vote (85%/7%/8%)

**No Direct Control Over:**
- L1 protocol parameters (closed-source binary; changeable only by ≥2/3 of validator stake adopting a new release)
- Fee rates or AF split (same mechanism as above)
- Validator set composition (Foundation holds 54.44%)
- Trademark or IP (Hyper Foundation and Hyperliquid Labs respectively)
- Primary interface `app.hyperliquid.xyz` (operated by Labs)

### Why Should HYPE Have Value?

**Positive:** Active, verifiable value accrual:
1. Fee-driven effective burn of ~44.2M HYPE (4.42% of genesis) — split between hard-burn and validator-recognised AF balance
2. Staking rewards (~2.0-2.4% APR) drawn from the 38.89% future-emissions reserve — diluting non-stakers but rewarding stakers
3. Fee discounts for HYPE stakers (5-40%)

**Concerns:**
- Fee rates and routing parameters are not source-auditable (closed-source L1)
- Tokenholders cannot directly modify accrual parameters — only validators can (via binary adoption or stake-weighted vote), and Foundation holds 54.44%

### What Threatens HYPE Value?

1. **Foundation Concentration:** 54.44% active stake means the Foundation can pass or block stake-weighted votes unilaterally
2. **Closed-Source L1:** Cannot verify token logic, fee handling, or censorship capability
3. **Offchain Dependencies:** Labs controls interface; Foundation controls trademark; no binding governance link
4. **Cliff Vesting:** Core-contributor unlocks 9.92M HYPE/month from Nov 2025 over 24 months, with no public onchain vesting contract
5. **No Binding Governance:** No Governor/Timelock; tokenholders must route influence through validators

### Overall Assessment

HYPE demonstrates active, verifiable value accrual mechanisms. The effective ~44.2M HYPE burn has real on-chain footprint, and staking rewards plus fee discounts reward holders who stake. However, the concentration of validator stake in the Hyper Foundation (54.44%), the closed-source L1 binary, and the absence of any Governor-style onchain voting mean tokenholders must trust the Foundation and its validator majority rather than verify outcomes via enforceable on-chain mechanisms. Unlike protocols with binding onchain governance (AAVE, Curve), HYPE holders cannot directly alter protocol parameters.

---

## Appendix A: Data Sources

| Source | URL | Last Verified |
|--------|-----|---------------|
| Bridge2 source (open) | https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol | 2026-04-20 |
| Node repo (binaries only) | https://github.com/hyperliquid-dex/node | 2026-04-20 |
| Staking docs | https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking | 2026-04-20 |
| Fee docs | https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees | 2026-04-20 |
| Bridge docs | https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/bridge | 2026-04-20 |
| Validator docs | https://hyperliquid.gitbook.io/hyperliquid-docs/validators | 2026-04-20 |
| Delegation program | https://hyperliquid.gitbook.io/hyperliquid-docs/validators/delegation-program | 2026-04-20 |
| WHYPE docs | https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype | 2026-04-20 |
| HIP-1 docs | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-1-native-token-standard | 2026-04-20 |
| HIP-3 docs | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals | 2026-04-20 |
| Info API (validators, balances, tokenDetails, votes) | https://api.hyperliquid.xyz/info | 2026-04-20 |
| AF burner-recognition vote (85% for) | https://x.com/HyperFND/status/2003684257949188396 | 2026-04-20 |
| WHYPE source (Etherscan-family) | https://hyperevmscan.io/address/0x5555555555555555555555555555555555555555#code | 2026-04-20 |
| WHYPE source (Blockscout) | https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555 | 2026-04-20 |
| Bridge2 on Arbiscan | https://arbiscan.io/address/0x2df1c51e09aecf9cacb7bc98cb1742757f163df7#code | 2026-04-20 |
| Tokenomist (allocation breakdown) | https://tokenomist.ai/hyperliquid | 2026-04-20 |
| Labs registration (Singapore) | https://tracxn.com/d/legal-entities/singapore/hyperliquid-labs-pte.ltd./ | 2026-04-20 |
| USPTO trademark 99599981 | https://tsdr.uspto.gov/ (serial 99599981) | 2026-04-20 |

## Appendix B: API Queries Used

**Validator Summaries:**
```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "validatorSummaries"}'
```

**Assistance Fund State (Spot Balances):**
```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "spotClearinghouseState", "user": "0xfefefefefefefefefefefefefefefefefefefefe"}'
# Result: 43,459,601.14 HYPE (entry notional: $1,066,893,148.58)
```

**HYPE Supply Details (Spot):**
```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "tokenDetails", "tokenId": "0x0d01dc56dcaaca66ad901c959b4011ec"}'
# Result: maxSupply 1,000,000,000; totalSupply 999,252,738.37; circulatingSupply 298,873,248.18
```

**WHYPE Source Verification (HyperScan, Blockscout API):**
```bash
curl -s "https://www.hyperscan.com/api/v2/smart-contracts/0x5555555555555555555555555555555555555555"
# Result: is_verified=true; name=WCTC (stale); compiler=v0.5.17+commit.d19bba13; optimization=true; proxy_type=null
# Authoritative name per GitBook + HyperEVMScan: WHYPE9
```

**WHYPE Onchain name/symbol/decimals (HyperEVM JSON-RPC):**
```bash
# name() - 0x06fdde03
curl -s https://api.hyperliquid.xyz/evm -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_call","params":[{"to":"0x5555555555555555555555555555555555555555","data":"0x06fdde03"},"latest"],"id":1}'
# → ABI-decodes to "Wrapped HYPE"
# symbol() - 0x95d89b41 → "WHYPE"
# decimals() - 0x313ce567 → 0x12 (18)
```

**Bridge2 Epoch (Arbitrum JSON-RPC):**
```bash
curl -X POST https://arb1.arbitrum.io/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_call","params":[{"to":"0x2df1c51e09aecf9cacb7bc98cb1742757f163df7","data":"0x900cf0cf"},"latest"],"id":1}'
# Result: epoch = 7; paused = false
```

**L1 Governance Votes (live):**
```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "validatorL1Votes"}'
# Returns currently open stake-weighted validator votes (if any)
```

## Appendix C: Items Aragon Could Not Independently Verify

The following items depend on code or data that is not publicly available. Each is flagged in the body of the report where relevant. Aragon has not been able to independently verify them from source; the corresponding findings rely on observable onchain behaviour and official Hyperliquid documentation.

1. **L1 consensus and execution code.** HyperCore and HyperEVM run a closed-source Rust binary (https://github.com/hyperliquid-dex/node distributes only compiled binaries under Apache 2.0). Aragon has not been able to verify from source: (a) the exact fee split between HLP / Assistance Fund / deployers, (b) the automatic validator-jailing thresholds and slashing amounts, (c) whether any censorship or freeze capability exists at the L1 level, (d) the EIP-1559 base-fee burn behaviour on HyperEVM, and (e) the exact staking-emission curve.

2. **AF → hard-burn cadence (Channel 1).** `totalSupply` has decreased by 747,261.63 HYPE (hard-burn). The Dec 2025 validator vote recognises the 43.46M HYPE in the AF as burned for accounting purposes (Channel 2), but the mechanical cadence by which AF balances are destroyed to reduce `totalSupply` is not documented.

3. **Tokenholder (non-validator) influence on upgrades.** No onchain mechanism has been identified for HYPE holders who are not validators to veto, trigger, or sign off on L1 binary upgrades. Influence is indirect via delegation.

4. **Per-address genesis airdrop allocations and core-contributor vesting.** The Foundation has not published a canonical onchain vesting contract for the 23.80% core-contributor allocation, nor a per-address genesis-airdrop table; these are tracked L1-internally. Aragon therefore cannot independently verify the exact beneficiary addresses or per-month release amounts beyond the aggregate schedule.

5. **Exact split of the 6% Hyper Foundation Budget between "in treasury" and "delegated to Foundation validators".** Aragon has identified the 5 Hyper Foundation validators (233.5M HYPE staked) but has not been able to verify how much of that balance is specifically drawn from the 6% Foundation budget versus other Foundation-adjacent allocations.

These items are flagged here for transparency and are reflected in the status ratings of the relevant sub-sections (notably 1.2, 1.3, 1.6, 1.7, 2.3, 3.2, 4.1, and 4.2).

---

*Report generated by Aragon Research - 2026-04-20*
