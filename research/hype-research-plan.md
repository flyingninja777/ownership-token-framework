# HYPE Token Research Plan

## Overview

This research plan maps the Aragon Ownership Token Framework to Hyperliquid's HYPE token. The plan provides a structured approach to investigating each framework criterion with specific sources, evidence requirements, and anticipated challenges.

**Key Context:**
- HYPE is the native token of Hyperliquid, an L1 blockchain with HyperBFT consensus
- Hyperliquid has two components: HyperCore (perps/spot orderbooks) and HyperEVM (EVM-compatible smart contracts)
- The validator set uses delegated proof-of-stake with 24 active validators
- The protocol operates without VC funding; 70% of supply is allocated to community

---

## Resource Inventory

### Primary Documentation (Confirmed)

| Resource | URL | Content |
|----------|-----|---------|
| Hyperliquid Docs | https://hyperliquid.gitbook.io/hyperliquid-docs | Official documentation |
| Staking Documentation | https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking | Staking mechanics, rewards, delegation |
| Fees Documentation | https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees | Fee structure, distribution, buybacks |
| Bridge Documentation | https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/bridge | Bridge mechanics, validator control |
| Validator Docs | https://hyperliquid.gitbook.io/hyperliquid-docs/validators/running-a-validator | Validator requirements |
| Delegation Program | https://hyperliquid.gitbook.io/hyperliquid-docs/validators/delegation-program | Foundation delegation criteria |
| HIP-1 Documentation | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-1-native-token-standard | Native token standard |
| HIP-3 Documentation | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-3-builder-deployed-perpetuals | Builder-deployed perps |
| HIPs Overview | https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips | All HIPs |
| Wrapped HYPE | https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype | WHYPE on HyperEVM |
| Info API Endpoint | https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/info-endpoint | Staking/delegation queries |

### GitHub Repositories (Confirmed)

| Repository | URL | License | Content |
|------------|-----|---------|---------|
| node | https://github.com/hyperliquid-dex/node | Apache 2.0 | Validator node software |
| contracts | https://github.com/hyperliquid-dex/contracts | - | Bridge2.sol, Signature.sol |
| hyperliquid-python-sdk | https://github.com/hyperliquid-dex/hyperliquid-python-sdk | MIT | Python trading SDK |
| hyperliquid-rust-sdk | https://github.com/hyperliquid-dex/hyperliquid-rust-sdk | MIT | Rust trading SDK |

### Block Explorers (Confirmed)

| Explorer | URL | Network |
|----------|-----|---------|
| HyperScan (Blockscout) | https://www.hyperscan.com/ | HyperEVM |
| HyperScan Gas Tracker | https://www.hyperscan.com/gas-tracker | HyperEVM gas fees |
| HypurrScan Staking | https://hypurrscan.io/staking | Validator/staking data |
| HyperZap | https://hyperzap.io/explorer | HyperEVM |
| Hyperliquid Explorer | https://app.hyperliquid.xyz/explorer | HyperCore |
| Validator Performance | https://app.hyperliquid.xyz/staking/validatorPerformance | Validator metrics |
| Staking UI | https://app.hyperliquid.xyz/staking | Stake delegation |

### Bridge Contracts (Confirmed)

| Contract | Address | Network |
|----------|---------|---------|
| Bridge2 | 0x2df1c51e09aecf9cacb7bc98cb1742757f163df7 | Arbitrum |

### HyperEVM System Contracts (Confirmed)

| Contract | Address | Network | Notes |
|----------|---------|---------|-------|
| Wrapped HYPE (WHYPE) | 0x5555555555555555555555555555555555555555 | HyperEVM | Immutable, mirrors WETH |

### Legal/Corporate Entities (Confirmed)

| Entity | Type | Location | Source |
|--------|------|----------|--------|
| Hyperliquid Labs Pte. Ltd. | Private Company | Singapore (202402326K) | https://www.sgpbusiness.com/company/Hyperliquid-Labs-Pte-Ltd |
| Hyper Foundation | Foundation | Unknown | https://hyperfoundation.org/ |

### Trademark (Confirmed)

| Mark | Applicant | Status | Source |
|------|-----------|--------|--------|
| HYPERLIQUID | Hyper Foundation | Filed (USPTO 99599981) | USPTO |

### Governance Sources (Confirmed)

| Resource | URL | Content |
|----------|-----|---------|
| Discord Server | https://discord.gg/hyperliquid | Governance forum channel |
| Hyper Foundation X | https://x.com/HyperFND | Governance announcements |

**Note:** Governance discussions occur in Discord's governance forum channel. Validators must post positions publicly. No dedicated governance forum website exists; Discord is the primary venue.

