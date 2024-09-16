# GAS OPTIMISATION FOR FUNCTIONS IN THE src/libraries/MarkertState.sol

## Gas optimisation for the liquidityRequired function

You should try to minimize redundant computations by storing intermediate results. For instance, in the liquidityRequired function, you should store the result of state.scaledTotalSupply - scaledWithdrawals in a variable instead of computing it many times, i have shown a sample code example below.


### my sample code
function liquidityRequired(
  MarketState memory state
) internal pure returns (uint256 _liquidityRequired) {
  uint256 scaledWithdrawals = state.scaledPendingWithdrawals;
  uint256 outstandingDebt = state.scaledTotalSupply - scaledWithdrawals;
  uint256 scaledRequiredReserves = outstandingDebt.bipMul(state.reserveRatioBips) + scaledWithdrawals;
  uint256 normalizedReserves = state.normalizeAmount(scaledRequiredReserves);
  return normalizedReserves + state.accruedProtocolFees + state.normalizedUnclaimedWithdrawals;
} 


#### github code reference(permalink)
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/libraries/MarketState.sol#L130-L136

### optimisation
 By storing intermediate results such as scaledWithdrawals, outstandingDebt, scaledRequiredReserves, and normalizedReserves, the function avoids redundant computations and multiple accesses to the state variables.

### IN SUMMMARY
Complex operations like bipMul and normalizeAmount are computed once and stored, reducing the need to repeat these operations multiple times.
Complex operations like bipMul and normalizeAmount are computed once and stored, reducing the need to repeat these operations multiple times.
The final calculation of liquidityRequired aggregates the precomputed values, ensuring that each value is computed  once only and used very efficiently.





# Gas optimisation for the hasPendingExpiredBatch function

#### github code reference(permalink)
 https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/libraries/MarketState.sol#L130-L136

### optimisation
In the hasPendingExpiredBatch function, logical operations and comparisons can be optimized. Instead of using assembly, which here is not too necessary, stick with Solidity’s built-in operations, which are usually optimized by the compiler. i have shown a code sample below

### my code sample
function hasPendingExpiredBatch(MarketState memory state) internal view returns (bool) {
  return state.pendingWithdrawalExpiry > 0 && state.pendingWithdrawalExpiry < block.timestamp;
}

### IN SUMMMARY
using solidity in this context is simple, performs direct comparisons, and involves minimal operations. For such straightforward logic, the Solidity compiler handles optimization effectively, and inline assembly is unlikely to offer significant improvements in comparison to using assembly




## Gas optimisation for functions like state.normaliseAmount

### optimisation
Inlining small functions directly into the functions where they are used can reduce the overhead of function calls, especially if they are simple and frequently used.
For example, instead of havig state.normalizeAmount as a separate function, you could inline its logic since it’s only used in a couple of places.

#### github code reference(permalink)
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/libraries/MarketState.sol#L66

 ### IN SUMMMARY
my suuggestion reduces function call overhead, leading to function call costs and gas savings.it also increases execution efficiency.





# GAS OPTIMISATION FOR FUNCTIONS IN THE rc/wildcatSanctionsEscrow.sol


## Gas optimisation for the canRealeaseEscrow 

#### github code reference(permalink)
 https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsEscrow.sol#L26-28

### optimisation
the canReleaseEscrow function calls the isSanctioned method from the IwildSanctionSentinel contract, and does not explictly store the result of the call, if cnaRealeased escrow is called multiple times within the same transactionit will make multiple external contract calls. i have shown a way to optimise the function below by fetching and storing the result of the isSanctioned call in a local variable, sanctioned.


### my code sample
function canReleaseEscrow() public view override returns (bool) {
    // Fetch and cache the result of the external call to avoid repeated calls
    bool sanctioned = IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, account);
    return !sanctioned;
}


### IN SUMMARY
in the event that canReleaseEscrow is called multiple times, gas usage can be optimised by caching the results of external calls to avoid repeated calls 



# GAS OPTIMISATION FOR FUNCTIONS IN THE src/libraries/LibStoredInitCode.sol

#### github code reference(permalink)
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/libraries/LibStoredInitCode.sol#L85-L123


### optimisation
a single exectcodecopy operation should be used wherever possible to reduce gas costs associated with copying code from an external address.In the original code, each function 
that deploys a contract from stored init code uses extcodecopy to copy the init code from the initCodeStorage address to memory.
 This operation involves: Copying a portion of code from the initCodeStorage address and The memory layout for the init code and additional arguments might differ.
To optimize, i reduced redundant extcodecopy operations by combining the operations where possible, such as handling constructor arguments within the same process rather than separately. This reduces the total number of operations and hence saves gas.
see my code below:

function createWithStoredInitCode(
  address initCodeStorage,
  uint256 value,
  bytes memory constructorArgs
) internal returns (address deployment) {
  assembly {
    let memPtr := mload(0x40)
    let initCodeSize := sub(extcodesize(initCodeStorage), 1)

    // Copy init code and optionally append constructor args
    extcodecopy(initCodeStorage, memPtr, 1, initCodeSize)
    let constructorArgsSize := mload(constructorArgs)
    if gt(constructorArgsSize, 0) {
      mcopy(add(memPtr, initCodeSize), add(constructorArgs, 0x20), constructorArgsSize)
      initCodeSize := add(initCodeSize, constructorArgsSize)
    }

    // Deploy the contract and handle errors
    deployment := create(value, memPtr, initCodeSize)
    if iszero(deployment) {
      mstore(0x00, 0x30116425) // DeploymentFailed()
      revert(0x1c, 0x04)
    }
  }
}

function create2WithStoredInitCode(
  address initCodeStorage,
  bytes32 salt,
  uint256 value,
  bytes memory constructorArgs
) internal returns (address deployment) {
  assembly {
    let memPtr := mload(0x40)
    let initCodeSize := sub(extcodesize(initCodeStorage), 1)

    // Copy init code and optionally append constructor args
    extcodecopy(initCodeStorage, memPtr, 1, initCodeSize)
    let constructorArgsSize := mload(constructorArgs)
    if gt(constructorArgsSize, 0) {
      mcopy(add(memPtr, initCodeSize), add(constructorArgs, 0x20), constructorArgsSize)
      initCodeSize := add(initCodeSize, constructorArgsSize)
    }

    // Deploy contract using CREATE2 and handle errors
    deployment := create2(value, memPtr, initCodeSize, salt)
    if iszero(deployment) {
      mstore(0x00, 0x30116425) // DeploymentFailed()
      revert(0x1c, 0x04)
    }
  }
}

### IN SUMMARY

in my code sampl, i used the extcodecopy to copy the init code and handle constructor arguments in a single pass where possible.
 This avoids multiple separate calls for copying code and appending arguments, which can be costly.i also copied constructor arguments right after copying the init code instead of copying constructor arguments in a separate step. ths consolidation reduces gas costs by minimysing redundant memory operations and copying.