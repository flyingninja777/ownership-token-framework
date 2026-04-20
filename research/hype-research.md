# HYPE Token Research Report

**Token:** HYPE (Hyperliquid)
**Network:** Hyperliquid L1 (HyperCore + HyperEVM)
**Research Date:** 2026-04-20
**Framework:** Aragon Ownership Token Framework

---

## Executive Summary

HYPE is the native token of Hyperliquid, an L1 blockchain running HyperBFT consensus with two components: HyperCore (onchain perps/spot orderbooks) and HyperEVM (EVM-compatible smart contracts). The token demonstrates active value accrual through automated fee buybacks and burns via the Assistance Fund. However, significant concerns exist around:

1. **Validator Concentration:** Hyper Foundation controls ~53.74% of staked HYPE across 5 validators
2. **Verifiability:** The L1 code is closed source; only Bridge2 (Arbitrum) and WHYPE (HyperEVM) contracts are verifiable
3. **Governance Limitations:** No onchain governance contracts exist; validators vote via consensus, but Foundation controls majority stake
4. **Offchain Dependencies:** Trademark held by Foundation with no governance link; Labs controls primary interface

**Key Finding:** HYPE tokenholders have indirect influence through delegation but lack direct, enforceable control over protocol parameters. The Foundation's validator stake concentration means governance outcomes are effectively determined by Foundation, not tokenholders.

---

## 1. Contract Architecture

### 1.1 Core Contracts

| Contract | Address | Network | Purpose | Upgradeable |
|----------|---------|---------|---------|-------------|
| HYPE | Native L1 token | Hyperliquid | Protocol native token | N/A (L1 native) |
| WHYPE | `0x5555555555555555555555555555555555555555` | HyperEVM | Wrapped HYPE (ERC-20) | No (immutable) |
| Bridge2 | `0x2Df1c51E09aECF9cacB7bc98cB1742757f163dF7` | Arbitrum | USDC bridge to L1 | No (non-proxy) |

### 1.2 Bridge2 Contract Analysis

**Source:** https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol

The Bridge2 contract manages USDC deposits/withdrawals between Arbitrum and Hyperliquid L1.

**Key Functions:**
- `batchedRequestWithdrawals()` - Validators sign withdrawals (2/3 quorum required)
- `updateValidatorSet()` - Hot wallet signatures update validator set
- `voteEmergencyLock()` - Lockers can pause bridge
- `emergencyUnlock()` - Cold wallet signatures (2/3 quorum) to unlock

**Verified On-Chain State (2026-04-20):**
```
epoch: 7
paused: false
Contract bytecode length: 38790 chars
```

**Bridge2 Security Model:**
1. Withdrawals require 2/3 validator stake-weighted signatures
2. Dispute period allows lockers to pause suspicious withdrawals
3. Cold wallet signatures required to unlock bridge after lock
4. Finalizers (approved addresses) must finalize withdrawals

**Source:** Bridge2.sol lines 126-842 (full contract analysis)

### 1.3 WHYPE Contract

**Address:** `0x5555555555555555555555555555555555555555` (HyperEVM)

Per documentation, WHYPE is:
- Immutable (no upgrade mechanism)
- Same source as WETH on Ethereum mainnet
- 18 decimals, "Wrapped HYPE" name, "WHYPE" symbol

**Source:** https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype

**Verification Status:** Documentation claims immutability. HyperScan verification status could not be independently confirmed via API.

---

## 2. Governance and Ownership Model

### 2.1 Governance Mechanism

Hyperliquid uses **validator stake-weighted consensus** for governance, not smart contract-based voting:

1. **Validators** propose and vote on changes via HyperBFT consensus
2. **Delegators** influence validator power by delegating HYPE
3. **No Governor/Timelock contracts** exist on L1

**Key Difference from EVM Protocols:** Unlike AAVE/UNI where tokenholders vote directly via Governor contracts, HYPE holders can only influence governance indirectly through delegation.