### Data Sources (Confirmed)

| Source | URL | Data |
|--------|-----|------|
| Tokenomist | https://tokenomist.ai/hyperliquid | Vesting schedules, unlock events |
| DefiLlama | https://defillama.com/unlocks/hyperliquid | Token unlocks |
| DefiLlama Bridge | https://defillama.com/protocol/hyperliquid-bridge | Bridge TVL |
| CoinGecko | https://www.coingecko.com/en/coins/hyperliquid | Price, supply data |

### Sources That Don't Exist

| Expected Resource | Status | Notes |
|------------------|--------|-------|
| Dedicated Governance Forum | Does not exist | Uses Discord #governance channel |
| Foundation Budget Documentation | Does not exist | No public transparency reports |
| Foundation Legal Structure | Not disclosed | hyperfoundation.org has no content |

---

## HyperEVM Investigation Scope

### HIP-1 HYPE Wrapper (WHYPE)

**Contract Address:** `0x5555555555555555555555555555555555555555`

**Verification Method:**
1. Query HyperScan: https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555
2. Verify source code matches WETH pattern
3. Confirm immutability (no proxy, no admin functions)

**Key Properties (per documentation):**
- Immutable contract (no upgrade mechanism)
- Same source as WETH on Ethereum mainnet
- 18 decimals, "Wrapped HYPE" name, "WHYPE" symbol
- Deposit/withdrawal for native HYPE ↔ WHYPE

### HyperEVM Gas Fees

**Fee Flow Investigation:**
1. Query HyperScan Gas Tracker: https://www.hyperscan.com/gas-tracker
2. Document base fee mechanism (EIP-1559 enabled)
3. Verify burn mechanics

**Known Properties (per documentation):**
- EIP-1559 enabled on HyperEVM
- Base fees are burned (standard EIP-1559)
- Priority fees are also burned (differs from Ethereum—no proposer tips)
- Small blocks: static 1 gwei base fee
- Big blocks: 0.5 gwei per unit, 20% rebated to stakers

**Observable Test:**
- Compare gas tracker data to staking rewards
- Verify burned amounts visible on explorer

### HyperEVM vs HyperCore Governance

**Investigation:**
1. Determine if HyperEVM has separate governance parameters
2. Check if HIP proposals can modify HyperEVM behavior
3. Document any HyperEVM-specific admin roles

**Expected Finding:**
- HyperEVM shares HyperBFT consensus with HyperCore
- No separate governance; same validator set controls both
- L1 parameters apply to both components

---

## Validator Enumeration Methodology

### API-Based Enumeration

**Endpoint:** `POST https://api.hyperliquid.xyz/info`

**Query Validator List:**
```json
{
  "type": "validatorSummaries"
}
```

**Query User Delegations:**
```json
{
  "type": "delegations",
  "user": "0x..."
}
```

**Query Delegator Summary:**
```json
{
  "type": "delegatorSummary",
  "user": "0x..."
}
```

### Stake Concentration Analysis

**Method:**
1. Query `validatorSummaries` to get all validators and their total stake
2. Calculate each validator's stake as % of total staked HYPE
3. Identify Foundation-delegated validators (per delegation program)
4. Cross-reference with HypurrScan staking dashboard: https://hypurrscan.io/staking

**Concentration Thresholds:**

| Stake % | Status | Rationale |
|---------|--------|-----------|
| Any single validator >33% | Warning | Can block finality |
| Any coordinated group >50% | Warning | Controls majority |
| Foundation-delegated validators >67% | Warning | Foundation controls quorum |
| Top 5 validators >80% | Concern | High concentration |

**Known Data Point:** Per third-party analysis (January 2025), ~81% of staked shares were controlled by Foundation nodes. This must be re-verified with current data.

### Validator Identity Attribution

**Method:**
1. Check if validator addresses are self-identified (validator name on-chain)
2. Cross-reference with delegation program approved validators
3. Identify "Foundation Node" validators (5 validators run by team per documentation)
4. Document any validators with unknown operators

---

## Observable Behavior Tests for L1 Claims

### 1.4 Token Upgrade Authority

**Claim:** HYPE token behavior is defined in L1 code (closed source)

**Observable Tests:**
| Test | Method | What Would Confirm | What Would Deny |
|------|--------|-------------------|-----------------|
| Token behavior consistency | Compare HYPE transfers/balances over time | No unexpected behavior changes | Transfer rules or balance calculations change without announcement |
| WHYPE parity | Verify 1:1 HYPE↔WHYPE conversion | Always 1:1 | Conversion rate changes or failures |
| Historical behavior | Query old transactions | Consistent gas/transfer semantics | Different behavior in old vs. new txs |

