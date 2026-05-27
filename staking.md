# Staking Research Notes

> A core concept in crypto and blockchain. This document starts from basic definitions, outlines categories and mainstream protocols, dives into typical strategies, and ends with a design draft for the xGMB staking system.

---

## Part 1: Concepts & Ecosystem

### 1. What Is Staking?

**Staking**: Lock cryptocurrency on a blockchain network to help it run (validate transactions, produce blocks, etc.). The network pays extra tokens as a reward.

Often compared to “deposit in a bank for interest,” but the mechanics differ.


| Dimension           | Description                                                                                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Core mechanism**  | Based on **PoS (Proof of Stake)** and variants. The network picks who validates the next block by staked amount and duration; more stake → higher selection probability. |
| **Role**            | Stakers are the network’s “security guardians.” Tokens act as collateral; misbehavior can lead to partial forfeiture (**Slash**).                                        |
| **For users**       | Passive yield (interest-like).                                                                                                                                           |
| **For the network** | Stronger security and decentralization; more energy-efficient and scalable than PoW.                                                                                     |


**Everyday analogy**: A neighborhood runs “owner on duty” — you pledge your property deed (tokens) with the HOA, take weekly security shifts, earn points; negligence costs points. That’s staking.

---

### 2. Staking Categories (by participation style)

The dimensions most retail users encounter:


| Category              | English           | Description                                                                   | Best for                                     | Pros & cons                                                                                                                                  |
| --------------------- | ----------------- | ----------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **Solo staking**      | Solo Staking      | Run your own node and validate directly (e.g. Ethereum 32 ETH)                | Technical users, large capital, full control | **Pros**: Higher yield, direct network support. **Cons**: High bar (capital + ops + hardware), long lockups, Slash risk.                     |
| **Delegated staking** | Delegated Staking | Delegate tokens to reputable validators; share rewards                        | Most retail investors                        | **Pros**: Low barrier, simple. **Cons**: Must trust the node; node takes a commission.                                                       |
| **CEX staking**       | CEX Staking       | One-click stake on Binance, Coinbase, etc. (exchange delegates or runs nodes) | Beginners, convenience                       | **Pros**: Very easy; some products stay liquid. **Cons**: Slightly lower yield; exchange risk (“not your keys, not your coins”).             |
| **Liquid staking**    | Liquid Staking    | Receive a liquid “receipt” token (LST) while staked; use in DeFi              | Advanced DeFi users                          | **Pros**: Unlocks liquidity, stackable yield. **Cons**: Extra smart-contract risk; LST may trade at a discount. Examples: Lido, Rocket Pool. |


---

### 3. Mainstream Protocol Landscape

Cross-protocol comparison of staking-related models, scale, and recent moves.


| Project         | Business model                 | Liquidity token (LST/LRT) | TVL & highlights                       | Recent moves / innovation                                                |
| --------------- | ------------------------------ | ------------------------- | -------------------------------------- | ------------------------------------------------------------------------ |
| **Lido**        | Liquid staking                 | stETH                     | TVL ~$40B; Ethereum LST leader         | V3 modular vaults stVaults; community staking V2, Staking Router V3      |
| **Rocket Pool** | Liquid staking                 | rETH                      | TVL ~$1.5B; decentralization benchmark | Lower node operator barriers                                             |
| **Ankr**        | Liquid staking                 | aETH, etc. (ankrETH)      | Early cross-chain staking              | Ankr Network 2.0, stronger decentralized infra                           |
| **Aave**        | Lending & safety staking       | aTokens / GHO             | TVL ~$20B                              | **Umbrella** safety module; aToken staking replaces legacy safety module |
| **Morpho**      | Lending & staking optimization | Vault shares              | TVL ~$6B                               | V2 fixed/floating rates, long-tail collateral                            |
| **Compound**    | Lending & governance staking   | cTokens                   | Stable TVL                             | Non-liquid COMP staking design to strengthen governance                  |
| **Curve**       | DEX & governance staking       | veCRV                     | TVL ~$2.1B                             | Originator of **vote-escrow (ve)** model                                 |
| **EigenLayer**  | Restaking                      | LRT (e.g. eETH)           | TVL ~$20.1B                            | AVS ecosystem live; EIGEN as universal security layer                    |
| **Pendle**      | Yield derivatives              | PT/YT (on LSTs)           | TVL ~$5B                               | Tokenizes staking yield into YT/PT                                       |
| **Ether.fi**    | Non-custodial liquid staking   | eETH                      | TVL ~$9.5B                             | Deep EigenLayer integration                                              |
| **Jito**        | Liquid staking (Solana)        | JitoSOL                   | >14M SOL (>40% share)                  | MEV revenue + restaking                                                  |


