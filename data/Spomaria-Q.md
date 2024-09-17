# [L-01]: The `WildcatMarketWithdrawals::_executeWithdrawal` emits wrong event whenever `accountAddress` is sanctioned


## Summary
The `WildcatMarketWithdrawals::_executeWithdrawal` function is designed to send requested tokens to the `accountAddress` or send the tokens to an escrow account if `accountAddress` is sanctioned. In addition, there are two events to be emitted by the function: `emit_SanctionedAccountWithdrawalSentToEscrow` if the tokens were sent to an escrow account; and `emit_WithdrawalExecuted` if the tokens were sent to the `accountAddress`.

However, in the event that `accountAddress` is sanctioned, the function emits both events i.e. saying that the tokens were sent to the `accountAddress` and also sent to the escrow account whereas in the real sense, the tokens were only sent to the escrow account and only the `emit_SanctionedAccountWithdrawalSentToEscrow` event ought to have been emitted. This can potentially mislead the frontend or some oracles that are listening to the events.

See https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L231-L281 where the affected portion is highlighted in the code snippet below
```javascript
  function _executeWithdrawal(
    MarketState memory state,
    address accountAddress,
    uint32 expiry,
    uint baseCalldataSize
  ) internal returns (uint256) {
    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];
    // If the market is closed, allow withdrawal prior to expiry.
    if (expiry >= block.timestamp && !state.isClosed) {
      revert_WithdrawalBatchNotExpired();
    }


    AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
      accountAddress
    ];


    uint128 newTotalWithdrawn = uint128(
      MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount)
    );


    uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;


    if (normalizedAmountWithdrawn == 0) revert_NullWithdrawalAmount();


    hooks.onExecuteWithdrawal(accountAddress, normalizedAmountWithdrawn, state, baseCalldataSize);


    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;


    if (_isSanctioned(accountAddress)) {
      // Get or create an escrow contract for the lender and transfer the owed amount to it.
      // They will be unable to withdraw from the escrow until their sanctioned
      // status is lifted on Chainalysis, or until the borrower overrides it.
      address escrow = _createEscrowForUnderlyingAsset(accountAddress);
      asset.safeTransfer(escrow, normalizedAmountWithdrawn);


      // Emit `SanctionedAccountWithdrawalSentToEscrow` event using a custom emitter.
      emit_SanctionedAccountWithdrawalSentToEscrow(
        accountAddress,
        escrow,
        expiry,
        normalizedAmountWithdrawn
      );
    } else {
      asset.safeTransfer(accountAddress, normalizedAmountWithdrawn);
    }


@>  emit_WithdrawalExecuted(expiry, accountAddress, normalizedAmountWithdrawn);


    return normalizedAmountWithdrawn;
  }
```

## Recommended Mitigation Steps

1. Adjust the function so that if `accountAddress` is sanctioned, only one event i.e. `emit_SanctionedAccountWithdrawalSentToEscrow` is emitted instead of both events.

```diff
  function _executeWithdrawal(
    MarketState memory state,
    address accountAddress,
    uint32 expiry,
    uint baseCalldataSize
  ) internal returns (uint256) {
    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];
    // If the market is closed, allow withdrawal prior to expiry.
    if (expiry >= block.timestamp && !state.isClosed) {
      revert_WithdrawalBatchNotExpired();
    }


    AccountWithdrawalStatus storage status = _withdrawalData.accountStatuses[expiry][
      accountAddress
    ];


    uint128 newTotalWithdrawn = uint128(
      MathUtils.mulDiv(batch.normalizedAmountPaid, status.scaledAmount, batch.scaledTotalAmount)
    );


    uint128 normalizedAmountWithdrawn = newTotalWithdrawn - status.normalizedAmountWithdrawn;


    if (normalizedAmountWithdrawn == 0) revert_NullWithdrawalAmount();


    hooks.onExecuteWithdrawal(accountAddress, normalizedAmountWithdrawn, state, baseCalldataSize);


    status.normalizedAmountWithdrawn = newTotalWithdrawn;
    state.normalizedUnclaimedWithdrawals -= normalizedAmountWithdrawn;


    if (_isSanctioned(accountAddress)) {
      // Get or create an escrow contract for the lender and transfer the owed amount to it.
      // They will be unable to withdraw from the escrow until their sanctioned
      // status is lifted on Chainalysis, or until the borrower overrides it.
      address escrow = _createEscrowForUnderlyingAsset(accountAddress);
      asset.safeTransfer(escrow, normalizedAmountWithdrawn);


      // Emit `SanctionedAccountWithdrawalSentToEscrow` event using a custom emitter.
      emit_SanctionedAccountWithdrawalSentToEscrow(
        accountAddress,
        escrow,
        expiry,
        normalizedAmountWithdrawn
      );
    } else {
      asset.safeTransfer(accountAddress, normalizedAmountWithdrawn);
+     emit_WithdrawalExecuted(expiry, accountAddress, normalizedAmountWithdrawn);
    }


-  emit_WithdrawalExecuted(expiry, accountAddress, normalizedAmountWithdrawn);


    return normalizedAmountWithdrawn;
  }
```


