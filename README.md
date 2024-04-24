# Take Profits Hook

This hook was designed as part of the course curriculum for the Uniswap Hook Incubator.

## Notes

To pass the test suite, we must comment out two lines from `v4-core/src/test/PoolSwapTest.sol`

```
// assertEq(reserveBefore0, reserveAfter0);
// assertEq(reserveBefore1, reserveAfter1);
```

These two lines ensure that the reserve balances of currencies in the pool haven't changed before and after the swap - prior to the router settling balances. But our hook can execute orders inside of a swap (in `afterSwap`) which can affect the reserve balances - so the tests will not pass if these two lines exist.