**Falsifiable Test:** Monitor WHYPE contract events. If HYPE can be modified, WHYPE wrapping behavior would reflect changes.

### 1.7 Token Censorship

**Claim:** No blacklist/freeze functions exist

**Observable Tests:**
| Test | Method | What Would Confirm | What Would Deny |
|------|--------|-------------------|-----------------|
| Bridge2 blacklist | Audit Bridge2.sol code | No blacklist functions in code | Blacklist/freeze functions present |
| L1 transaction inclusion | Monitor mempool→block inclusion | All valid txs included proportionally | Some addresses consistently excluded |
| Historical censorship | Search for failed txs from sanctioned addresses | No pattern of address-based exclusion | Sanctioned addresses fail differently |

**Falsifiable Test:** Submit small transactions from multiple addresses including recently-flagged OFAC addresses. If L1 has censorship, these would fail or be delayed.

**Practical Limitation:** This test may not be feasible to execute but historical data can be analyzed.

### 3.1 Token Source Verification

**Claim:** WHYPE is verifiable; native HYPE is not

**Observable Tests:**
| Test | Method | What Would Confirm | What Would Deny |
|------|--------|-------------------|-----------------|
| WHYPE source verification | HyperScan contract verification | Source matches claimed WETH pattern | Source differs or unverified |
| WHYPE immutability | Check for proxy/admin patterns | No upgrade functions | Proxy or admin detected |
| WHYPE behavior | Test deposit/withdraw | Works as documented | Unexpected behavior |

---

## Evidence Sufficiency Thresholds

### Metric 1: Onchain Control

| Criterion | Positive | Neutral | Warning | Unknown |
|-----------|----------|---------|---------|---------|
| **1.1 Governance Workflow** | Onchain voting with tokenholder authority; executed via contract | Validator voting exists; delegators influence stake | Offchain voting only; validators controlled by team | No evidence of governance mechanism |
| **1.2 Role Accountability** | All roles elected/revocable by tokenholder vote | Roles documented; some tokenholder oversight | Roles controlled by Foundation/team discretion | Cannot identify privileged roles |
| **1.3 Protocol Upgrade** | Upgrades require tokenholder approval; timelock | Validator quorum required; public process | Team/Foundation can upgrade unilaterally | Upgrade mechanism unknown |
| **1.4 Token Upgrade** | Token immutable or tokenholder-controlled | Token non-upgradeable (verified code) | Token upgradeable by team | Cannot verify upgradeability |
| **1.5 Supply Control** | Fixed supply; no mint without governance | Fixed supply; programmatic emissions | Discretionary minting possible | Supply control unknown |
| **1.6 Access Gating** | No privileged gating; permissionless | Time-limited gating (unstaking queue) | Arbitrary pause/lock by bounded set | Cannot determine gating powers |
| **1.7 Censorship** | No freeze/blacklist in verified code | No evidence of historical censorship | Freeze/blacklist functions exist | Cannot verify censorship capability |

### Metric 2: Value Accrual

| Criterion | Positive | Neutral | Warning | Unknown |
|-----------|----------|---------|---------|---------|
| **2.1 Accrual Active** | Onchain evidence of recent buyback/burn + documented fee flow | Documentation only; no recent onchain evidence | Mechanism inactive or discretionary | Cannot verify mechanism exists |
| **2.2 Treasury Ownership** | Treasury controlled by tokenholder governance | Automated treasury (Assistance Fund) | Discretionary treasury (Foundation budget) | Treasury control unknown |
| **2.3 Mechanism Control** | Only tokenholders can modify fee parameters | Parameters require validator quorum | Team can modify fee parameters | Fee parameter control unknown |
| **2.4 Offchain Accrual** | Offchain value flows to tokenholders via legal structure | No offchain accrual claimed | Offchain revenue retained by Labs | Cannot verify offchain flows |

### Metric 3: Verifiability

| Criterion | Positive | Neutral | Warning | Unknown |
|-----------|----------|---------|---------|---------|
| **3.1 Token Source** | Verified source matching deployed bytecode | Native token; wrapper verified | Partial verification; some closed | Cannot verify token code |
| **3.2 Protocol Source** | All core contracts verified and audited | Key contracts verified; L1 closed | Critical contracts unverified | No source verification possible |

### Metric 4: Distribution

| Criterion | Positive | Neutral | Warning | Unknown |
|-----------|----------|---------|---------|---------|
| **4.1 Concentration** | No single actor >20%; no coordinated >40% | Some concentration; vesting mitigates | Foundation/team controls >50% validators | Cannot determine stake distribution |
| **4.2 Unlocks** | Gradual unlocks; <10%/year impact | Moderate unlocks; 10-20%/year | Large cliffs; >20% single event | Unlock schedule unknown |

