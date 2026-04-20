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
| HyperEVMScan | https://hyperevmscan.io/ | HyperEVM |
| HyperZap | https://hyperzap.io/explorer | HyperEVM |
| Hyperliquid Explorer | https://app.hyperliquid.xyz/explorer | HyperCore |

### Bridge Contracts (Confirmed)

| Contract | Address | Network |
|----------|---------|---------|
| Bridge2 | 0x2df1c51e09aecf9cacb7bc98cb1742757f163df7 | Arbitrum |

### Legal/Corporate Entities (Confirmed)

| Entity | Type | Location |
|--------|------|----------|
| Hyperliquid Labs Pte. Ltd. | Private Company | Singapore (202402326K) |
| Hyper Foundation | Foundation | Unknown |

### Trademark (Confirmed)

| Mark | Applicant | Status |
|------|-----------|--------|
| HYPERLIQUID | Hyper Foundation | Filed (USPTO 99599981) |

### Data Sources (Confirmed)

| Source | URL | Data |
|--------|-----|------|
| Tokenomist | https://tokenomist.ai/hyperliquid | Vesting schedules, unlock events |
| DefiLlama | https://defillama.com/unlocks/hyperliquid | Token unlocks |
| DefiLlama Bridge | https://defillama.com/protocol/hyperliquid-bridge | Bridge TVL |
| CoinGecko | https://www.coingecko.com/en/coins/hyperliquid | Price, supply data |

---

## Framework Criteria Analysis

### Metric 1: Onchain Control

#### 1.1 Onchain Governance Workflow

**Question:** Does an onchain process exist that grants HYPE holders ultimate authority over protocol decisions?

**Investigation Approach:**
1. Document the governance mechanism: validator stake-weighted voting via HyperBFT consensus
2. Verify if HYPE stakers/delegators have governance voting rights
3. Analyze the relationship between delegation and governance power
4. Document any recent governance votes (e.g., Assistance Fund burn vote, USDH ticker vote)
5. Identify if there's a formal proposal process (HIPs)

**Sources:**
- Staking documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/staking
- Governance forum/announcements
- HIP-3 documentation
- Recent governance vote records

**Evidence Required:**
- Documentation of governance voting mechanism
- Evidence that HYPE stake weight determines voting power
- Examples of executed governance decisions
- Any timelock or execution delay mechanisms

**Anticipated Status:** Neutral/Warning
- Governance exists through validator stake-weighted voting
- However, validators may be concentrated under team control
- No evidence of formal tokenholder governance contracts like Governor/Timelock

**Gaps/Concerns:**
- Hyperliquid is not fully open-source; core L1 code not publicly auditable
- Team operates majority of validators (needs verification)
- Difference between "validator governance" and "tokenholder governance"

---

#### 1.2 Role Accountability

**Question:** Are all privileged roles governed, revocable, and accountable to HYPE holders?

**Investigation Approach:**
1. Map all privileged roles in the system
2. Identify who controls validator selection (Foundation delegation vs. organic stake)
3. Document the jailing mechanism and its parameters
4. Analyze bridge contract roles (lockers, finalizers, validators)
5. Determine if HYPE holders can remove or replace privileged roles

**Sources:**
- Bridge2.sol contract: https://github.com/hyperliquid-dex/contracts/blob/master/Bridge2.sol
- Validator documentation
- Delegation program terms

**Evidence Required:**
- List of all privileged roles with onchain verification
- Evidence that roles are elected/revocable by tokenholder governance
- Documentation of jailing mechanism
- Bridge contract role analysis

**Anticipated Status:** Warning
- Validators can jail peers via quorum voting
- Foundation has discretionary delegation power
- Bridge roles (lockers, finalizers) appear controlled by validators, not tokenholders directly

**Gaps/Concerns:**
- Core L1 code is not open-source; cannot verify all privileged functions
- Foundation delegation is discretionary ("reserves the right to cease delegation at any time")
- Unclear if HYPE holders can replace bridge validators

