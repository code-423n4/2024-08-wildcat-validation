## No access control on `executeWithdrawal` and `executeWithdrawals`

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L194

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L212

### Impact

`executeWithdrawal` and `executeWithdrawals`  allows anyone to execute withdrawal on behalf of other lenders. However, the lender at the time might not need or want the tokens withdrawn. For instance, a compromised address or wallet. So executing the withdrawal on their behalf may lead to potential fund loss.
```solidity
  function executeWithdrawal(
    address accountAddress,
    uint32 expiry
  ) public nonReentrant sphereXGuardExternal returns (uint256) {
```

```solidity
  function executeWithdrawals(
    address[] calldata accountAddresses,
    uint32[] calldata expiries
  ) external nonReentrant sphereXGuardExternal returns (uint256[] memory amounts) {
```

Recommend allowing users to execute withdrawals for themselves. 


***

## Sanction Overrides may are ineffective if the underlying asset also relies on chainalysis

* Lines of code
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsEscrow.sol#L34

### Impact

To [release escrow](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsEscrow.sol#L34), the account must no longer be sanctioned by chainalysis, or must have had their sanction overridden. 
```solidity
  function canReleaseEscrow() public view override returns (bool) {
    return !IWildcatSanctionsSentinel(sentinel).isSanctioned(borrower, account);
  }
```
However, some tokens outright rely on chainalysis oracle for their transfer action. An example is ondo's [USDY](https://etherscan.io/address/0xea0f7eebdc2ae40edfe33bf03d332f8a7f617528#code#F1#L95) which ensures that tokens cannot be transferred if the caller, receiver or sender is sanctioned by chainalysis. This renders sanction override useless, especially if the escrow address related to the account is also sanctioned by the chainalysis dossing protocol operations for the users.


***

## WildcatMarketToken may not need approval from owner to perform transferFrom

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketToken.sol#L56-L60

### Impact

WildcatMarketToken.sol's implementation of transferFrom may allow the token owner to not need approval before transferring. This can help cut down on unnecessary approvals for owners and smart contracts that use the "pull" method to transfer tokens.

```solidity
    if (allowed != type(uint256).max) {
      uint256 newAllowance = allowed - amount;
      _approve(from, msg.sender, newAllowance);
    }
```

### Recommended Mitigation Steps

Use this instead

```solidity
        if (allowed != type(uint256).max || msg.sender == from) {
          uint256 newAllowance = allowed - amount;
          _approve(from, msg.sender, newAllowance);
       }
```
***

## Asset may be stuck if token blocklists users, making sanction overrides useless

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatSanctionsEscrow.sol#L34-L45

### Impact

Protocol will be working with blocklisting token, accroding to the information provided in the readme and they can prevent escrow releases even if the account's sanction is overriden by the borrower.\

```solidity
  function releaseEscrow() public override {
    if (!canReleaseEscrow()) revert CanNotReleaseEscrow();

    uint256 amount = balance();
    address _account = account;
    address _asset = asset;

    asset.safeTransfer(_account, amount);

    emit EscrowReleased(_account, _asset, amount);
  }
```

***

## No access control in `nukeFromOrbit` may lead to forceful queing of withdrawals for users and blocking.

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketConfig.sol#L82-L88

### Impact

Due to the fact that by default, anyone sanctioned by chainalysis is automatically eligibe to be blocked because `sanctionOverrides[borrower][account]` is always false.

```solidity
  function isSanctioned(address borrower, address account) public view override returns (bool) {
    return !sanctionOverrides[borrower][account] && isFlaggedByChainalysis(account);
  }
```

Anyone can call the `nukeFromOrbit` function to block a potential user and forcefully queue their withdrawal. 

```solidity
  function nukeFromOrbit(address accountAddress) external nonReentrant sphereXGuardExternal {
    if (!_isSanctioned(accountAddress)) revert_BadLaunchCode();
    MarketState memory state = _getUpdatedState();
    hooks.onNukeFromOrbit(accountAddress, state);
    _blockAccount(state, accountAddress);
    _writeState(state);
  }
```

### Recommended Mitigation Steps

Recommend limiting this to the borrower who can get to decide which users can be sanctioned, which cannot and which to block.

***

## Markets cannot be deployed with tokens that do not return string for name and symbol

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L436-L437

### Impact

From the readme, tokens that return bytes32 name or string are supported

> ... and name, symbol and decimals must all return valid results (for name and symbol, either bytes32 or a string)...

But when deploying markets, an attempt is made to concatenate the string prefixes to the assset names and symbols. But attempting to concatenate a string to a bytes32 parameter will result in an EVM error.

```solidity
    string memory name = string.concat(parameters.namePrefix, parameters.asset.name());
    string memory symbol = string.concat(parameters.symbolPrefix, parameters.asset.symbol());
```
### Recommended Mitigation Steps

Recommend introducing a try catch to individually query asset name and symbol, checking its type and either concatenating as a string or a bytes32 parameter.

***

## Sanctioned users are still able to transfer WildcatMarketTokens

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketToken.sol#L49-65

### Impact

Consider preventing sanctioned users from being able to be approved and access the `transferFrom` function. This can be done by querying msg.sender's account status on the `transferFrom` function. As it stands, a sanctioned user can still transfer tokens from one user to another.

```solidity
  function transferFrom(
    address from,
    address to,
    uint256 amount
  ) external virtual nonReentrant sphereXGuardExternal returns (bool) {
    uint256 allowed = allowance[from][msg.sender];

    // Saves gas for unlimited approvals.
    if (allowed != type(uint256).max) {
      uint256 newAllowance = allowed - amount;
      _approve(from, msg.sender, newAllowance);
    }

    _transfer(from, to, amount, 0x64);

    return true;
  }
```
***
## Redundant check for hook template existence can be removed

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L530-L533

### Impact

`deployMarketAndHooks` checks for template existence before deployment. But this check is not needed as `_deployHooksInstance` already checks for that [here](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L295). Consider removing this check here.

```solidity
    HooksTemplate memory templateDetails = _templateDetails[hooksTemplate];
    if (!templateDetails.exists) {
      revert HooksTemplateNotFound();
    }
```


***

## `onQueueWithdrawal` should is missing the `isHooked` checks.

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/access/AccessControlHooks.sol#L812-L826

### Impact

Unlike its FixedTermLoanHooks.sol counterpart, `onQueueWithdrawal` in AccessControlHooks.sol, `onQueueWithdrawal` does not check that the market is hooked, which may lead to the hooks being called from the wrong market. Recommend adding the `if (!market.isHooked) revert NotHookedMarket();` check.

```solidity
  function onQueueWithdrawal(
    address lender,
    uint32 /* expiry */,
    uint /* scaledAmount */,
    MarketState calldata /* state */,
    bytes calldata hooksData
  ) external override {
    LenderStatus memory status = _lenderStatus[lender];
    if (
      !isKnownLenderOnMarket[lender][msg.sender] && !_tryValidateAccess(status, lender, hooksData)
    ) {
      revert NotApprovedLender();
    }
  }
```
***

## Most getter functions return one less than intended count

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L224

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L247

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L179

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L222

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L268

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L319

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L370
### Impact

`getHooksTemplates`, `getMarketsForHooksTemplate`, `getRegisteredBorrowers`, `getBlacklistedAssets`, `getRegisteredControllerFactories`, `getRegisteredControllers` and `getRegisteredMarkets` all follow the same pattern.

```solidity
  function getXXXX(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = XXXX.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = XXXX.at(start + i);
    }
  }
```

Because `count = end - start`, the condition `i < count` should be `i <= count` so as to include the final entry in the array. E.g to get the first 5 entries, the `start` = 0, and `end` = 4 - therefore `count` = 4; This means the new array `arr` is now made up of the 0, 1, 2, 3th elements because of the condition.

Recommend getting using the `i <= count` logic instead.

***

## Confusing ERC20 queries `balanceOf` and `totalSupply`

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketToken.sol#L17-L27

### Impact

The WildcatMarketToken.sol contract utilizes the standard ERC20 functions, `balanceOf` and `totalSupply`. However, these functions provide the balance of the underlying tokens rather than the market tokens. This mismatch between the function names and their true functionality may cause confusion or complications when interfacing with other protocols.

```solidity
  function balanceOf(address account) public view virtual nonReentrantView returns (uint256) {
    return
      _calculateCurrentStatePointers.asReturnsMarketState()().normalizeAmount(
        _accounts[account].scaledBalance
      );
  }

  /// @notice Returns the normalized total supply with interest.
  function totalSupply() external view virtual nonReentrantView returns (uint256) {
    return _calculateCurrentStatePointers.asReturnsMarketState()().totalSupply();
  }
```

Recommend renaming the current functions to balanceOfScaled and totalScaledSupply. Additionally, new implementations of balanceOf and totalSupply should be created to accurately reflect the balance of the market token



***

## Limit to maximum amount that can be borrowed due to scaled amounts being limited to `type(uint104).max`

* Lines of code

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/libraries/MarketState.sol#L20

https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarket.sol#L67

### Impact

According to the [README](https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/README.md#general-questions), the amount of assets that can be borrowed in a market should be up to type(uint128).max. But the `scaledAmount`, `scaledTotalSupply` and `scaledBalance` in use in various functions is only scaled up to a max of uint104. This limits the maximum amount that borrowers will be able to borrow especially if the assets have a max value greater than `type(uint104).max`. Recommend increasing the precision of `scaleFactor`, such as changing it to a uint128 instead.

```solidity
struct MarketState {
//...
  uint104 scaledTotalSupply;
//...
```

```solidity
struct Account {
  uint104 scaledBalance;
}
```

```solidity
  function _depositUpTo(
    uint256 amount
  ) internal virtual nonReentrant returns (uint256 /* actualAmount */) {
//...
    // Scale the mint amount
    uint104 scaledAmount = state.scaleAmount(amount).toUint104();
```

***
