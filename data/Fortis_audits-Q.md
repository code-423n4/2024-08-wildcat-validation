# QA Report

## Unbounded Loop Vulnerability in WildCat Contracts May Lead to Gas Exhaustion and DoS

### Summary:

The WildCat protocol contains many functions implementing unbounded loops, which processes elements without imposing any upper limit on the number of iterations. This leads to a potential vulnerability where a large number of iterations could cause gas exhaustion, rendering the transaction to fail. The result is a Denial of Service (DoS) condition for users, preventing them from successfully executing the function when the loop exceeds the gas limit.

### Impact:

Unbounded loops pose a critical risk for any function as they may lead to:

- **Transaction Failure**: If the number of iterations in the loop exceeds the gas limit, the transaction will run out of gas and revert.
- **Denial of Service (DoS)**: Attackers can exploit this unbounded loop by triggering transactions with a large input size, effectively causing a DoS condition where legitimate users are unable to interact with the contract.
- **Increased Gas Costs**: Even in non-malicious scenarios, executing large loops will incur disproportionately high gas costs, making the function inefficient and costly for regular users.

This vulnerability can degrade the functionality of the contract.

#### Affected Functions:

These are some of the functions that contain unbounded loops in the WildCat protocol:

1. `grantRoles()`:

```javascript
function grantRoles(address[] memory accounts, uint32[] memory roleGrantedTimestamps) external {
   RoleProvider callingProvider = _roleProviders[msg.sender];

        if (callingProvider.isNull()) revert ProviderNotFound();

        if (accounts.length != roleGrantedTimestamps.length) revert InvalidArrayLength();
        for (uint256 i = 0; i < accounts.length; i++) {
            _grantRole(callingProvider, accounts[i], roleGrantedTimestamps[i]);
        }
}
```

2. `_loopTryGetCredential()`:

```javascript
function _loopTryGetCredential(LenderStatus memory status, address accountAddress, uint256 pullProviderIndexToSkip)
        internal
        view
        returns (bool foundCredential)
    {
        uint256 providerCount = _pullProviders.length;
        for (uint256 i = 0; i < providerCount; i++) {
            if (i == pullProviderIndexToSkip) continue;
            RoleProvider provider = _pullProviders[i];
            if (_tryGetCredential(status, provider, accountAddress)) return (true);
        }
    }
```

3. `getRegisteredBorrowers()`:

```javascript
function getRegisteredBorrowers(uint256 start, uint256 end) external view returns (address[] memory arr) {
        uint256 len = _borrowers.length();
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i < count; i++) {
            arr[i] = _borrowers.at(start + i);
        }
    }
```

4. `getBlacklistedAssets()`:

```javascript
function getBlacklistedAssets(uint256 start, uint256 end) external view returns (address[] memory arr) {
        uint256 len = _assetBlacklist.length();
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i < count; i++) {
            arr[i] = _assetBlacklist.at(start + i);
        }
    }
```

5. `getRegisteredControllerFactories()`:

```javascript
function getRegisteredControllerFactories(uint256 start, uint256 end)
        external
        view
        returns (address[] memory arr)
    {
        uint256 len = _controllerFactories.length();
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i < count; i++) {
            arr[i] = _controllerFactories.at(start + i);
        }
    }
```

6. `getRegisteredPullProviders()`:

```javascript
function getRegisteredMarkets(uint256 start, uint256 end) external view returns (address[] memory arr) {
        uint256 len = _markets.length();
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i < count; i++) {
            arr[i] = _markets.at(start + i);
        }
    }
```

7. `getRegisteredControllers()`:

```javascript
function getRegisteredControllers(uint256 start, uint256 end) external view returns (address[] memory arr) {
        uint256 len = _controllers.length();
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i < count; i++) {
            arr[i] = _controllers.at(start + i);
        }
    }
```

8. `getHooksTemplates()`:

```javascript
function getHooksTemplates(uint256 start, uint256 end) external view override returns (address[] memory arr) {
        uint256 len = _hooksTemplates.length;
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i < count; i++) {
            arr[i] = _hooksTemplates[start + i];
        }
    }
```

9. `getMarketsForHooksTemplate()`:

```javascript
function getMarketsForHooksTemplate(address hooksTemplate, uint256 start, uint256 end)
        external
        view
        override
        returns (address[] memory arr)
    {
        address[] storage markets = _marketsByHooksTemplate[hooksTemplate];
        uint256 len = markets.length;
        end = MathUtils.min(end, len);
        uint256 count = end - start;
        arr = new address[](count);
        for (uint256 i = 0; i < count; i++) {
            arr[i] = markets[start + i];
        }
    }
```

10. `executeWithdrawals()`:

```javascript
function executeWithdrawals(address[] calldata accountAddresses, uint32[] calldata expiries)
        external
        nonReentrant
        sphereXGuardExternal
        returns (uint256[] memory amounts)
    {
        if (accountAddresses.length != expiries.length) revert_InvalidArrayLength();

        amounts = new uint256[](accountAddresses.length);

        MarketState memory state = _getUpdatedState();

        for (uint256 i = 0; i < accountAddresses.length; i++) {
            // Use calldatasize() for baseCalldataSize to indicate no data should be passed as `extraData`
            amounts[i] = _executeWithdrawal(state, accountAddresses[i], expiries[i], msg.data.length);
        }
        // Update stored state
        _writeState(state);
        return amounts;
    }
```

### Proof of Concept (PoC):

#### Example Scenario:

Consider the following scenario where the unbounded loop is exploited:

1. A user calls the `[Function Name]` function, which iterates over a large dataset or runs a potentially infinite loop.
2. If the dataset size is significantly large or the loop condition remains true for an extended number of iterations, the function will consume all available gas.
3. The transaction fails due to gas exhaustion, and the user is unable to interact with the contract.
4. An attacker can exploit this by providing intentionally large inputs, leading to continuous failure of legitimate user transactions and causing a DoS attack on the contract.

### Recommendations:

To prevent gas exhaustion and potential DoS attacks, we recommend the following mitigations:

**Implement Batching for Large Data Sets**:
Break down large datasets into smaller, manageable batches to avoid processing them in a single transaction. This can be achieved by implementing pagination mechanisms or processing data across multiple transactions.
