# QA Report

##

## [L-] Address Predictability and Replay Attack Vulnerabilities in Contract Deployment Using create

### Impact

``Create`` uses the deployer’s address and current nonce to calculate the new contract’s address. This is where the issues of address predictability and replay attacks come into play.

The address of the newly deployed contract is calculated as

The deployer's address (the contract or account executing this code).
The nonce of the deployer, which is the number of contracts they have already deployed.

In a replay attack, the same contract creation transaction could be executed on another network or a fork of the blockchain where the deployer’s nonce is the same. Since create uses only the deployer’s address and nonce to calculate the address, this could result in the contract being deployed at the same address across different networks or forks.

#### Problems
 - Since the nonce is incremented with each contract deployment, anyone who knows the deployer's address and current nonce can predict the address of the contract that will be deployed next. 
 - If the deployed contract interacts with network-specific assets (like tokens or governance functions), it could cause unintended behavior, such as duplicate contract instances interacting with the same external systems.

```soldiity
FILE: 2024-08-wildcat/src/HooksFactory.sol

hooksInstance := create(0, initCodePointer, initCodeSizeWithArgs)

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L319

### Recommended Mitigation
By implementing ``create2``, your contract deployment process becomes more secure, protecting against both front-running and replay attacks.

The salt is a value you choose, and it allows you to control the address deterministically but unpredictably.

```solidity
hooksInstance := create2(0, initCodePointer, initCodeSizeWithArgs, someUniqueSalt)

```

##

## [L-] Deactivated Templates Vulnerable to Unintended Updates 

Once a template is disabled using ``disableHooksTemplate``, the expectation is that it should no longer be modifiable, but the current implementation doesn't enforce this. As a result, fees and other properties can still be updated for disabled templates, which might not align with the intended behavior.

```solidity
FILE: 2024-08-wildcat/src/HooksFactory.sol

/// @dev Update the fees for a hooks template
  /// Note: The new fee structure will apply to all NEW markets created with existing
  ///       or future instances of the hooks template, and the protocol fee can be pushed
  ///       to existing markets using `pushProtocolFeeBipsUpdates`.
  function updateHooksTemplateFees(
    address hooksTemplate,
    address feeRecipient,
    address originationFeeAsset,
    uint80 originationFeeAmount,
    uint16 protocolFeeBips
  ) external override onlyArchControllerOwner {
    if (!_templateDetails[hooksTemplate].exists) {
      revert HooksTemplateNotFound();
    }
    _validateFees(feeRecipient, originationFeeAsset, originationFeeAmount, protocolFeeBips);
    HooksTemplate storage template = _templateDetails[hooksTemplate];
    template.feeRecipient = feeRecipient;
    template.originationFeeAsset = originationFeeAsset;
    template.originationFeeAmount = originationFeeAmount;
    template.protocolFeeBips = protocolFeeBips;
    emit HooksTemplateFeesUpdated(
      hooksTemplate,
      feeRecipient,
      originationFeeAsset,
      originationFeeAmount,
      protocolFeeBips
    );
  }

  function disableHooksTemplate(address hooksTemplate) external override onlyArchControllerOwner {
    if (!_templateDetails[hooksTemplate].exists) {
      revert HooksTemplateNotFound();
    }
    _templateDetails[hooksTemplate].enabled = false;
    // Emit an event to indicate that the template has been removed
    emit HooksTemplateDisabled(hooksTemplate);
  }

```

### Recommended Mitigation 
Modify the logic only updates possible when the template is active 

```diff

 function updateHooksTemplateFees(
    address hooksTemplate,
    address feeRecipient,
    address originationFeeAsset,
    uint80 originationFeeAmount,
    uint16 protocolFeeBips
  ) external override onlyArchControllerOwner {
+    if (!_templateDetails[hooksTemplate].exists || !_templateDetails[hooksTemplate].enabled ) {
      revert HooksTemplateNotFound();
    }
    _validateFees(feeRecipient, originationFeeAsset, originationFeeAmount, protocolFeeBips);
    HooksTemplate storage template = _templateDetails[hooksTemplate];
    template.feeRecipient = feeRecipient;
    template.originationFeeAsset = originationFeeAsset;
    template.originationFeeAmount = originationFeeAmount;
    template.protocolFeeBips = protocolFeeBips;
    emit HooksTemplateFeesUpdated(
      hooksTemplate,
      feeRecipient,
      originationFeeAsset,
      originationFeeAmount,
      protocolFeeBips
    );
  }

