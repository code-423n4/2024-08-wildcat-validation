Cache Array Length Outside of Loop:
Description
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

```
  2024-08-wildcat/src/HooksFactory.sol::140 => index: uint24(_hooksTemplates.length)
  2024-08-wildcat/src/HooksFactory.sol::228 => uint256 len = _hooksTemplates.length;
  2024-08-wildcat/src/HooksFactory.sol::238 => return _hooksTemplates.length;
  2024-08-wildcat/src/HooksFactory.sol::253 => uint256 len = markets.length;
  2024-08-wildcat/src/HooksFactory.sol::265 => return _marketsByHooksTemplate[hooksTemplate].length;
  2024-08-wildcat/src/HooksFactory.sol::312 => let constructorArgsSize := constructorArgs.length
  2024-08-wildcat/src/HooksFactory.sol::378 => let length := mload(str)
  2024-08-wildcat/src/HooksFactory.sol::381 => if gt(length, 0x3f) {
  2024-08-wildcat/src/HooksFactory.sol::464 => if (market.code.length != 0) {
  2024-08-wildcat/src/HooksFactory.sol::562 => marketEndIndex = MathUtils.min(marketEndIndex, markets.length);
  2024-08-wildcat/src/WildcatArchController.sol::183 => uint256 len = _borrowers.length();
  2024-08-wildcat/src/WildcatArchController.sol::193 => return _borrowers.length();
  2024-08-wildcat/src/WildcatArchController.sol::236 => return _assetBlacklist.length();
  2024-08-wildcat/src/WildcatArchController.sol::272 => uint256 len = _controllerFactories.length();
  2024-08-wildcat/src/WildcatArchController.sol::282 => return _controllerFactories.length();
  2024-08-wildcat/src/WildcatArchController.sol::323 => uint256 len = _controllers.length();
  2024-08-wildcat/src/WildcatArchController.sol::333 => return _controllers.length();
  2024-08-wildcat/src/WildcatArchController.sol::374 => uint256 len = _markets.length();
  2024-08-wildcat/src/WildcatArchController.sol::384 => return _markets.length();
  2024-08-wildcat/src/access/AccessControlHooks.sol::226 => isPullProvider ? uint24(_pullProviders.length) : NotPullProviderIndex
  2024-08-wildcat/src/access/AccessControlHooks.sol::547 => let dataLength := sub(hooksData.length, 0x14)
  2024-08-wildcat/src/access/AccessControlHooks.sol::593 => uint256 providerCount = _pullProviders.length;


  2024-08-wildcat/src/access/FixedTermLoanHooks.sol::263 => isPullProvider ? uint24(_pullProviders.length) : NotPullProviderIndex
  2024-08-wildcat/src/access/FixedTermLoanHooks.sol::435 => if (accounts.length != roleGrantedTimestamps.length) revert InvalidArrayLength();
  2024-08-wildcat/src/access/FixedTermLoanHooks.sol::584 => let dataLength := sub(hooksData.length, 0x14)
  2024-08-wildcat/src/access/FixedTermLoanHooks.sol::630 => uint256 providerCount = _pullProviders.length;


  2024-08-wildcat/src/libraries/LibStoredInitCode.sol::168 => let constructorArgsSize := constructorArgs.length
  2024-08-wildcat/src/market/WildcatMarket.sol::273 => uint256 numBatches = _withdrawalData.unpaidBatches.length();
  2024-08-wildcat/src/market/WildcatMarket.sol::305 => msg.data.length
  2024-08-wildcat/src/market/WildcatMarketWithdrawals.sol::218 => amounts = new uint256[](accountAddresses.length);
  2024-08-wildcat/src/market/WildcatMarketWithdrawals.sol::224 => amounts[i] = _executeWithdrawal(state, accountAddresses[i], expiries[i], msg.data.length);
  2024-08-wildcat/src/market/WildcatMarketWithdrawals.sol::306 => uint256 numBatches = MathUtils.min(maxBatches, _withdrawalData.unpaidBatches.length());
```

Example

```
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
```

```
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}
```




Use immutable for OpenZeppelin AccessControl's Roles Declarations
Description
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

```
  2024-08-wildcat/src/WildcatSanctionsSentinel.sol::14 => keccak256(type(WildcatSanctionsEscrow).creationCode);
  2024-08-wildcat/src/WildcatSanctionsSentinel.sol::64 => salt := keccak256(0, 0x60)

  2024-08-wildcat/src/spherex/SphereXConfig.sol::15 => bytes32(uint256(keccak256('eip1967.spherex.spherex')) - 1);
  2024-08-wildcat/src/spherex/SphereXConfig.sol::17 => bytes32(uint256(keccak256('eip1967.spherex.pending')) - 1);
  2024-08-wildcat/src/spherex/SphereXConfig.sol::19 => bytes32(uint256(keccak256('eip1967.spherex.operator')) - 1);
  2024-08-wildcat/src/spherex/SphereXConfig.sol::21 => bytes32(uint256(keccak256('eip1967.spherex.spherex_engine')) - 1);
  2024-08-wildcat/src/spherex/SphereXProtectedRegisteredBase.sol::29 => bytes32(uint256(keccak256('eip1967.spherex.spherex_engine')) - 1);
```

Example

```
🤦 Bad:

bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
```

```
🚀 Good:

bytes32 public immutable GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
```