### Offchain Dependencies

| Criterion | Positive | Neutral | Warning | Unknown |
|-----------|----------|---------|---------|---------|
| **5.1 Trademark** | Tokenholder-controlled entity owns mark | Foundation owns mark; governance link unclear | Private company owns mark | Trademark ownership unknown |
| **5.2 Distribution** | Primary interface tokenholder-controlled | Multiple interfaces; some independent | Labs controls primary interface | Interface control unknown |
| **5.3 Licensing** | Permissive open source; tokenholder IP | Mixed licensing; L1 closed | Proprietary L1; Labs owns IP | IP ownership unknown |

---

## Framework Criteria Analysis

### Metric 1: Onchain Control

#### 1.1 Onchain Governance Workflow

**Question:** Does an onchain process exist that grants HYPE holders ultimate authority over protocol decisions?

**Investigation Approach:**
1. Document the governance mechanism: validator stake-weighted voting via HyperBFT consensus
2. Verify if HYPE stakers/delegators have governance voting rights
3. Analyze the relationship between delegation and governance power
4. Document recent governance votes (Assistance Fund burn vote Dec 2025, USDH ticker vote Sep 2025)
5. Identify formal proposal process (HIPs)

**Sources:**
- Staking documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking
- Discord #governance channel: https://discord.gg/hyperliquid
- Hyper Foundation X: https://x.com/HyperFND
- HIPs overview: https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips

**Evidence Required:**
- Documentation of governance voting mechanism
- Evidence that HYPE stake weight determines voting power
- Examples of executed governance decisions
- Any timelock or execution delay mechanisms

**Observable Test:** Query recent validator votes (USDH, Assistance Fund) and verify stake-weighted outcome matches announcement.

**Anticipated Status:** Warning
- Governance exists through validator stake-weighted voting
- Validators concentrated under Foundation control (~81% reported)
- No onchain governance contracts (Governor/Timelock)
- Tokenholders influence via delegation, not direct voting

---

#### 1.2 Role Accountability

**Question:** Are all privileged roles governed, revocable, and accountable to HYPE holders?

**Investigation Approach:**
1. Map all privileged roles in the system
2. Identify who controls validator selection (Foundation delegation vs. organic stake)
3. Document jailing mechanism and parameters
4. Analyze bridge contract roles (lockers, finalizers, validators)
5. Determine if HYPE holders can remove or replace privileged roles

**Sources:**
- Bridge2.sol contract: https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol
- Validator documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/validators/running-a-validator
- Delegation program: https://hyperliquid.gitbook.io/hyperliquid-docs/validators/delegation-program

**Evidence Required:**
- List of all privileged roles with onchain verification
- Evidence that roles are elected/revocable by tokenholder governance
- Documentation of jailing mechanism
- Bridge contract role analysis

**Observable Test:** Audit Bridge2.sol for all role definitions; cross-reference with onchain state.

**Anticipated Status:** Warning
- Validators can jail peers via 2/3 quorum voting
- Foundation has discretionary delegation power ("reserves the right to cease delegation at any time")
- Bridge roles (lockers, finalizers) controlled by validators, not tokenholders directly
- Cold wallet signers for emergency unlock not publicly identified

---

#### 1.3 Protocol Upgrade Authority

**Question:** Can core protocol logic be upgraded, and if so, is it controlled by HYPE holders?

**Investigation Approach:**
1. Analyze if L1 code can be upgraded and by whom
2. Review Bridge2 contract upgradeability
3. Check HyperEVM system contract upgradeability
4. Document any upgrade history or mechanisms

**Sources:**
- Bridge2 contract on Arbiscan: https://arbiscan.io/address/0x2df1c51e09aecf9cacb7bc98cb1742757f163df7
- Node repository: https://github.com/hyperliquid-dex/node
- WHYPE contract: https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555

**Evidence Required:**
- Bridge contract upgradeability status (appears non-upgradeable)
- WHYPE upgradeability status (documented as immutable)
- L1 upgrade mechanism documentation
- Evidence of who controls L1 upgrades

**Observable Test:**
- Verify Bridge2 has no proxy pattern or upgrade functions
- Verify WHYPE has no proxy pattern
- Check node repository for upgrade-related code paths

**Anticipated Status:** Neutral
- Bridge2 appears non-upgradeable (no proxy pattern)
- WHYPE documented as immutable
- L1 upgrades require validator coordination (closed source, cannot verify)

---

#### 1.4 Token Upgrade Authority

**Question:** Can HYPE token behavior be modified, and if so, is it controlled by tokenholder governance?