```





##

## [L-] Governance Conflict in Controller Management Due to Separate Addition and Removal Authorities

The ControllerFactory could add a controller that introduces vulnerabilities or unwanted features, and the owner may not be able to remove it quickly enough to prevent damage. During the time it takes for the owner to remove the controller, the system could be exposed to exploitation.

```solidity
FILE:2024-08-wildcat/src
/WildcatArchController.sol

function registerController(address controller) external onlyControllerFactory {
    if (!_controllers.add(controller)) {
      revert ControllerAlreadyExists();
    }
    _addAllowedSenderOnChain(controller);
    emit ControllerAdded(msg.sender, controller);
  }

  function removeController(address controller) external onlyOwner {
    if (!_controllers.remove(controller)) {
      revert ControllerDoesNotExist();
    }
    emit ControllerRemoved(controller);
  }

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L296-L309

## Recommended Mitigation
Add the onlyOwner access control for both registerController() and removeController() functions

##

## [L-] 

##

## [L] Potential for Indirect Reentrancy in updateSphereXEngineOnRegisteredContracts() function

Even with access control, it's still important to consider reentrancy protections when dealing with low-level call() operations.

Even if the ``call()`` target is controlled by a contract that implements access control, the reentrancy risk comes from how the external contract could behave. If the external contract (the ``target`` in ``_callWith``) is compromised or implements malicious logic, it can invoke other contracts or itself within the same transaction in ways that circumvent the access control. For instance, the external contract could call back into the original contract or exploit any other unprotected function. 

Example Scenario:
The external contract A receives the call via call() and immediately calls back into the contract making the original call (WildcatArchController in this case). If the WildcatArchController has any other publicly accessible functions that don't have reentrancy protections, this callback could reenter those functions and cause unintended behavior.

```solidity
FILE: 2024-08-wildcat/src/WildcatArchController.sol

function _updateSphereXEngineOnRegisteredContractsInSet(
    EnumerableSet.AddressSet storage set,
    address engineAddress,
    address[] memory contracts,
    bytes memory changeSphereXEngineCalldata,
    bytes memory addAllowedSenderOnChainCalldata,
    bytes4 notInSetErrorSelectorBytes
  ) internal {
    for (uint256 i = 0; i < contracts.length; i++) {
      address account = contracts[i];
      if (!set.contains(account)) {
        uint32 notInSetErrorSelector = uint32(notInSetErrorSelectorBytes);
        assembly {
          mstore(0, notInSetErrorSelector)
          revert(0x1c, 0x04)
        }
      }
      _callWith(account, changeSphereXEngineCalldata);
      if (engineAddress != address(0)) {
        assembly {
          mstore(add(addAllowedSenderOnChainCalldata, 0x24), account)
        }
        _callWith(engineAddress, addAllowedSenderOnChainCalldata);
        emit_NewAllowedSenderOnchain(account);
      }
    }

 function _callWith(address target, bytes memory data) internal {
    assembly {
      if iszero(call(gas(), target, 0, add(data, 0x20), mload(data), 0, 0)) {
        returndatacopy(0, 0, returndatasize())
        revert(0, returndatasize())
      }
    }
  }

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L144-L150

### Recommended Mitigation
Apply the ``Checks-Effects-Interactions`` pattern to ensure that state changes are made before any external calls. Additionally, consider using the ``ReentrancyGuard`` from ``OpenZeppelin`` to prevent reentrant calls.

##

## [] Lack of contract existence check when using assembly calls 

```
The low-level functions call, delegatecall and staticcall return true as their first
return value if the account called is non-existent, as part of the design of the
EVM. Account existence must be checked prior to calling if needed.

```
 A snippet of the Solidity documentation detailing unexpected behavior related to call


It is important to check whether the target address is a valid smart contract before performing a low-level call. Without this check, the call may fail if the target is an EOA or a non-existent contract, potentially causing unintended behavior.

```solidity
FILE: 2024-08-wildcat/src/WildcatArchController.sol

