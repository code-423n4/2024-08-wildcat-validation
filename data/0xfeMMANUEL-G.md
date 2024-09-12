#Gas optimisation for the liquidityRequired function

##you should try to minimize redundant computations by storing intermediate results. For instance, in the liquidityRequired function, you should store the result of state.scaledTotalSupply - scaledWithdrawals in a variable instead of computing it many times, i have shown a sample code example below.



###my sample code
function liquidityRequired(
  MarketState memory state
) internal pure returns (uint256 _liquidityRequired) {
  uint256 scaledWithdrawals = state.scaledPendingWithdrawals;
  uint256 outstandingDebt = state.scaledTotalSupply - scaledWithdrawals;
  uint256 scaledRequiredReserves = outstandingDebt.bipMul(state.reserveRatioBips) + scaledWithdrawals;
  uint256 normalizedReserves = state.normalizeAmount(scaledRequiredReserves);
  return normalizedReserves + state.accruedProtocolFees + state.normalizedUnclaimedWithdrawals;
} 


###github code reference
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/libraries/MarketState.sol#L130-L136

###
 By storing intermediate results such as scaledWithdrawals, outstandingDebt, scaledRequiredReserves, and normalizedReserves, the function avoids redundant computations and multiple accesses to the state variables.

##IN SUMMMARY
Complex operations like bipMul and normalizeAmount are computed once and stored, reducing the need to repeat these operations multiple times.
Complex operations like bipMul and normalizeAmount are computed once and stored, reducing the need to repeat these operations multiple times.
The final calculation of liquidityRequired aggregates the precomputed values, ensuring that each value is computed  once only and used very efficiently.