**Investigation Approach:**
1. Confirm HYPE is native L1 token (not EVM contract)
2. Verify WHYPE wrapper on HyperEVM
3. Analyze if token behavior can be modified via L1 upgrade

**Sources:**
- HIP-1 documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-1-native-token-standard
- WHYPE documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype
- WHYPE on HyperScan: https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555

**Evidence Required:**
- Confirmation HYPE is native token
- WHYPE verification status
- Any evidence of token behavior modification capability

**Observable Tests:**
- Verify WHYPE source matches documented WETH pattern
- Monitor for any token behavior changes over time
- Test WHYPE deposit/withdraw functionality

**Anticipated Status:** Neutral
- HYPE is native L1 token (like ETH on Ethereum)
- WHYPE is immutable ERC-20 wrapper (verified)
- L1 token logic cannot be independently verified (closed source)

---

#### 1.5 Supply Control

**Question:** Are HYPE supply changes programmatic or subject to tokenholder governance?

**Investigation Approach:**
1. Document total supply (1 billion fixed)
2. Verify no inflation/mint mechanism
3. Analyze burn mechanisms (Assistance Fund)
4. Document emission schedule for future rewards (38.89% allocation)

**Sources:**
- Tokenomics data: https://tokenomist.ai/hyperliquid
- Assistance Fund documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees
- CoinGecko supply: https://www.coingecko.com/en/coins/hyperliquid

**Evidence Required:**
- Total supply verification (1 billion)
- Evidence of fixed supply (no mint function claims)
- Burn mechanism documentation
- Future emission schedule
- Onchain burn transaction evidence

**Observable Tests:**
- Query total supply via API or explorer
- Track Assistance Fund burn transactions
- Compare circulating supply changes to unlock schedule

**Anticipated Status:** Positive
- Fixed 1 billion supply (documented)
- Burns via Assistance Fund (automated)
- Future emissions from allocated reserves, not inflation
- Cannot verify no mint function in L1 code

---

#### 1.6 Privileged Access Gating

**Question:** Can any bounded actor set block or restrict economically meaningful actions or exit paths?

**Investigation Approach:**
1. Analyze bridge dispute mechanism and locking capability
2. Document validator jailing mechanism
3. Identify pause/halt functions
4. Check withdrawal restrictions

**Sources:**
- Bridge2.sol: https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol
- Bridge documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/bridge
- Staking documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking

**Evidence Required:**
- Bridge locker mechanism analysis
- Validator jailing parameters
- Any pause/halt capabilities
- Withdrawal restrictions (7-day unstaking queue, 1-day delegation lockup)

**Observable Tests:**
- Audit Bridge2.sol for locker/pause functions
- Verify unstaking queue timing
- Check for historical pause events

**Anticipated Status:** Warning
- Bridge can be locked by "locker" addresses
- 7-day unstaking queue exists (documented)
- 1-day delegation lockup (documented)
- Validators can be jailed by peer 2/3 vote
- Cold wallet signers for emergency unlock not public

---

#### 1.7 Token Censorship

**Question:** Can any roles freeze, blacklist, seize, or censor HYPE balances or transfers?

**Investigation Approach:**
1. Audit Bridge2 for blacklist/freeze functions
2. Check WHYPE for admin functions
3. Analyze L1 censorship capabilities (limited by closed source)

**Sources:**
- Bridge2.sol: https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol
- WHYPE on HyperScan: https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555

**Evidence Required:**
- Absence of blacklist/freeze in Bridge2 (code audit)
- Absence of admin functions in WHYPE (code audit)
- Any historical evidence of censorship

**Observable Tests:**
- Full audit of Bridge2.sol and WHYPE source
- Search for failed transactions from sanctioned addresses
- Monitor transaction inclusion patterns

**Anticipated Status:** Neutral
- Bridge2 does not appear to have blacklist functions
- WHYPE documented as simple WETH clone (no admin)
- L1 capabilities cannot be verified
- Validator set could theoretically censor (requires 2/3 consensus)

---

### Metric 2: Value Accrual

#### 2.1 Accrual Active

**Question:** Are value flows to HYPE holders currently active?

**Investigation Approach:**
1. Document Assistance Fund buyback mechanism
2. Quantify fee flows (>$1B annualized claim)
3. Verify staking rewards distribution
4. Document fee discount mechanism for stakers

**Sources:**
- Fee documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees
- Staking documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking
- Assistance Fund address: `0xfefefefefefefefefefefefefefefefefefefefe`

**Evidence Required:**
- Active buyback evidence (onchain transactions)
- Staking reward distribution data
- Fee revenue data
- Burn transaction history