146:  if iszero(call(gas(), target, 0, add(data, 0x20), mload(data), 0, 0)) {

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L146

## Recommended Mitigation
Implement a contract existence check before each assembly call 


##

## [L-1] Redundant address(0) Validation 

The engineAddress is fetched from a predefined storage slot (SPHEREX_ENGINE_STORAGE_SLOT), which is set to hold a constant and valid address. Since this slot is guaranteed to contain a valid non-zero address, engineAddress can never be address(0).

```solidity
FILE: 2024-08-wildcat/src/WildcatArchController.sol

84: if (engineAddress != address(0)) {

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L84

### Recommended Mitigation
Remove redundant check 

##

## [] Brittle and Inefficient Assembly-based Custom Error Handling

The code uses assembly to revert with a custom error (revert(0x1c, 0x04)). This is inefficient and brittle for modern Solidity code. The usage of hardcoded memory offsets for error data (e.g., 0x1c and 0x04) could be incorrect if the error handling format changes, leading to unexpected behavior.

```solidity
FILE:2024-08-wildcat/src/WildcatArchController.sol

assembly {
          mstore(0, notInSetErrorSelector)
          revert(0x1c, 0x04)
        }

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L128-L132

## Recommended Mitigation
Consider using Solidity's native require or revert functionality instead of assembly to avoid the need to handle memory manually.

##

## [L-] Unrestricted Gas Forwarding in External Calls Leading to Potential Out-of-Gas Errors

The code does not control gas usage properly in the call() function in _callWith(). Since it forwards all available gas (gas()), it can potentially lead to out-of-gas errors in the calling contract if the target contract consumes excessive gas.

```solidity
FILE: 2024-08-wildcat/src/WildcatArchController.sol

 function _callWith(address target, bytes memory data) internal {
    assembly {
      if iszero(call(gas(), target, 0, add(data, 0x20), mload(data), 0, 0)) {
        returndatacopy(0, 0, returndatasize())
        revert(0, returndatasize())
      }
    }

```

## Recommended Mitigation
Limit the gas passed to external calls to avoid running out of gas in the calling contract. You can modify the assembly block like so:

```solidity

 call(gasleft(), target, 0, add(data, 0x20), mload(data), 0, 0)

```

##

## [L-] Potential for DoS by returning overly large arrays

The arr array is created dynamically based on the count of assets to return, which could result in an overly large array if the difference between start and end is large. This can lead to excessive memory usage and higher gas costs, potentially causing the function to fail or making it cost-prohibitive to use.

```solidity
FILE: 2024-08-wildcat/src/WildcatArchController.sol

function getRegisteredBorrowers(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _borrowers.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _borrowers.at(start + i);
    }
  }

function getBlacklistedAssets(
    uint256 start,
    uint256 end
  ) external view returns (address[] memory arr) {
    uint256 len = _assetBlacklist.length();
    end = MathUtils.min(end, len);
    uint256 count = end - start;
    arr = new address[](count);
    for (uint256 i = 0; i < count; i++) {
      arr[i] = _assetBlacklist.at(start + i);
    }
  }


```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/WildcatArchController.sol#L222-L233

### Recommend Mitigation 
In addition to limiting the batch size (as in the previous point), ensure that the function is only called with reasonable ranges.

```solidity

require(count <= maxBatchSize, "Requested range is too large");

```

##

## [L-] Lack of input validations when assigning values to immutable variables

The addresses passed to ``archController_``, ``_sanctionsSentinel``, and ``_marketInitCodeStorage`` are assigned directly without validation. If any of these addresses are incorrectly set (e.g., set to a zero address or malicious address), the contract might malfunction or become vulnerable to security exploits.

 ``_marketInitCodeHash`` is assigned directly without any validation or checks on the integrity or correctness of the input. If an incorrect hash is passed, the contract might fail to operate as expected, particularly in situations where market code initialization is critical.

If the constructor allows external or untrusted parties to pass in values, it opens up a risk of malicious input, which could compromise the security or functionality of the contract.

```solidity
FILE: 2024-08-wildcat/src/HooksFactory.sol

 constructor(
    address archController_,
    address _sanctionsSentinel,
    address _marketInitCodeStorage,
    uint256 _marketInitCodeHash
  ) {
    marketInitCodeStorage = _marketInitCodeStorage;
    marketInitCodeHash = _marketInitCodeHash;
    _archController = archController_;
    sanctionsSentinel = _sanctionsSentinel;
    __SphereXProtectedRegisteredBase_init(IWildcatArchController(archController_).sphereXEngine());
  }

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L55-L66

## Recommended Mitigation
Add ``address(0)`` check and uint checks like > 0 and min,max value checks

##

## [L-] Lack of Validation for Critical Market Parameters Leads to Potential Misconfiguration and Security Risks 

The issue here involves missing validation checks for critical parameters when deploying new markets, specifically for parameters like parameters.maxTotalSupply, parameters.withdrawalBatchDuration, parameters.reserveRatioBips, and parameters.delinquencyGracePeriod. These parameters likely play an important role in the functionality and behavior of the new market, and failing to validate them properly before deployment introduces several risks, such as misconfiguration, abuse, or security vulnerabilities.

```solidity
FILE:2024-08-wildcat/src/HooksFactory.sol

 TmpMarketParameterStorage memory tmp = TmpMarketParameterStorage({
      borrower: msg.sender,
      asset: parameters.asset,
      packedNameWord0: bytes32(0),
      packedNameWord1: bytes32(0),
      packedSymbolWord0: bytes32(0),
      packedSymbolWord1: bytes32(0),
      decimals: decimals,
      feeRecipient: templateDetails.feeRecipient,
      protocolFeeBips: templateDetails.protocolFeeBips,
      maxTotalSupply: parameters.maxTotalSupply,
      annualInterestBips: parameters.annualInterestBips,
      delinquencyFeeBips: parameters.delinquencyFeeBips,
      withdrawalBatchDuration: parameters.withdrawalBatchDuration,
      reserveRatioBips: parameters.reserveRatioBips,
      delinquencyGracePeriod: parameters.delinquencyGracePeriod,
      hooks: parameters.hooks
    });
    {

```
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L439-L457

### Recommended Mitigation
Add required checks >0 ,min, max value checks before assigning values 

##

## [L-] New Market Deployment Allowed Even After Hooks Template Is Disabled

Yes,``!template.enabled`` is checked when deploying newHook instance but not checked when deploy new markets. Its possible to can set  template.enabled false after deployed new hook instance.

The disableHooksTemplate() function is used to disable a hooksTemplate by setting enabled to false. However, in the deployMarket() function, there is no check to confirm whether the hooksTemplate is still enabled before proceeding with the deployment of a new market.

This could lead to unauthorized market deployments based on disabled templates, which may no longer be approved or secure for use.

```solidity
FILE:2024-08-wildcat/src/HooksFactory.sol

  function deployMarket(
    DeployMarketInputs calldata parameters,
    bytes calldata hooksData,
    bytes32 salt,
    address originationFeeAsset,
    uint256 originationFeeAmount
  ) external override nonReentrant returns (address market) {
    if (!IWildcatArchController(_archController).isRegisteredBorrower(msg.sender)) {
      revert NotApprovedBorrower();
    }
    address hooksInstance = parameters.hooks.hooksAddress();
    address hooksTemplate = getHooksTemplateForInstance[hooksInstance];
    if (hooksTemplate == address(0)) {
      revert HooksInstanceNotFound();
    }
    HooksTemplate memory templateDetails = _templateDetails[hooksTemplate];
    market = _deployMarket(
      parameters,
      hooksData,
      hooksTemplate,
      templateDetails,
      salt,
      originationFeeAsset,
      originationFeeAmount
    );
  }


``` 
https://github.com/code-423n4/2024-08-wildcat/blob/fe746cc0fbedc4447a981a50e6ba4c95f98b9fe1/src/HooksFactory.sol#L491-L516

### Recommended Mitigation

Add the ``!hooksTemplate.enabled`` check in ``deployMarket`` function

```solidity
if (!hooksTemplate.enabled){ revert;}

```