---

#### 1.3 Protocol Upgrade Authority

**Question:** Can core protocol logic be upgraded, and if so, is it controlled by HYPE holders?

**Investigation Approach:**
1. Analyze if the L1 code can be upgraded and by whom
2. Review Bridge2 contract upgradeability
3. Check HyperEVM contract upgradeability patterns
4. Document any upgrade history or mechanisms

**Sources:**
- Bridge2 contract on Arbiscan: https://arbiscan.io/address/0x2df1c51e09aecf9cacb7bc98cb1742757f163df7
- Node repository: https://github.com/hyperliquid-dex/node
- HyperEVM explorer for any proxy contracts

**Evidence Required:**
- Bridge contract upgradeability status (appears non-upgradeable)
- L1 upgrade mechanism documentation
- Evidence of who controls L1 upgrades

**Anticipated Status:** Warning/Unknown
- Bridge2 appears non-upgradeable (no proxy pattern)
- L1 upgrades likely require validator coordination
- Core L1 code is closed-source

**Gaps/Concerns:**
- Cannot verify L1 upgrade mechanism without source code
- Validator set concentration may mean team controls upgrades de facto

---

#### 1.4 Token Upgrade Authority

**Question:** Can HYPE token behavior be modified, and if so, is it controlled by tokenholder governance?

**Investigation Approach:**
1. Identify the HYPE token contract(s)
2. Analyze upgradeability patterns
3. Determine who controls any upgrade paths

**Sources:**
- HyperEVM explorer for HYPE token contract
- HIP-1 documentation (native token standard)

**Evidence Required:**
- HYPE token contract address(es)
- Upgradeability analysis
- Owner/admin identification

**Anticipated Status:** Unknown
- HYPE is a native L1 token, not an EVM contract
- Token behavior likely defined in L1 code (closed source)
- May have HIP-1 wrapper on HyperEVM

**Gaps/Concerns:**
- Native L1 tokens operate differently than ERC-20
- Cannot verify token logic without L1 source code

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
- Assistance Fund documentation
- Fee documentation: https://hyperliquid.gitbook.io/hyperliquid-docs/trading/fees

**Evidence Required:**
- Total supply verification
- Evidence of fixed supply (no mint function)
- Burn mechanism documentation
- Future emission schedule

**Anticipated Status:** Positive
- Fixed 1 billion supply
- Burns occur via Assistance Fund
- Future emissions from allocated reserves, not inflation

**Gaps/Concerns:**
- Cannot verify no mint function exists in L1 code
- Team allocation (23.8%) is large and vesting

---

#### 1.6 Privileged Access Gating

**Question:** Can any bounded actor set block or restrict economically meaningful actions or exit paths?

**Investigation Approach:**
1. Analyze bridge dispute mechanism and locking capability
2. Document validator jailing mechanism
3. Identify any pause/halt functions
4. Check for withdrawal restrictions

**Sources:**
- Bridge2.sol contract
- Bridge documentation
- Staking documentation

**Evidence Required:**
- Bridge locker mechanism analysis
- Validator jailing parameters
- Any evidence of halt/pause capabilities
- Withdrawal restrictions (7-day unstaking queue)

**Anticipated Status:** Warning
- Bridge can be locked by lockers
- 7-day unstaking queue exists
- Validators can be jailed by peer vote

**Gaps/Concerns:**
- Bridge locker identity unclear
- Cold wallet signers for emergency unlock not publicly identified

---

#### 1.7 Token Censorship

**Question:** Can any roles freeze, blacklist, seize, or censor HYPE balances or transfers?

**Investigation Approach:**
1. Search for blacklist/freeze functions in Bridge2
2. Check for restricted address mechanisms on L1
3. Verify no censorship functions in token wrapper contracts

**Sources:**
- Bridge2.sol contract
- HyperEVM token contracts
- L1 documentation

