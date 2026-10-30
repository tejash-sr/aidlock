# AidLock — disaster relief money that can only be spent on relief

> A stablecoin for disaster aid where the rules live in the money itself. Only **verified victims** receive it, they can only spend it at **approved merchants** for food, medicine, shelter, education or utilities, and **every payment is publicly auditable** on the blockchain.

**Built solo** for the **IIT EBIS 2.0 hackathon** (Bybit – Real World Assets & DeFi track) · Solidity smart contracts on Ethereum (Hardhat, OpenZeppelin) · web dashboard with ethers.js · **35 automated contract tests, all passing**

---

## The problem

When a disaster hits, relief money passes through many hands before it reaches the people who need it:

- **Slow:** each intermediary adds days or weeks while families need food and medicine now.
- **Leaky:** once cash or a bank transfer is handed out, nobody can control or see what it is spent on.
- **Opaque:** donors give money and then lose sight of it. Without proof of where it went, trust (and future donations) drops.

## The solution: purpose-bound money

AidLock issues **ReliefUSD (RUSD)**, a custom ERC-20 stablecoin (1 RUSD = 1 unit of aid) whose transfers are checked against relief rules on every payment:

| Who | What they can do |
|---|---|
| **Relief agency (admin)** | Verifies beneficiaries, approves merchants and assigns each a spending category, issues RUSD, can pause the whole system in an emergency |
| **Beneficiary** (verified victim) | Receives RUSD and can pay **only approved merchants**. Payments to any other wallet, including other beneficiaries, are rejected by the contract |
| **Merchant** (approved shop, pharmacy, shelter…) | Accepts RUSD in its category: food, medical, shelter, education or utilities |
| **Donor / public** | Sees distributions and payments on the public dashboard. No wallet needed |

```
Agency mints 500 RUSD to a verified family
Family  → "City Grocery" (approved, FOOD)       ✅ allowed, recorded under FOOD
Family  → random wallet                         ❌ rejected by the contract
Family  → another beneficiary (to cash out)     ❌ rejected by the contract
```

## Why it matters

- **Aid can't be diverted.** The spending rule is enforced by the token contract on every transfer, so money meant for relief can't quietly be moved elsewhere.
- **Instant delivery.** Funds reach a verified family in one transaction, without a chain of intermediaries.
- **Proof for donors.** Every distribution and payment is recorded on-chain with amount, merchant and category, so the agency can show exactly where money went. The dashboard exports the audit trail as CSV or PDF.
- **Reporting by category, automatically.** Because each payment carries the merchant's category, "how much went to food vs medicine" comes straight from the chain instead of from receipts.

## Key design decisions

| Decision | Why |
|---|---|
| **Rules in the token, not in an app** (`transfer` asks `ReliefFundManager.isTransferAllowed` before moving funds) | An app can be bypassed; a rule in the token contract applies to every wallet and every interface |
| **Permissioned, not open** (the agency verifies beneficiaries and merchants) | Relief needs accountability; anonymous recipients would invite fraud |
| **Merchants carry the category** | Spending is classified at the point of sale, so reports by category need no manual tagging |
| **Emergency pause** (OpenZeppelin `Pausable`) | If a key is compromised or something goes wrong, all minting and transfers can be frozen at once |
| **Audited building blocks** (OpenZeppelin `ERC20`, `Ownable`, `Pausable`) | Security-critical code reuses battle-tested libraries instead of hand-written token logic |
| **Public view without a wallet** | Donors and journalists shouldn't need MetaMask to check where the money went |

## What's in the box

| Part | Details |
|---|---|
| `contracts/ReliefStablecoin.sol` | RUSD token: mint, burn, pause, and transfer checks against the fund manager |
| `contracts/ReliefFundManager.sol` | Beneficiary and merchant registry, the transfer rule, spending categories and the on-chain transaction history |
| `contracts/ReliefStablecoinV2.sol` | **Extended design (compiles, not yet deployed or tested):** multi-authority approval for minting, time-locked aid that expires and can be reclaimed, emergency and recovery modes, a beneficiary proof-of-life check, per-category spending totals, and a traceable journey for every token |
| `frontend/` | Dashboard (ethers.js + MetaMask): admin, beneficiary and merchant views, a public view-only mode with a map of aid (Leaflet), donation flow and audit export (CSV, PDF) |
| `test/ReliefSystem.test.js` | 35 tests: token basics, pausing, beneficiary and merchant management, the transfer rules, and the audit trail |

## Run it

**Prerequisites:** Node.js 18+, and MetaMask for the wallet views.

```bash
npm install
npm test                 # 35 contract tests

npm run node             # terminal 1: local blockchain (Hardhat)
npm run deploy:local     # terminal 2: deploy the token and fund manager
npm run frontend         # terminal 3: dashboard on http://localhost:3000
```

In MetaMask, add the local network (`http://127.0.0.1:8545`, chain ID 31337) and import one of the test accounts that `npm run node` prints. Account #0 is the relief agency (admin).

**Deploy to the Sepolia testnet:** put `SEPOLIA_RPC_URL` and `PRIVATE_KEY` in `.env`, then run `npm run deploy:sepolia`.

## Demo in 3 minutes

1. **Agency:** connect as account #0 and open the admin dashboard.
2. **Verify a victim:** add account #1 as a beneficiary.
3. **Approve a shop:** add account #2 as a FOOD merchant, "City Grocery".
4. **Send aid:** mint 500 RUSD to the beneficiary.
5. **Spend correctly:** as the beneficiary, pay City Grocery. It succeeds and appears under FOOD.
6. **Try to divert it:** pay a random wallet. The contract rejects it.
7. **Show the proof:** open the public view and export the audit trail.

The full walkthrough is in [`HACKATHON_DEMO_GUIDE.md`](HACKATHON_DEMO_GUIDE.md); the no-wallet public view is described in [`PUBLIC_VIEW_GUIDE.md`](PUBLIC_VIEW_GUIDE.md).

## Project layout

```
contracts/      ReliefStablecoin, ReliefFundManager, ReliefStablecoinV2
scripts/        deploy.js (local and Sepolia)
test/           ReliefSystem.test.js (35 tests)
frontend/       index.html, app.js, abi.js, config.js, styles.css
deployments/    addresses from the last local deployment
```

**Stack:** Solidity 0.8.24 · Hardhat · OpenZeppelin Contracts · ethers.js 6 · Leaflet · MetaMask