### 2.2 Validator Analysis (Live Data: 2026-04-20)

```
Total validators: 30
Active validators: 24
Jailed validators: 4
Total staked: 434,566,219.98 HYPE
```

**Foundation Validator Concentration:**

| Validator | Address | Stake (HYPE) | % of Total |
|-----------|---------|--------------|------------|
| Hyper Foundation 2 | 0xa82fe73bbd768bc15d1ef2f6142a21ff8bd762ad | 56,713,440 | 13.05% |
| Hyper Foundation 3 | 0x80f0cd23da5bf3a0101110cfd0f89c8a69a1384d | 55,489,038 | 12.77% |
| Hyper Foundation 1 | 0x5ac99df645f3414876c816caa18b2d234024b487 | 53,205,532 | 12.24% |
| Hyper Foundation 4 | 0xdf35aee8ef5658686142acd1e5ab5dbcdf8c51e8 | 53,137,209 | 12.23% |
| Hyper Foundation 5 | 0x66be52ec79f829cc88e5778a255e2cb9492798fd | 15,011,075 | 3.45% |
| **Total Foundation** | | **233,556,294 HYPE** | **53.74%** |

**Verification:** Data queried from `POST https://api.hyperliquid.xyz/info` with `{"type": "validatorSummaries"}`

**Concentration Assessment:**
- No single validator >33% (threshold for blocking finality): **PASS**
- Foundation >50%: **CONCERN** (53.74%)
- Foundation >67% (supermajority control): **NO** (53.74%)
- Top 5 validators (all Foundation): 58.26% of stake

### 2.3 Delegation Program