**Evidence Required:**
- Absence of blacklist/freeze functions in Bridge2 (confirmed: none visible)
- L1 censorship capabilities (unknown)

**Anticipated Status:** Unknown/Positive
- Bridge2 does not appear to have blacklist functions
- L1 capabilities cannot be verified

**Gaps/Concerns:**
- Cannot verify L1 does not have address blocking
- Validator set could theoretically censor transactions

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
- Staking documentation
- Assistance Fund address: 0xfefefefefefefefefefefefefefefefefefefefe

**Evidence Required:**
- Active buyback evidence (onchain transactions)
- Staking reward distribution data
- Fee revenue data
- Burn transaction history

**Anticipated Status:** Positive
- 97% of fees to Assistance Fund → HYPE buyback → burn
- Staking rewards active (~2.37% APY at 400M staked)
- Fee discounts for stakers (5-40%)

**Gaps/Concerns:**
- Verify buyback is fully automated, not discretionary
- Quantify actual burn amounts

---

#### 2.2 Treasury Ownership

**Question:** Are protocol treasury assets programmatically controlled by tokenholder governance?

**Investigation Approach:**
1. Identify treasury/Assistance Fund control mechanism
2. Determine if HYPE holders can direct treasury usage
3. Document Hyper Foundation budget (6% allocation)

**Sources:**
- Assistance Fund documentation
- Foundation budget documentation

**Evidence Required:**
- Treasury control mechanism
- Evidence of tokenholder authority over treasury
- Foundation spending discretion

**Anticipated Status:** Neutral/Warning
- Assistance Fund is automated (not discretionary)
- But Foundation budget (6%) appears discretionary
- HLP vault and deployer fees are separate

**Gaps/Concerns:**
- Foundation budget governance unclear
- Cannot verify Assistance Fund automation in L1 code

---

#### 2.3 Accrual Mechanism Control

**Question:** Can only tokenholders modify parameters governing value capture?

**Investigation Approach:**
1. Document who can modify fee parameters
2. Analyze Assistance Fund parameter changes
3. Verify if fee switch exists

**Sources:**
- Fee documentation
- Governance documentation

**Evidence Required:**
- Fee parameter modification authority
- Historical fee changes

**Anticipated Status:** Warning
- Fee parameters likely controlled at L1 level
- Unclear if tokenholder governance can modify fees
- Deployer fees (up to 50%) are a separate flow

**Gaps/Concerns:**
- L1 fee logic not publicly verifiable

---

#### 2.4 Offchain Value Accrual

**Question:** Are there additional offchain value accrual flows benefiting HYPE holders?

**Investigation Approach:**
1. Document any offchain revenue or IP licensing
2. Analyze Hyperliquid Labs business model

**Sources:**
- Corporate filings
- Terms of service

**Evidence Required:**
- Evidence of offchain value flows (or lack thereof)

**Anticipated Status:** Unevaluated
- No evidence of offchain revenue sharing
- Hyperliquid Labs is separate from token

---

### Metric 3: Verifiability

#### 3.1 Token Contract Source Verification

**Question:** Is HYPE token source code publicly available and verifiable?

**Investigation Approach:**
1. Identify HYPE token on HyperEVM (if exists as HIP-1 wrapper)
2. Check verification status on block explorers
3. Analyze native token implementation

**Sources:**
- HyperScan: https://www.hyperscan.com/
- HIP-1 documentation

**Evidence Required:**
- Token contract address (if applicable)
- Source verification status
- Code review

**Anticipated Status:** Unknown
- HYPE is native L1 token (like ETH on Ethereum)
- May have HIP-1 wrapper on HyperEVM
- L1 token logic is not publicly verifiable

**Gaps/Concerns:**
- Native tokens don't have traditional contract verification
- Core L1 code is closed-source

---

#### 3.2 Protocol Component Source Verification

**Question:** Are core protocol contracts publicly accessible and verifiable?

**Investigation Approach:**
1. Verify Bridge2 contract on Arbitrum
2. Check HyperEVM contracts
3. Document what is and isn't open source