# [L-02]: The `WildcatMarketWithdrawals::_queueWithdrawal` emits event with wrong parameters

## Summary
The `WildcatMarketWithdrawals::_queueWithdrawal` function is designed to queue a lender's withdrawal request for future execution. The function also emits an event `emit_Transfer` to indicate that some amount of the assets are sent to the `accountAddress` from the `WildcatMarketWithdrawals`.

However, the parameters used in emitting the event are interchanged implying that assets are transfered or to be transfered from `accountAddress` to `address(this)` when in fact, it is the other way round. This can potentially mislead the frontend or some oracles that are listening to the events.

See https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/market/WildcatMarketWithdrawals.sol#L80-L130 where the affected portion is highlighted in the code snippet below
```javascript
    function _queueWithdrawal(
    MarketState memory state,
    Account memory account,
    address accountAddress,
    uint104 scaledAmount,
    uint normalizedAmount,
    uint baseCalldataSize
  ) internal returns (uint32 expiry) {


    // Cache batch expiry on the stack for gas savings
    expiry = state.pendingWithdrawalExpiry;


    // If there is no pending withdrawal batch, create a new one.
    if (state.pendingWithdrawalExpiry == 0) {
      // If the market is closed, use zero for withdrawal batch duration.
      uint duration = state.isClosed.ternary(0, withdrawalBatchDuration);
      expiry = uint32(block.timestamp + duration);
      emit_WithdrawalBatchCreated(expiry);
      state.pendingWithdrawalExpiry = expiry;
    }


    // Execute queueWithdrawal hook if enabled
    hooks.onQueueWithdrawal(accountAddress, expiry, scaledAmount, state, baseCalldataSize);


    // Reduce account's balance and emit transfer event
    account.scaledBalance -= scaledAmount;
    _accounts[accountAddress] = account;


@>  emit_Transfer(accountAddress, address(this), normalizedAmount);


    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];


    // Add scaled withdrawal amount to account withdrawal status, withdrawal batch and market state.
    _withdrawalData.accountStatuses[expiry][accountAddress].scaledAmount += scaledAmount;
    batch.scaledTotalAmount += scaledAmount;
    state.scaledPendingWithdrawals += scaledAmount;


    emit_WithdrawalQueued(expiry, accountAddress, scaledAmount, normalizedAmount);


    // Burn as much of the withdrawal batch as possible with available liquidity.
    uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());
    if (availableLiquidity > 0) {
      _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);
    }


    // Update stored batch data
    _withdrawalData.batches[expiry] = batch;


    // Update stored state
    _writeState(state);
  }
```

## Recommended Mitigation Steps

1. Edit the `emit_Transfer` function call so that `address(this)` is the `from` address and `accountAddress` is the `to` address as shown below:

```diff
    function _queueWithdrawal(
    MarketState memory state,
    Account memory account,
    address accountAddress,
    uint104 scaledAmount,
    uint normalizedAmount,
    uint baseCalldataSize
  ) internal returns (uint32 expiry) {


    // Cache batch expiry on the stack for gas savings
    expiry = state.pendingWithdrawalExpiry;


    // If there is no pending withdrawal batch, create a new one.
    if (state.pendingWithdrawalExpiry == 0) {
      // If the market is closed, use zero for withdrawal batch duration.
      uint duration = state.isClosed.ternary(0, withdrawalBatchDuration);
      expiry = uint32(block.timestamp + duration);
      emit_WithdrawalBatchCreated(expiry);
      state.pendingWithdrawalExpiry = expiry;
    }


    // Execute queueWithdrawal hook if enabled
    hooks.onQueueWithdrawal(accountAddress, expiry, scaledAmount, state, baseCalldataSize);


    // Reduce account's balance and emit transfer event
    account.scaledBalance -= scaledAmount;
    _accounts[accountAddress] = account;


-   emit_Transfer(accountAddress, address(this), normalizedAmount);
+   emit_Transfer(address(this), accountAddress, normalizedAmount);

    WithdrawalBatch memory batch = _withdrawalData.batches[expiry];


    // Add scaled withdrawal amount to account withdrawal status, withdrawal batch and market state.
    _withdrawalData.accountStatuses[expiry][accountAddress].scaledAmount += scaledAmount;
    batch.scaledTotalAmount += scaledAmount;
    state.scaledPendingWithdrawals += scaledAmount;


    emit_WithdrawalQueued(expiry, accountAddress, scaledAmount, normalizedAmount);


    // Burn as much of the withdrawal batch as possible with available liquidity.
    uint256 availableLiquidity = batch.availableLiquidityForPendingBatch(state, totalAssets());
    if (availableLiquidity > 0) {
      _applyWithdrawalBatchPayment(batch, state, expiry, availableLiquidity);
    }


    // Update stored batch data
    _withdrawalData.batches[expiry] = batch;


    // Update stored state
    _writeState(state);
  }
```