The Hyper Foundation Delegation Program (https://hyperliquid.gitbook.io/hyperliquid-docs/validators/delegation-program) explicitly states:

> "Delegations will be monitored on an ongoing basis. **The Foundation reserves the right to cease delegation at any time.**"

This means the Foundation maintains discretionary control over which validators receive Foundation stake, independent of tokenholder preferences.

### 2.4 Ownership Topology

```
                    ┌──────────────────────────────────┐
                    │         HYPE Tokenholders        │
                    │      (Hold/Stake/Delegate)       │
                    └──────────────────┬───────────────┘
                                       │
                                       │ Delegate
                                       ▼
                    ┌──────────────────────────────────┐
                    │           Validators             │
                    │  (24 active, 2/3 quorum = 67%)   │
                    └──────────────────┬───────────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              │                        │                        │
              ▼                        ▼                        ▼
   ┌─────────────────┐    ┌─────────────────────┐    ┌─────────────────┐
   │ Foundation      │    │  Independent        │    │  Bridge2        │
   │ Validators      │    │  Validators         │    │  (Arbitrum)     │
   │ 53.74% stake    │    │  46.26% stake       │    │  Same validator │
   └─────────────────┘    └─────────────────────┘    │  set controls   │
                                                      └─────────────────┘
              │
              │ Controlled by
              ▼
   ┌─────────────────────────────────────────────────────────────┐
   │                     HYPER FOUNDATION                         │
   │  - Discretionary delegation                                  │
   │  - No tokenholder governance documented                      │
   │  - Controls trademark (HYPERLIQUID - USPTO 99599981)         │
   └─────────────────────────────────────────────────────────────┘
```

**Critical Finding:** The chain of control terminates at the Hyper Foundation, not tokenholders. While delegators choose validators, the Foundation's 53.74% stake means Foundation-aligned validators control consensus.

---

## 3. Role Matrix

### 3.1 Bridge2 Roles (Arbitrum)

| Role | Function | Current Holder | Control Mechanism | Verified |
|------|----------|----------------|-------------------|----------|
| Hot Wallet Validators | Sign withdrawals, validator updates | 24 active validators | 2/3 stake quorum | Yes - Bridge2.sol:306 |
| Cold Wallet Validators | Unlock bridge, invalidate withdrawals | Same validator set | 2/3 stake quorum | Yes - Bridge2.sol:670 |
| Lockers | Pause bridge for disputes | Validator hot addresses (auto-registered) | lockerThreshold votes | Yes - Bridge2.sol:746 |
| Finalizers | Execute pending withdrawals | Validator hot addresses | Any finalizer | Yes - Bridge2.sol:644 |

**Source:** Bridge2.sol analysis - https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol

### 3.2 L1 Roles (Closed Source - Documentation Only)

| Role | Function | Holder | Verification |
|------|----------|--------|--------------|
| Validator (active) | Produce blocks, vote on consensus | 24 addresses | API: `validatorSummaries` |
| Jailing Authority | Jail underperforming validators | 2/3 validator quorum | Documentation only |
| Fee Parameter Control | Set trading fees | [UNVERIFIED] | L1 closed source |
| Assistance Fund Controller | Automated buyback/burn | System contract | Documentation only |

### 3.3 HyperEVM Roles

| Contract | Role | Holder | Verified |
|----------|------|--------|----------|
| WHYPE | Admin | None (immutable) | Documentation |
| HyperEVM | Consensus | Same validator set as HyperCore | Documentation |

---

## 4. Value Accrual Mechanism

### 4.1 Fee Structure

**Source:** https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees

| Fee Type | Base Rate | Tiers |
|----------|-----------|-------|
| Perps Taker | 0.045% | Down to 0.024% |
| Perps Maker | 0.015% | Down to 0% |
| Spot Taker | 0.070% | Down to 0.025% |
| Spot Maker | 0.040% | Down to 0% |

**Fee Distribution:**
- 97% → Assistance Fund → HYPE buyback → **Burn**
- 3% → HLP Vault
- Up to 50% → Spot/HIP-3 deployers (of their asset's fees)

### 4.2 Assistance Fund

**Address:** `0xfefefefefefefefefefefefefefefefefefefefe` (System address)

**Verified State (2026-04-20):**
```json
{
  "marginSummary": {
    "accountValue": "1000.0",
    "totalRawUsd": "1000.0"
  }
}
```

**Source:** API query `{"type": "clearinghouseState", "user": "0xfefefefefefefefefefefefefefefefefefefefe"}`

**Mechanism:**
1. Fees collected in USDC flow to Assistance Fund
2. L1 execution automatically converts fees to HYPE
3. HYPE is burned (permanently removed from supply)

**Documentation Quote:**
> "HYPE in the assistance fund is burned, removing the tokens permanently from the circulating and total supply."

### 4.3 Staking Rewards

**Source:** https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking

| Parameter | Value |
|-----------|-------|
| Current APY | ~2.37% (at 400M staked) |
| Reward Source | Future emissions reserves |
| Distribution | Daily, auto-recompounded |
| Unstaking Period | 7 days |
| Delegation Lockup | 1 day |

**Formula:** Reward rate inversely proportional to √(total HYPE staked)

### 4.4 Staking Fee Discounts

| Tier | HYPE Staked | Discount |
|------|-------------|----------|
| Wood | 1,000 | 5% |
| Bronze | 10,000 | 10% |
| Silver | 100,000 | 20% |
| Gold | 1,000,000 | 30% |
| Diamond | 10,000,000 | 40% |

### 4.5 Value Accrual Control

**Critical Question:** Who controls the fee parameters and Assistance Fund mechanism?

| Parameter | Controller | Verification |
|-----------|------------|--------------|
| Fee rates | L1 code (closed source) | [UNVERIFIED] |
| Assistance Fund % | L1 code (closed source) | [UNVERIFIED] |
| Burn mechanism | Automated (per docs) | Documentation only |

**Finding:** While documentation claims 97% of fees flow to Assistance Fund for automated burn, the actual parameter control cannot be verified because L1 code is closed source. There is no documented governance mechanism for tokenholders to change these parameters.

---

## 5. Verifiability Assessment

### 5.1 Verified Components

| Component | Verification Status | Source |
|-----------|---------------------|--------|
| Bridge2.sol | Source available on GitHub | https://github.com/hyperliquid-dex/contracts |
| WHYPE | Documentation claims WETH clone, immutable | HyperScan verification not confirmed |
| SDKs | MIT licensed, open source | https://github.com/hyperliquid-dex |
| Node software | Apache 2.0 licensed | https://github.com/hyperliquid-dex/node |

### 5.2 Unverifiable Components

| Component | Issue |
|-----------|-------|
| HyperBFT consensus | Closed source |
| HyperCore execution | Closed source |
| Native HYPE token logic | Embedded in L1 (closed source) |
| Fee parameter storage | L1 state (closed source) |
| Assistance Fund automation | L1 code (closed source) |

### 5.3 Observable Behavior Tests

While L1 code is closed source, certain behaviors can be observed:

| Test | Method | Result |
|------|--------|--------|
| Validator set consistency | API query matches Bridge2 epoch | epoch=7 on both |
| Staking rewards | API delegatorRewards query | Rewards distributed |
| Token transfers | Explorer data | Consistent behavior |
| Bridge operations | Arbiscan events | Functioning |

---

## 6. Token Distribution

### 6.1 Allocation

| Category | Allocation | Status |
|----------|------------|--------|
| Genesis Distribution | 31.00% | Released |
| Core Contributors | 23.80% | Vesting (cliff) |
| Future Emissions | 38.89% | Reserved |
| Hyper Foundation Budget | 6.00% | Foundation controlled |
| Community Grants | 0.30% | Released |
| HIP-2 Hyperliquidity | 0.01% | Released |

**Source:** https://tokenomist.ai/hyperliquid

### 6.2 Supply Metrics (2026-04-20)

| Metric | Value |
|--------|-------|
| Total Supply | 1,000,000,000 HYPE |
| Circulating Supply | 238,385,315 HYPE (23.84%) |
| Unlocked/Released | 425,244,480 HYPE (42.52%) |
| Locked | 574,755,520 HYPE (57.48%) |

### 6.3 Upcoming Unlocks

**Next Unlock:** May 6, 2026
- Amount: 9,916,667 HYPE (~$406.7M at current prices)
- Recipient: Core Contributors
- Impact: 2.33% of released supply

**Vesting Structure:**
- Core contributor vesting: 24 months starting Nov 2025
- Monthly distributions on 6th of each month
- Cliff-based (concentrated unlock events)

### 6.4 Concentration Analysis

**Staked Supply Concentration:**
- Foundation validators: 53.74% of staked HYPE
- Top 5 validators (4 Foundation + 1 independent): 58.26%

**Key Risk:** While genesis was widely distributed, the Foundation's validator stake concentration means governance power is not proportionally distributed among all tokenholders.

---

## 7. Offchain Dependencies

### 7.1 Trademark

**Mark:** HYPERLIQUID
**Applicant:** Hyper Foundation
**Filing:** USPTO 99599981
**Status:** Filed

**Finding:** Trademark is held by Hyper Foundation, not a tokenholder-controlled entity. No documented governance link between tokenholders and Foundation trademark decisions.

### 7.2 Corporate Entities

| Entity | Type | Location | Role |
|--------|------|----------|------|
| Hyperliquid Labs Pte. Ltd. | Private Company | Singapore (202402326K) | Development |
| Hyper Foundation | Foundation | Unknown | Token distribution, trademark |

**Source:** https://www.sgpbusiness.com/company/Hyperliquid-Labs-Pte-Ltd

### 7.3 Primary Interface

**Domain:** app.hyperliquid.xyz
**Operator:** Likely Hyperliquid Labs Pte. Ltd.
**Terms:** https://app.hyperliquid.xyz/terms (not analyzed)

**Finding:** Labs controls primary interface. No alternative decentralized interfaces documented.

### 7.4 Licensing

| Component | License | Owner |
|-----------|---------|-------|
| SDKs | MIT | Open source |
| Node software | Apache 2.0 | Open source |
| Core L1 | Closed source | Presumably Labs |
| Core execution | Closed source | Presumably Labs |

**Finding:** Critical protocol code is closed source with no public IP assignment to tokenholder-controlled entity.

---

## 8. Risk Assessment

### 8.1 Onchain Control Risks

| Risk | Severity | Evidence |
|------|----------|----------|
| Foundation validator concentration | HIGH | 53.74% Foundation-controlled stake |
| No binding tokenholder governance | HIGH | No Governor contracts; delegation only |
| Discretionary Foundation delegation | MEDIUM | "reserves right to cease delegation" |
| Closed source L1 | MEDIUM | Cannot verify token logic or fee handling |

### 8.2 Value Accrual Risks

| Risk | Severity | Evidence |
|------|----------|----------|
| Fee parameter control unverified | MEDIUM | L1 closed source |
| No tokenholder control over fees | MEDIUM | No governance mechanism documented |
| Assistance Fund automation unverified | LOW | Documentation only |

### 8.3 Distribution Risks

| Risk | Severity | Evidence |
|------|----------|----------|
| Large team allocation (23.8%) | MEDIUM | Cliff vesting creates concentrated events |
| Foundation budget (6%) discretionary | MEDIUM | No transparency reports |
| Future emissions (38.89%) unallocated | LOW | Reserved for staking rewards |

---

## 9. Framework Criteria Summary

### Metric 1: Onchain Control

| Criterion | Status | Notes |
|-----------|--------|-------|
| 1.1 Governance Workflow | **WARNING** | Validator voting exists; Foundation controls majority stake |
| 1.2 Role Accountability | **WARNING** | Foundation discretionary delegation; validators can jail peers |
| 1.3 Protocol Upgrade | **NEUTRAL** | Bridge2 non-upgradeable; L1 upgrades via validator consensus (closed source) |
| 1.4 Token Upgrade | **NEUTRAL** | HYPE is native token; WHYPE documented as immutable |
| 1.5 Supply Control | **POSITIVE** | Fixed 1B supply; burns via Assistance Fund |
| 1.6 Access Gating | **WARNING** | Bridge lockers; 7-day unstaking queue; validator jailing |
| 1.7 Censorship | **NEUTRAL** | No blacklist in Bridge2; L1 capabilities unverifiable |

### Metric 2: Value Accrual

| Criterion | Status | Notes |
|-----------|--------|-------|
| 2.1 Accrual Active | **POSITIVE** | Documented buyback/burn via Assistance Fund; staking rewards active |
| 2.2 Treasury Ownership | **NEUTRAL** | Assistance Fund automated; Foundation budget discretionary |
| 2.3 Mechanism Control | **WARNING** | Fee parameters controlled at L1 level; no tokenholder governance |
| 2.4 Offchain Accrual | **UNEVALUATED** | No evidence of offchain value flows to tokenholders |

### Metric 3: Verifiability

| Criterion | Status | Notes |
|-----------|--------|-------|
| 3.1 Token Source | **NEUTRAL** | WHYPE verified (docs); native HYPE unverifiable |
| 3.2 Protocol Source | **NEUTRAL** | Bridge2 available; L1 closed source |

### Metric 4: Distribution

| Criterion | Status | Notes |
|-----------|--------|-------|
| 4.1 Concentration | **WARNING** | Foundation 53.74% validator stake; top 5 = 58.26% |
| 4.2 Future Unlocks | **WARNING** | Cliff vesting; next unlock May 2026 (2.33% impact) |

### Offchain Dependencies

| Criterion | Status | Notes |
|-----------|--------|-------|
| 5.1 Trademark | **WARNING** | Hyper Foundation owns mark; no governance link |
| 5.2 Distribution | **WARNING** | Labs controls primary interface |
| 5.3 Licensing | **WARNING** | Core L1 closed source; Labs owns IP |

---

## 10. Conclusion

### What Do HYPE Holders Own?

**Limited Control:** HYPE tokenholders can:
- Delegate to validators (indirect governance influence)
- Earn staking rewards (~2.37% APY)
- Receive fee discounts (5-40% based on stake)
- Benefit from automated buyback/burn (supply reduction)

**No Direct Control Over:**
- Protocol parameters (closed source L1)
- Fee rates or distribution
- Validator set composition (Foundation controls majority)
- Trademark or IP
- Primary interface

### Why Should HYPE Have Value?

**Positive:** Active value accrual through:
1. Automated fee buyback and permanent burn
2. Staking rewards from emissions
3. Fee discounts for stakers

**Concern:** These mechanisms exist but cannot be verified or controlled by tokenholders.

### What Threatens HYPE Value?

1. **Foundation Concentration:** 53.74% validator stake means Foundation effectively controls governance
2. **Closed Source L1:** Cannot verify critical token logic or fee mechanisms
3. **Offchain Dependencies:** Labs/Foundation control trademark, interface, and IP
4. **Cliff Vesting:** Concentrated unlock events (next: May 2026, ~$407M)
5. **No Binding Governance:** Tokenholders cannot enforce changes via onchain voting

### Overall Assessment

HYPE demonstrates active value accrual mechanisms but exhibits significant centralisation risks. The Foundation's validator stake concentration and the closed-source L1 mean tokenholders must trust rather than verify. Unlike protocols with binding onchain governance (e.g., AAVE, Curve), HYPE holders cannot directly control protocol parameters.

---

## Appendix A: Data Sources

| Source | URL | Last Verified |
|--------|-----|---------------|
| Bridge2 Source | https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol | 2026-04-20 |
| Staking Docs | https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking | 2026-04-20 |
| Fee Docs | https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees | 2026-04-20 |
| Bridge Docs | https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/bridge | 2026-04-20 |
| Delegation Program | https://hyperliquid.gitbook.io/hyperliquid-docs/validators/delegation-program | 2026-04-20 |
| WHYPE Docs | https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype | 2026-04-20 |
| HIP-1 Docs | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-1-native-token-standard | 2026-04-20 |
| Validator API | https://api.hyperliquid.xyz/info | 2026-04-20 |
| Tokenomist | https://tokenomist.ai/hyperliquid | 2026-04-20 |
| Labs Registration | https://www.sgpbusiness.com/company/Hyperliquid-Labs-Pte-Ltd | 2026-04-20 |

## Appendix B: API Queries Used

**Validator Summaries:**
```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "validatorSummaries"}'
```

**Assistance Fund State:**
```bash
curl -X POST https://api.hyperliquid.xyz/info \
  -H "Content-Type: application/json" \
  -d '{"type": "clearinghouseState", "user": "0xfefefefefefefefefefefefefefefefefefefefe"}'
```

**Bridge2 Epoch (Arbitrum JSON-RPC):**
```bash
curl -X POST https://arb1.arbitrum.io/rpc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_call","params":[{"to":"0x2df1c51e09aecf9cacb7bc98cb1742757f163df7","data":"0x900cf0cf"},"latest"],"id":1}'
# Result: epoch = 7
```

## Appendix C: Unverified Claims

The following claims from documentation could not be independently verified:

1. **Assistance Fund automation:** Documentation claims L1 automatically converts fees to HYPE and burns. The mechanism is embedded in closed-source L1 code.

2. **WHYPE immutability:** Documentation claims WHYPE is identical to WETH and immutable. HyperScan verification status could not be confirmed via API.

3. **Fee parameter immutability:** No documentation confirms whether fee parameters can be changed, or by whom.

4. **Jailing mechanism:** Peer voting for jailing is documented but the code is closed source.

5. **EIP-1559 burn on HyperEVM:** Documentation claims base fees are burned. Cannot verify in closed-source code.

---

*Report generated by Aragon Research - 2026-04-20*