**Observable Tests:**
- Query Assistance Fund transactions on HyperCore explorer
- Query staking rewards via API (`delegatorRewards`)
- Compare documented fee % to actual flows

**Evidence Sufficiency:**
- Positive: Recent buyback/burn transactions + documented 97% fee flow
- Neutral: Documentation only; no recent onchain evidence
- Warning: Mechanism inactive or discretionary

**Anticipated Status:** Positive
- 97% of fees → Assistance Fund → HYPE buyback → burn (documented)
- Staking rewards active (~2.37% APY at 400M staked)
- Fee discounts for stakers (5-40%)
- Need to verify with onchain data

---

#### 2.2 Treasury Ownership

**Question:** Are protocol treasury assets programmatically controlled by tokenholder governance?

**Investigation Approach:**
1. Identify treasury/Assistance Fund control mechanism
2. Determine if HYPE holders can direct treasury usage
3. Document Hyper Foundation budget (6% allocation)

**Sources:**
- Assistance Fund documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees
- Foundation budget: **No public documentation exists**

**Evidence Required:**
- Treasury control mechanism
- Evidence of tokenholder authority over treasury
- Foundation spending discretion

**Evidence Sufficiency:**
- Positive: Treasury controlled by tokenholder governance
- Neutral: Automated treasury (Assistance Fund burns automatically)
- Warning: Discretionary treasury (Foundation budget)

**Anticipated Status:** Neutral
- Assistance Fund is automated (burns HYPE, not discretionary)
- Foundation budget (6%) has no public governance documentation
- HLP vault and deployer fees are separate flows (3% + deployer share)

---

#### 2.3 Accrual Mechanism Control

**Question:** Can only tokenholders modify parameters governing value capture?

**Investigation Approach:**
1. Document who can modify fee parameters
2. Analyze Assistance Fund parameter changes
3. Verify if fee switch exists

**Sources:**
- Fee documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees
- HIPs for fee changes

**Evidence Required:**
- Fee parameter modification authority
- Historical fee changes
- Any governance votes on fee parameters

**Observable Tests:**
- Search for fee parameter change announcements
- Check for HIPs related to fee changes

**Evidence Sufficiency:**
- Positive: Only tokenholder governance can modify fees
- Neutral: Validator quorum required for fee changes
- Warning: Team can modify fee parameters

**Anticipated Status:** Warning
- Fee parameters controlled at L1 level (closed source)
- Unclear if tokenholder governance can modify fees
- Deployer fees (up to 50%) are a separate flow

---

#### 2.4 Offchain Value Accrual

**Question:** Are there additional offchain value accrual flows benefiting HYPE holders?

**Investigation Approach:**
1. Document any offchain revenue or IP licensing
2. Analyze Hyperliquid Labs business model

**Sources:**
- Corporate filings: Singapore ACRA
- Terms of service: https://app.hyperliquid.xyz/terms

**Evidence Required:**
- Evidence of offchain value flows (or lack thereof)

**Anticipated Status:** Unevaluated
- No evidence of offchain revenue sharing
- Hyperliquid Labs is separate from token governance

---

### Metric 3: Verifiability

#### 3.1 Token Contract Source Verification

**Question:** Is HYPE token source code publicly available and verifiable?

**Investigation Approach:**
1. Confirm HYPE is native L1 token (no contract to verify)
2. Verify WHYPE wrapper on HyperEVM
3. Document HIP-1 specification

**Sources:**
- HIP-1 documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/hyperliquid-improvement-proposals-hips/hip-1-native-token-standard
- WHYPE on HyperScan: https://www.hyperscan.com/address/0x5555555555555555555555555555555555555555
- WHYPE documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/wrapped-hype

**Evidence Required:**
- WHYPE contract verification status on HyperScan
- Source code comparison to WETH
- Confirmation of immutability

**Observable Tests:**
- Check HyperScan for verified source code badge
- Compare deployed bytecode to documented WETH source
- Verify no proxy/upgrade patterns

**Evidence Sufficiency:**
- Positive: WHYPE verified, matches WETH pattern
- Neutral: WHYPE verified; native HYPE unverifiable
- Warning: WHYPE unverified or differs from docs

**Anticipated Status:** Neutral
- HYPE is native L1 token (cannot verify like ERC-20)
- WHYPE documented as verified WETH clone
- L1 token logic is not publicly verifiable

---

#### 3.2 Protocol Component Source Verification

**Question:** Are core protocol contracts publicly accessible and verifiable?

**Investigation Approach:**
1. Verify Bridge2 contract on Arbitrum
2. Check HyperEVM system contracts
3. Document open source vs. closed source components

