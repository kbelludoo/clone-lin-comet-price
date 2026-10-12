# clone-lin-comet-price

Experimental i64-scale LIN clone of [compound-finance/comet](https://github.com/compound-finance/comet) `contracts/CometWithExtendedAssetList.sol` (`mulPrice` / `divPrice` / `isBorrowCollateralized` / `absorbInternal` price wrap) plus `CometCore.sol` scales (`PRICE_FEED_DECIMALS=8`, `PRICE_SCALE=1e8`, `FACTOR_SCALE=1e18`, `BASE_INDEX_SCALE=1e15`).

- **LIN clone:** [`src/lin_comet_price.lin`](src/lin_comet_price.lin)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_COMET_PRICE_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** Compound III, **not** a Comet / USDC market / Chainlink PriceFeed, and does **not** claim Solidity uint256 / mainnet parity. It is **not** the tracking-index kernel (`clone-lin-comet-track`), **not** the getUtilization kernel (`clone-lin-comet-util`), **not** the presentValue/principalValue kernel (`clone-lin-comet-pv`), and **not** the accruedInterestIndices kernel (`clone-lin-comet-accrue`).

Upstream was BUSL-1.1 with Change Date 2025-12-31; the Change License is GPL-2.0-or-later (this run is 2026-10-12). This LIN kernel is an original uint64-scale implementation of the published price wrap, not a verbatim Solidity dump.

Pinned upstream: commit `f766f51583c23acc33b2a7824654ef2029a96804`.

Canonical goldens (uint64-scale, 128-bit product):

- `mul_price(1e6, 1e8, 1e6)` = **1e8** (1 USDC at $1 → common-price units)
- `factor_as_price(1e6, 1e6)` = **1e18** (~1e10× if FACTOR_SCALE is used instead of the USD price)
- `index_as_price(1e6, 1e6)` = **1e15** (~1e7× if BASE_INDEX_SCALE is used as the price)
- `mulfactor_as_price(1e6, 1e8)` = **0** (`mulFactor(n, price)` uses /1e18)
- `mul_price(3, 5, 4)` = **3**; `reorder(3, 5, 4)` = **0** (rejected `(n/fromScale)*price` path)
- `mul_price(1e8, 6e12, 1e8)` = **6e12** and `mul_price(1e8, 6e12, PRICE_SCALE)` = **6e12** (WBTC 8-dec hides PRICE_SCALE vs asset.scale)
- `mul_price(1e18, 2e11, 1e18)` = **2e11**; `mul_price(1e18, 2e11, 1e8)` = **0** (WETH 18-dec / PRICE_SCALE overflows uint64)
- `div_price(1e8, 1e8, 1e6)` = **1e6** ($1 → 1 USDC); `toScale=PRICE_SCALE` → **1e8** (100× payout)
- `is_borrow_ok(200e6, 1e8, 1e6, 0.1 WETH, $2000, 1e18, CF=50%)` = **0**; skip-CF = **1**
- `absorb_base(0.1 WETH, $2000, LF=80%, USDC $1)` = **160e6** (160 USDC)
- `1e18 * 1e18 / 1` fail-closes to **0**
