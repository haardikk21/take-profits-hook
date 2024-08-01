# Limit Orders - Part 1

Limit Order based hook - how can we create a hook that behaves like a simepl orderbook on chain
and benefits from being integrated into Uniswap

Part 1, there is a Part 2 of this workshop on Monday

---

We're going to build out a "Take Profit" style limit order

Its a type of limit order where you want to sell an asset at a price higher than its current price

if right now 1 ETH = 3000 USDC, lets say

a TP order would be something like "sell 5 ETH of mine when ETH = 3500 USDC"

---

Today:

1. building the key functionality required around being able to place limit orders, cancel orders that are placed, some of the core logic for executing those orders

Monday

1. writing code to figure out "how" and "when" to execute those orders
   -> actual hook functions will come into play

## Mechanism Design

At a very high level

1. Ability to place an order
2. Ability to cancel an order (if it hasnt already been filled yet)
3. Ability to withdraw/redeem the output tokens that we get from executing an order

None of these things have anything to do particularly with Uniswap's core codebase.
-> these will all be public functions on the hook contract that folks can call directly

---

Assuming an order has now been placed:

1. When the price is right, how do we actually execute it? How do we do this as a part of a hook?
2. How do we know th price is right? Figuring out the "when" to execute the order
3. How do we send/let the user redeem their output tokens from their order

---

Assume there's a pool of tokens A and B. We'll treat A as the Token 0, and B as the Token 1.

Let's assume the current tick of the pool is 500.

Tick being 500 means A is more valuable than B right now.

The types of TP orders that can be placed:

1. Sell some amount of `A` as `A` gets even more valuable than B
2. Sell some amount of `B` as `B` gets more valuable (A dropping in value)

Case 1. Tick goes up even further, beyond what it is right now (500)
Case 2. Tick goes down, below what is right now (500)

When do these tick values change? Swaps

1. Alice places an order to sell some `A` for `B` when Tick = 600
2. Bob comes around and does a swap on the pool to buy `A` for `B`, which increases the tick.
   -> Lets say after Bob's swap, new tick = 700
3. Inside the `afterSwap` hook function, we can see this happening
   -> Tick just shifted from 500 to 700 because of Bob's swap
4. Check if we have any TP orders placed in the opposite direction in the range that the tick was just shifted, and we'll find Alice's order over there
5. We can execute Alice's order now because her requirements have been met

## Assumptions

1. We are not really going to be concerned about gas costs/gas limits.
   1.1. Bob is going to be the one paying gas to execute Alice's order in the above example.
   1.2. We are not going to limit how many orders we execute because of a price shift

2. We will ignore slippage requirements for placed limit orders
   2.1. When alice places her order, ideally - she should also set some sort of slippage (some sort of minimum token B to get back).

3. We're not going to support pools that have native ETH token as one of the currencies. We only support ERC20-ERC20 pools.