**Sources:**
- Bridge2 on Arbiscan: https://arbiscan.io/address/0x2df1c51e09aecf9cacb7bc98cb1742757f163df7
- Bridge2 source: https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol
- GitHub repos: https://github.com/hyperliquid-dex

**Evidence Required:**
- Bridge2 verification status on Arbiscan
- List of verified vs. unverified components
- L1 code availability assessment

**Observable Tests:**
- Check Arbiscan verification badge
- Compare deployed bytecode to GitHub source
- Enumerate all HyperEVM system contracts

**Evidence Sufficiency:**
- Positive: All critical contracts verified
- Neutral: Key bridge verified; L1 closed
- Warning: Critical contracts unverified

**Anticipated Status:** Neutral
- Bridge2 source available in GitHub
- SDKs are open source (MIT)
- Node software has Apache 2.0 license
- Core L1 execution/consensus code is closed source

---

### Metric 4: Token Distribution

#### 4.1 Ownership Concentration

**Question:** Does a single actor or coordinated group control a majority of voting supply?

**Investigation Approach:**
1. Query validator stake distribution via API
2. Calculate Foundation delegation share
3. Document team/core contributor holdings
4. Identify Foundation holdings

**Sources:**
- Tokenomist: https://tokenomist.ai/hyperliquid
- HypurrScan staking: https://hypurrscan.io/staking
- API: `POST https://api.hyperliquid.xyz/info` with `{"type": "validatorSummaries"}`
- Validator Performance: https://app.hyperliquid.xyz/staking/validatorPerformance

**Evidence Required:**
- Validator stake distribution
- Foundation delegation percentage
- Team vesting status (23.8% total, currently ~0.56% unlocked)
- Foundation budget status (6%)

**Observable Tests:**
- Query validatorSummaries API
- Calculate top 5 validator stake concentration
- Identify Foundation Node validators

**Concentration Thresholds:**
- Warning: Foundation-delegated >67%, any single >33%, top 5 >80%
- Concern: Team + Foundation unlocked stake >50% of active

**Known Data:** ~81% Foundation-controlled stake reported (Jan 2025); needs verification.

**Anticipated Status:** Warning
- Genesis (31%) was widely distributed
- Team (23.8%) is significant but mostly locked
- Foundation (6%) + Future emissions (38.89%) are large controlled pools
- Foundation delegation dominates validator stake

---

#### 4.2 Future Token Unlocks

**Question:** Are there known future events that will materially affect token concentration?

**Investigation Approach:**
1. Document full unlock schedule
2. Identify key cliff dates
3. Analyze impact on circulating supply

**Sources:**
- Tokenomist: https://tokenomist.ai/hyperliquid
- DefiLlama unlocks: https://defillama.com/unlocks/hyperliquid

**Evidence Required:**
- Full unlock schedule with dates
- Monthly unlock amounts
- Impact analysis on circulating supply

**Observable Tests:**
- Cross-reference Tokenomist and DefiLlama data
- Calculate % supply unlocking per event

**Evidence Sufficiency:**
- Positive: Gradual unlocks; <10%/year
- Neutral: Moderate unlocks; 10-20%/year
- Warning: Large cliffs; >20% single event

**Anticipated Status:** Warning
- Core contributor vesting: 24 months starting Nov 2025
- Next unlock: May 6, 2026 (~9.9M HYPE, ~2.3% of released)
- Monthly distributions on 6th of each month
- 76% of supply currently locked

---

### Offchain Dependencies

#### 5.1 Trademark

**Question:** Are core trademarks owned by a tokenholder-controlled entity?

**Investigation Approach:**
1. Verify USPTO filing for HYPERLIQUID
2. Identify trademark owner
3. Analyze relationship to token governance

**Sources:**
- USPTO application 99599981
- Hyper Foundation website: https://hyperfoundation.org/ (minimal content)

**Evidence Required:**
- Trademark filing details
- Owner identification (Hyper Foundation)
- Governance relationship (none documented)

**Evidence Sufficiency:**
- Positive: Tokenholder-controlled entity owns mark
- Neutral: Foundation owns mark; governance unclear
- Warning: Private company owns mark

**Anticipated Status:** Warning
- HYPERLIQUID trademark filed by Hyper Foundation
- Foundation governance not documented publicly
- No clear mechanism for tokenholders to control trademark

---

#### 5.2 Distribution

**Question:** Are primary domains and interfaces controlled by a tokenholder-controlled entity?

**Investigation Approach:**
1. Identify domain ownership (hyperliquid.xyz, app.hyperliquid.xyz)
2. Review terms of service
3. Analyze who operates interface

**Sources:**
- Terms of service: https://app.hyperliquid.xyz/terms
- WHOIS records for domains

