
---

| Category | Core Financial Function | How It Functions Mechanism | Key Onchain Projects |
| --- | --- | --- | --- |
| **Automated Market Makers (AMM)** | **Spot Liquidity & Trading** | Replaces order books with liquidity pools governed by deterministic math formulas (like $x \cdot y = k$). | Uniswap, Curve, Balancer |
| **Lending & Borrowing** | **Credit & Leverage** | Uses over-collateralized pools and dynamic interest rate curves to let users rent capital or draw liquidity without selling assets. | Aave, Compound, Morpho |
| **Liquid Staking (LST)** | **Capital Efficiency** | Unlocks the liquidity of locked Proof-of-Stake (PoS) assets by issuing a wrapping, yield-bearing derivative token. | Lido (`stETH`), Rocket Pool (`rETH`), Jito |
| **Restaking / LRT** | **Security Exporting** | Takes staked assets or LSTs and pledges them to a shared registry to secure external infrastructure layers for extra yield. | EigenLayer, Ether.fi, Renzo |
| **Perpetuals & Derivatives** | **Leveraged Risk Trading** | Enables shorting and leverage via virtual AMMs (vAMMs) or oracle-priced liquidity vaults instead of a central clearing house. | Hyperliquid, dYdX, GMX |
| **Real-World Assets (RWA)** | **Offchain Capital Import** | Brings traditional financial instruments like US Treasuries, private credit, or commodities onchain via tokenized legal structures. | Ondo Finance, Centrifuge, Maker (Sky) |
| **Yield Tokenization** | **Fixed Income & Speculation** | Strips a yield-bearing asset into a fixed-rate Principal Token (PT) and a volatile, pure-yield Yield Token (YT). | Pendle Finance |
| **Yield Aggregators** | **Automated Asset Management** | Programs smart contract vaults to auto-rotate capital across different lending and AMM pools to capture the highest optimal return. | Yearn Finance, Beefy |

---

> **The DeFi Stack Perspective:** If you look closely at how these interact, they form a strict structural dependency. AMMs and Lending form the *base liquidity layer*. Liquid Staking wraps that base layer, Restaking exports its security, and Yield Tokenization (like Pendle) cuts up those resulting yields into distinct cash flows.