**Sources:**
- Bridge2 contract: https://arbiscan.io/address/0x2df1c51e09aecf9cacb7bc98cb1742757f163df7
- GitHub repos: https://github.com/hyperliquid-dex

**Evidence Required:**
- Bridge2 verification status
- List of verified vs. unverified components
- L1 code availability

**Anticipated Status:** Neutral
- Bridge2 source available in GitHub
- SDKs are open source (MIT)
- Node software has Apache 2.0 license
- Core L1 execution/consensus code is closed

**Gaps/Concerns:**
- L1 code not open source
- Cannot verify HyperCore logic

---

### Metric 4: Token Distribution

#### 4.1 Ownership Concentration

**Question:** Does a single actor or coordinated group control a majority of voting supply?

**Investigation Approach:**
1. Analyze token distribution by category
2. Document team/core contributor holdings
3. Identify Foundation holdings
4. Analyze validator stake concentration

**Sources:**
- Tokenomist: https://tokenomist.ai/hyperliquid
- Onchain holder analysis
- Validator stake distribution

**Evidence Required:**
- Distribution breakdown
- Team vesting status (23.8% total, currently ~0.56% unlocked)
- Foundation budget status (6%)
- Top holder analysis

**Anticipated Status:** Warning
- Genesis (31%) was widely distributed
- Team (23.8%) is significant but vesting
- Foundation (6%) + Future emissions (38.89%) are large controlled pools
- Validator concentration needs analysis

**Gaps/Concerns:**
- Team may control validators with unlocked stakes
- Future emissions control is unclear

---

#### 4.2 Future Token Unlocks

**Question:** Are there known future events that will materially affect token concentration?

**Investigation Approach:**
1. Document full unlock schedule
2. Identify key cliff dates
3. Analyze impact on circulating supply

**Sources:**
- Tokenomist unlock schedule
- DefiLlama unlocks

**Evidence Required:**
- Full unlock schedule with dates
- Monthly unlock amounts
- Impact analysis

**Anticipated Status:** Warning
- Core contributor vesting: 24 months starting Nov 2025
- Next unlock: May 6, 2026 (~9.9M HYPE)
- Monthly distributions on 6th of each month
- 76% of supply currently locked

**Gaps/Concerns:**
- Large unlock schedule over 2025-2027
- Future emissions (38.89%) distribution mechanism unclear

---

### Offchain Dependencies

#### 5.1 Trademark

**Question:** Are core trademarks owned by a tokenholder-controlled entity?

**Investigation Approach:**
1. Verify USPTO filing for HYPERLIQUID
2. Identify trademark owner
3. Analyze relationship to token governance

**Sources:**
- USPTO: 99599981
- Hyper Foundation website

**Evidence Required:**
- Trademark filing details
- Owner identification
- Governance relationship

**Anticipated Status:** Warning
- HYPERLIQUID trademark filed by Hyper Foundation
- Foundation is not directly tokenholder-controlled
- Unclear governance relationship

**Gaps/Concerns:**
- Foundation governance not transparent
- No clear mechanism for tokenholders to control trademark

---

#### 5.2 Distribution

**Question:** Are primary domains and interfaces controlled by a tokenholder-controlled entity?

**Investigation Approach:**
1. Identify domain ownership (hyperliquid.xyz, app.hyperliquid.xyz)
2. Review terms of service
3. Analyze who operates the interface

**Sources:**
- Terms of service: https://app.hyperliquid.xyz/terms
- WHOIS records

**Evidence Required:**
- Domain ownership
- Interface operator identity
- Relationship to governance

**Anticipated Status:** Warning
- Likely operated by Hyperliquid Labs Pte. Ltd. (Singapore)
- Labs is separate from tokenholder governance
- Terms of service analysis needed

**Gaps/Concerns:**
- Labs is a private company, not DAO-controlled
- Interface dependency risk

---

