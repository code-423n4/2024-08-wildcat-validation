
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