---

### 4. Core Protocol Deep Dives

#### 4.1 Lido: Liquid Staking

Lido is Ethereum’s largest liquid staking protocol. Core ideas: **liquidity for staked assets**, **decentralized node allocation**, **rebase yield**, **withdrawal queue**.
Similar: Rocket Pool (rETH), Ankr (aETH).

##### 4.1.1 Deposit & Mint

- **1:1 mint**: User deposits ETH into `StakingRouter`; contract mints **stETH** 1:1.
- **Buffer pool**: ETH queues in the mainnet deposit pool before going to the beacon chain.

##### 4.1.2 Validator Registration & Fund Distribution

Beacon chain requires **32 ETH** per validator. Lido routes the pool via a **non-custodial operator network**:

- **Node operators**: Generate validator keys; hold **signing keys** only, not **withdrawal keys**.
- **32 ETH batches**: When the buffer hits a multiple of 32 ETH, the contract calls the official `DepositContract` to activate validators.

##### 4.1.3 Rewards & Rebase

- **Daily oracle**: Tracks total assets under Lido validators (principal + rewards − penalties).
- **Balance rebase**: Mainnet contract increases all stETH balances proportionally — **no manual claim**.
- **Fees**: 10% of gross rewards (5% operators + 5% DAO); 90% to stETH holders.

##### 4.1.4 Withdrawal Engine

Post-Shanghai, full two-way withdrawals via queue and **NFT receipts**:

```
[User stETH] → (lock & burn) → [Withdrawal NFT minted] → (wait for node ETH release) → [Burn NFT, receive ETH]
```


| Step          | Description                                               |
| ------------- | --------------------------------------------------------- |
| 1. Request    | Redeem stETH; lock shares; stop rebase                    |
| 2. Mint NFT   | ERC-721 withdrawal voucher; tradable on secondary markets |
| 3. Source ETH | Node exit and/or buffer pool + daily rewards to fill gap  |
| 4. Claim      | When queue clears, burn NFT and receive ETH               |


**Design essence**: Turn non-fungible “32 ETH validator + async beacon rewards” into fungible, composable **ERC-20 stETH**. Withdrawal credentials are protocol-controlled, reducing operator runaway risk.

---

#### 4.2 Lending, Yield Trading & Restaking: Aave / Pendle / EigenLayer

Three different roles on one value chain:


| Protocol       | Role                                                            |
| -------------- | --------------------------------------------------------------- |
| **Aave**       | Money pool — “bank” for base lending yield                      |
| **Pendle**     | Derivatives market — trade “future interest”                    |
| **EigenLayer** | Security service — “resell” security from already-staked assets |


##### Aave: Supply / Borrow Staking

- **Mechanism**: Supply ETH, USDC, etc.; receive **aTokens** (e.g. aETH); balance grows with interest; borrowers over-collateralize and pay interest.
- **Risks**: Smart contract bugs; borrower liquidation (lenders usually not directly exposed); rates fluctuate with supply/demand.
- **UX**: Simple; usually redeem anytime if pool liquidity allows.
- **Position**: DeFi base “yield layer”; stETH/ETH in Aave is often step one for complex strategies.

##### Pendle: Yield Tokenization

Splits yield-bearing assets into two tokens:


| Token                    | Meaning                                                                                   | Best for                   |
| ------------------------ | ----------------------------------------------------------------------------------------- | -------------------------- |
| **PT (Principal Token)** | Principal; 1:1 redeem underlying at maturity; often discounted (e.g. buy $1 PT for $0.95) | Lock fixed yield           |
| **YT (Yield Token)**     | All future yield; price moves with yield expectations                                     | Long or speculate on rates |


- **Risks**: Steep learning curve; YT loses if rates fall; PT gives up upside if rates spike; PT/YT expire — YT goes to zero.
- **Governance staking**: Upgraded to **sPENDLE** (liquid staking); min unlock ~**14 days**; stakers get ~**80% of protocol revenue** plus governance.
- **Position**: Does not create base yield; trades yield streams from Lido, Aave, etc.

##### EigenLayer: Restaking

- **Mechanism**: Already-staked ETH/LST can be re-delegated to EigenLayer operators to secure **AVSs** (DA, oracles, bridges, etc.) for extra rewards.
- **Risks**: **Slash** = principal loss; diversify across operators; native restaking withdrawal delays (e.g. 7 days); EIGEN unlock up to 24 days.
- **Position**: “Security marketplace” — higher yield ceiling, slashing dimension added.

##### Side-by-Side


| Feature            | Aave                                   | Pendle                          | EigenLayer                     |
| ------------------ | -------------------------------------- | ------------------------------- | ------------------------------ |
| **Core role**      | Bank: lend/borrow yield                | Exchange: trade “interest”      | Security: reuse staked assets  |
| **What you stake** | ETH, USDC, DAI, etc.                   | Yield assets (aToken, stETH)    | Already-staked ETH / LST       |
| **Yield source**   | Borrow interest                        | PT/YT spread, incentives        | AVS tokens/fees                |
| **Main use**       | Base yield, liquidity                  | Lock yield (PT), speculate (YT) | Shared security, extra rewards |
| **Top risk**       | Contract bugs, liquidation (borrowers) | Rate moves, YT → 0              | **Slash**, bad operators       |
| **Barrier**        | Low                                    | Medium–high                     | High                           |


##### Stacked Strategies (“Matryoshka”)

Typical stack:

1. **Lido** ETH → **stETH** (base staking yield)
2. **stETH** into **EigenLayer** → restaking rewards + points (layer 2)
3. Restaking receipt or stETH into **Pendle** PT/YT → sell YT to lock future yield early

One asset can be “extracted” across layers; risk stacks too.

# Crypto-Economic Infrastructure & DeFi Protocol Classification Matrix


| Layer / Protocol Class                    | Protocols                   | Core Mechanism & Architectural Purpose                                                                                                                                           | Framework Alignment & Nuance                                                                                                                                                                                               |
| ----------------------------------------- | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LST** *(Liquid Staking Tokens)*         | **Lido, Rocket Pool, Ankr** | Users deposit native Layer 1 tokens ($ETH) to receive a liquid voucher ($stETH, $rETH, $ankrETH) that programmatically accrues base-layer network consensus rewards.             | **Pure-Play LST Base Layer.** These protocols focus exclusively on bootstrapping and decentralizing the primary validation layer of the Ethereum blockchain.                                                               |
| **LRT** *(Liquid Restaking Tokens)*       | **Ether.fi**                | Deposits validate the base layer **and** are programmatically routed into EigenLayer strategy contracts to secure multiple external networks concurrently.                       | **Multi-Layer Yield Primitive.** Adds a secondary layer of programmatic risk and reward. It exposes capital to external slashing parameters in exchange for enhanced yield and incentive points.                           |
| **Shared Security Marketplace**           | **EigenLayer**              | A core programmatic trust layer that unbundles Ethereum's economic security, allowing external decentralized networks (AVSs) to lease safety from restaked assets.               | **B2B Trust Infrastructure.** It is not a user-facing yield application or money market; it functions as a foundational hardware and cryptographic orchestration engine.                                                   |
| **Money Markets** *(Lending & Borrowing)* | **Aave, Compound, Morpho**  | Over-collateralized liquidity pools or matching engines that facilitate capital efficiency by connecting suppliers (earning interest) with borrowers (paying interest).          | **Credit Infrastructure.** While Aave and Compound use shared, DAO-managed multi-asset pools, **Morpho** operates as a hyper-optimized peer-to-peer matching layer and isolated-risk vault system.                         |
| **AMM / Liquidity Layer**                 | **Curve**                   | An automated market maker (DEX) specifically engineered with stableswap invariant curves to execute low-slippage trades between like-valued assets.                              | **Liquidity Hub.** Unlike lending protocols, capital deposited or locked here is used to facilitate decentralized asset exchange, asset peg defense, and market making.                                                    |
| **Yield Derivatives** *(Yield Stripping)* | **Pendle**                  | Tokenizes yield-bearing assets into two distinct components: Principal Tokens (**PT** for fixed-yield locking) and Yield Tokens (**YT** for leveraged yield/points speculation). | **Financial Meta-Layer.** Pendle is asset-agnostic. It does not generate restaking infrastructure itself; instead, it wraps yield-bearing assets (from LSTs, LRTs, or money markets) into tradeable financial derivatives. |
| **Cross-Ecosystem Restaking**             | **Jito**                    | A Solana-native hub executing both liquid staking ($JitoSOL) and multi-asset restaking architectures via modular Node Consensus Networks (NCNs).                                 | **SVM Sovereign Equivalent.** Represents a hybrid of Lido and EigenLayer built specifically for the high-throughput, MEV-intensive architecture of the Solana Virtual Machine.                                             |