#### 5.3 Licensing

**Question:** Is core protocol software owned by a tokenholder-controlled entity?

**Investigation Approach:**
1. Document open source licenses for public code
2. Analyze L1 code ownership
3. Review any IP assignments

**Sources:**
- GitHub repositories
- Corporate filings

**Evidence Required:**
- License terms for all components
- Code ownership identification

**Anticipated Status:** Warning
- SDKs: MIT (permissive)
- Node: Apache 2.0 (permissive)
- Core L1: Closed source (owned by Labs/Foundation?)

**Gaps/Concerns:**
- Core L1 code is not open source
- Labs owns proprietary code
- No clear IP assignment to tokenholder entity

---

## Special Considerations for Hyperliquid L1

### L1 Architecture Implications

Unlike EVM-based protocols, Hyperliquid is an L1 blockchain. This creates unique challenges for the framework:

1. **Native Token vs. Smart Contract:** HYPE is a native token, not an ERC-20. Traditional contract verification doesn't apply.

2. **Consensus-Level Governance:** Governance operates through validator voting, not smart contract execution.

3. **Closed Source Core:** The HyperBFT consensus and HyperCore execution code are not publicly available.

4. **Validator Concentration:** The 24 validator limit and Foundation delegation program concentrate power.

### Evidence Limitations

| Area | Verifiable | Method |
|------|------------|--------|
| Bridge mechanics | Yes | Bridge2.sol on Arbitrum |
| Fee flows | Partial | Onchain transactions, docs |
| Staking rewards | Partial | Documentation, observed behavior |
| Governance voting | Limited | Public announcements, no contract |
| L1 upgrades | No | Closed source |
| Token logic | No | Native token, closed source |
| Validator control | Partial | Observed stake distribution |

### Alternative Verification Methods

Since traditional smart contract verification is limited:

1. **Observe behavior:** Monitor buybacks, burns, staking rewards in practice
2. **Validator analysis:** Map validator stake distribution and identify operators
3. **Documentation review:** Treat docs as claims to verify against behavior
4. **Community governance:** Document governance votes and outcomes
5. **Bridge analysis:** Full verification possible for Arbitrum bridge

---

## Execution Checklist

### Phase 1: Onchain Data Collection
- [ ] Query HyperScan for HYPE-related contracts
- [ ] Analyze Bridge2 contract fully
- [ ] Map validator set and stake distribution
- [ ] Track Assistance Fund buyback/burn transactions
- [ ] Document staking reward distributions

### Phase 2: Documentation Verification
- [ ] Cross-reference docs against observed behavior
- [ ] Document governance votes and outcomes
- [ ] Verify fee flow claims
- [ ] Confirm unlock schedule against vesting tracker

### Phase 3: Legal/Offchain Research
- [ ] Verify trademark filing status
- [ ] Identify Hyperliquid Labs corporate details
- [ ] Analyze terms of service
- [ ] Document Foundation structure (if available)

### Phase 4: Gap Documentation
- [ ] List all claims that cannot be verified
- [ ] Identify closed-source dependencies
- [ ] Document validator control uncertainty
- [ ] Note L1-specific limitations

---

## Expected Outputs

1. **Research Report** (`hype-research.md`): Comprehensive findings with evidence links
2. **tokens.json entry**: Token metadata following schema
3. **metrics.json entry**: Per-criteria evidence and status

## Anticipated Overall Assessment

Based on preliminary research:

- **Onchain Control:** Mixed - validator governance exists but concentrated, L1 closed source
- **Value Accrual:** Positive - active buybacks, burns, staking rewards
- **Verifiability:** Limited - Bridge verifiable, L1 not
- **Distribution:** Warning - significant unlocks pending, team/foundation control
- **Offchain:** Warning - Labs/Foundation control IP, trademarks, interfaces

The HYPE token demonstrates active value accrual mechanisms but faces challenges around verifiability (closed source L1) and concentration (validator control, large locked allocations).
