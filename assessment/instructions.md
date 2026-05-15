# World Cup on-chain betting: candidate instructions

## Your task

Implement **`contracts/contracts/WorldCupBetting.sol`**.

The file you start from is a **stub**: every function reverts with `WorldCupBetting: candidate implementation required`. Your job is to replace that with working logic so the assessment tests pass.

Follow any extra rules you were given (for example whether you may read other contracts in the repo, time limits, and what to submit).

---

## Time and stack

- **Time**: use the limit you were assigned (often a few hours for a full pass).
- **Stack**: Hardhat, TypeScript tests, Solidity `^0.8.30`, OpenZeppelin (`ReentrancyGuard`, `Ownable`).

---

## Setup

1. From the repo root, go into the contracts package and install dependencies. If npm reports a `chai` peer conflict, use:

   ```bash
   cd contracts
   npm install --legacy-peer-deps
   ```

2. If Hardhat prints **HH801** and asks for missing toolbox packages, install the versions it lists, then try again.

3. Optional compile:

   ```bash
   npx hardhat compile
   ```

---

## How to run the assessment

Run only the World Cup assessment tests:

```bash
cd contracts
npx hardhat test
```

On an unimplemented stub, **all tests should fail**. When your `WorldCupBetting.sol` is correct, **all tests should pass**.

You may read **`test/WorldCupBetting.assessment.test.ts`** to see what is expected (scenarios A through I).

---

## What your contract must support

The tests call the public API on `WorldCupBetting` and check revert strings in several places. Implement behavior that matches what those tests do.

| Area | Functions to implement |
| --- | --- |
| Lifecycle | `createMarket`, `placeBet`, `resolveMarket`, `claimWinnings` |
| Access | Only the market's arbitrator may resolve; only the owner may `withdrawFees` |
| Time | Allow bets while `block.timestamp < resolutionTime`; allow resolution only when `block.timestamp >= resolutionTime` |
| Collateral | `tokenAddress == address(0)` means native ETH (`msg.value`); otherwise use ERC20 `transferFrom` / `transfer` on that token |
| Fees | Take a **2%** platform fee on the winning payout path; track it in `getAvailableFees` and pay it out via `withdrawFees` |
| Secondary market | `listPosition`, `buyPosition` (and `cancelListing` exists on the stub for parity) |
| Reputation | From `claimWinnings`, call `ReputationSystem.updateReputation` for both winning and losing claims, as the tests assume |

Revert messages the tests assert include, among others: `Too early`, `Only arbitrator`, `Market closed`, `Slippage exceeded`, `Already claimed`. Match those strings where the tests use `revertedWith(...)`.

---

## What each scenario is about

| ID | Idea |
| --- | --- |
| **A** | Three-way match result (1X2), then resolve and read `getMarket` status |
| **B** | Two-sided ETH pool, winner paid net of fee, owner withdraws ETH fees |
| **C** | Cannot resolve before `resolutionTime` |
| **D** | Only the arbitrator can resolve |
| **E** | No new bets at or after the resolution timestamp |
| **F** | `placeBet` enforces `_minShares` (slippage) |
| **G** | List a position, someone else buys it, that buyer claims if the outcome wins |
| **H** | Same flows using an ERC20 as collateral |
| **I** | Losing side still runs through `claimWinnings`; cannot claim twice |

---

## Before you submit

- [ ] `npx hardhat test test/WorldCupBetting.assessment.test.ts` passes with your `WorldCupBetting.sol`.
- [ ] You can explain how fees work in **Scenario B** and how ownership moves in **Scenario G**.

Good luck.

# WorldCupBetting Assessment Submission

## Test Status

The assessment test suite passes successfully using:

```bash
npx hardhat test test/WorldCupBetting.assessment.test.ts
```

---

# Scenario B – Fee Mechanism

The contract applies a 2% platform fee on winning payouts.

## Fee Flow

1. Users place bets into the market pool.
2. After market resolution, winning users claim rewards proportional to their shares.
3. During payout calculation:
   - Gross payout is calculated from the total pool.
   - A 2% platform fee is deducted.
   - Remaining amount is transferred to the winner.

Example:

- Total payout: 10 ETH
- Platform fee (2%): 0.2 ETH
- User receives: 9.8 ETH

Collected fees are tracked in:

```solidity
mapping(address => uint256) public collectedFees;
```

- `address(0)` is used for ETH fees.
- ERC20 token address is used for token-based fees.

The contract owner can withdraw accumulated fees using:

```solidity
withdrawFees(address tokenAddress)
```

---

# Scenario G – Position Ownership Transfer

The contract supports a secondary market for trading betting positions.

## Ownership Flow

1. A bettor lists their position using:

```solidity
listPosition(betId, price)
```

2. Another user purchases the position using:

```solidity
buyPosition(betId)
```

3. During purchase:
   - Payment is transferred to the seller.
   - Ownership of the bet is transferred to the buyer by updating:

```solidity
bet.bettor = msg.sender;
```

4. The new owner becomes eligible to:
   - claim winnings
   - receive payout
   - interact with the position

This behavior is validated in Scenario G where the buyer successfully claims winnings after purchasing a winning position.

---

# Deployment

The contract was successfully deployed to Ethereum Sepolia testnet.

## Deployed Contracts

- ReputationSystem: `0xE8212596A7391f2Ac4B0060296A863a210b0c483`
- WorldCupBetting / PredictionMarket:
  `0xFaBB2f32D52bc9b52c78074C25481D66B936ab13`

---