---

### 5. Summary & Strategic Takeaways

Staking has evolved from single-chain PoS validation into a multi-layer, multi-chain yield stack. Rough phases:


| Phase / paradigm  | Representative | Traits                   |
| ----------------- | -------------- | ------------------------ |
| Staking 1.0       | Lido           | Liquid staking, LST      |
| Security as yield | Aave Umbrella  | Safety-module staking    |
| Restaking         | EigenLayer     | Shared Ethereum security |
| Yield derivatives | Pendle         | PT/YT yield splitting    |


**For builders / developers:**

- **Separate liquidity**: Minting LSTs to free capital is the dominant design pattern.
- **Novel safety modules**: Like Aave Umbrella — stake assets to protect the ecosystem and attract TVL.
- **More yield sources**: EigenLayer partnerships, cross-chain staking, MEV sharing, etc.
- **Governance alignment**: veToken or real rights (fee share, votes) for long-term holders.
- **Bitcoin ecosystem**: Babylon and similar let BTC holders earn via staking — worth early attention.

---

## Part 2:  Staking System Architecture (ERC-4626)

Build the staking vault on **ERC-4626**: a standard tokenized vault interface. Users deposit ERC20 and receive yield-bearing share token **sERC20**, laying groundwork for revenue and profit sharing.

Attacks:

**Inflation attack**: Attacker deposits a tiny amount then donates heavily, skewing the rate so the next small depositor gets zero shares.

And this code version of ERC4626 in Openzeppling can not withstand this attack.

```
    /**
     * @dev Internal conversion function (from shares to assets) with support for rounding direction.
     */
    function _convertToAssets(uint256 shares, Math.Rounding rounding) internal view virtual returns (uint256 assets) {
        uint256 supply = totalSupply();
        return
            (supply == 0) ? _initialConvertToAssets(shares, rounding) : shares.mulDiv(totalAssets(), supply, rounding);
    }

    /**
     * @dev Internal conversion function (from shares to assets) to apply when the vault is empty.
     *
     * NOTE: Make sure to keep this function consistent with {_initialConvertToShares} when overriding it.
     */
    function _initialConvertToAssets(
        uint256 shares,
        Math.Rounding /*rounding*/
    ) internal view virtual returns (uint256 assets) {
        return shares;
    }
```

Correct one

```
    function _convertToShares(uint256 assets, Math.Rounding rounding) internal view virtual returns (uint256) {
        return assets.mulDiv(
            totalSupply() + 10 ** _decimalsOffset(), // Adds Virtual Shares
            totalAssets() + 1,                       // Adds Virtual Assets
            rounding
        );
    }
```

 Version (4.8.3), new version this problem is solved.

