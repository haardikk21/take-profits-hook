# Limit Orders Part 2

## Recap

- We have functionality for placing orders, cancelling orders, redeeming output tokens from filled orders
- We have a couple basic tests to test placing and cancelling orders
- We do have a `executeOrder` helper that we wrote, its just not being called anywhere right now.

---

## Goals For Today

- We're gonna actually call `executeOrder` somewhere

## Mechanism Design - Part 2

Prices are shifted every time a swap occurs.

We stubbed `afterSwap`

We need to calculate some sort of "tick shift range"

what was the last known tick? what is the new tick?

- Hint #1 = we need a way to keep track of "last known ticks"

---

Something to be careful about

- since we are executing a swap inside of `afterSwap`
  -> our swap will also end up triggering `afterSwap` again
  -> this sort of reentrancy and recursion should not happen
  -> `afterSwap` should not be triggerred if it is being entered because of US

User -> `Router.swap()`
-> `Poolmanager.swap()`
-> `Hook.afterSwap()`
-> `PoolManager.swap()`
-> `Hook.afterSwap()`

Thirdly:

- as we are fulfilling our orders, our orders themselves are also causing a price shift

Alice has an order to sell 5 ETH at 3000 USDC each, current price is like 2950

Bob does a swap to buy some ETH which shifts the price enough to make it 1 ETH = 3050 USDC

At this point, we'll trigger Alice's order

but fulfilling Alice's order will also move the price back down

---

If we have multiple orders that exist within the "original" tick shift range

Current Tick = 500, we have orders at Tick 550 and 540
Bob does a swap, moves Tick up to 600

once we fill one of those orders, its possible the NEW tick is low such that the second order can no longer be filled