**Evidence Required:**
- Domain ownership (likely Labs)
- Interface operator identity
- Relationship to governance

**Evidence Sufficiency:**
- Positive: Primary interface tokenholder-controlled
- Neutral: Multiple interfaces; some independent
- Warning: Labs controls primary interface

**Anticipated Status:** Warning
- Likely operated by Hyperliquid Labs Pte. Ltd. (Singapore)
- Labs is separate from tokenholder governance
- No alternative decentralized interfaces documented

---

#### 5.3 Licensing

**Question:** Is core protocol software owned by a tokenholder-controlled entity?

**Investigation Approach:**
1. Document open source licenses for public code
2. Analyze L1 code ownership
3. Review any IP assignments

**Sources:**
- GitHub repositories: https://github.com/hyperliquid-dex
- Corporate filings (Singapore ACRA)

**Evidence Required:**
- License terms for all components
- Code ownership identification
- IP assignment documentation

**Evidence Sufficiency:**
- Positive: Permissive open source; tokenholder IP
- Neutral: Mixed licensing; L1 closed
- Warning: Proprietary L1; Labs owns IP

**Anticipated Status:** Warning
- SDKs: MIT (permissive)
- Node: Apache 2.0 (permissive)
- Core L1: Closed source (owned by Labs presumably)
- No public IP assignment to tokenholder entity

---

## Special Considerations for Hyperliquid L1

### L1 Architecture Implications

Unlike EVM-based protocols, Hyperliquid is an L1 blockchain. This creates unique challenges for the framework:

1. **Native Token vs. Smart Contract:** HYPE is a native token, not an ERC-20. Traditional contract verification doesn't apply. WHYPE wrapper provides verifiable EVM representation.

2. **Consensus-Level Governance:** Governance operates through validator voting, not smart contract execution. Delegation provides indirect tokenholder influence.

3. **Closed Source Core:** The HyperBFT consensus and HyperCore execution code are not publicly available. Observable behavior tests are required.

4. **Validator Concentration:** The 24 validator limit and Foundation delegation program create concentration risk.

### Evidence Verification Matrix

| Area | Fully Verifiable | Method |
|------|------------------|--------|
| Bridge mechanics | Yes | Bridge2.sol audit on Arbitrum |
| WHYPE token | Yes | HyperScan source verification |
| Fee flows | Partial | Onchain transactions + documentation |
| Staking rewards | Partial | API queries + documentation |
| Governance voting | Limited | Public announcements, Discord forum |
| L1 upgrades | No | Closed source; observe behavior |
| Native token logic | No | Closed source; observe behavior |
| Validator control | Partial | API stake queries + attribution |

---

## Execution Checklist

### Phase 1: Onchain Data Collection
- [ ] Query HyperScan for WHYPE verification status
- [ ] Audit Bridge2 contract fully (all functions, roles, events)
- [ ] Query `validatorSummaries` API to enumerate validators
- [ ] Calculate validator stake concentration
- [ ] Track Assistance Fund address activity
- [ ] Query `delegatorRewards` for staking reward data

### Phase 2: Documentation Verification
- [ ] Cross-reference docs against observed behavior
- [ ] Document USDH and Assistance Fund governance votes
- [ ] Verify fee flow claims (97% to Assistance Fund)
- [ ] Confirm unlock schedule against Tokenomist

### Phase 3: Legal/Offchain Research
- [ ] Check USPTO for trademark 99599981 details
- [ ] Query Singapore ACRA for Labs corporate details
- [ ] Analyze terms of service for interface operator
- [ ] Document Foundation structure (likely minimal info)

### Phase 4: Gap Documentation
- [ ] List all claims that cannot be verified (L1 closed source)
- [ ] Identify closed-source dependencies
- [ ] Document validator concentration findings
- [ ] Note L1-specific limitations

---

## Expected Outputs

1. **Research Report** (`hype-research.md`): Comprehensive findings with evidence links
2. **tokens.json entry**: Token metadata following schema
3. **metrics.json entry**: Per-criteria evidence and status

## Anticipated Overall Assessment

Based on preliminary research:

- **Onchain Control:** Warning - validator governance exists but concentrated; L1 closed source
- **Value Accrual:** Positive - active buybacks, burns, staking rewards (need onchain verification)
- **Verifiability:** Neutral - Bridge and WHYPE verifiable; L1 not
- **Distribution:** Warning - significant unlocks pending; Foundation controls validator stake
- **Offchain:** Warning - Labs/Foundation control IP, trademarks, interfaces

The HYPE token demonstrates active value accrual mechanisms but faces challenges around verifiability (closed source L1) and concentration (Foundation validator control, large locked allocations).
