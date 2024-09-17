## Impact
unnecessary use of address() to convert already address type parameter into address type which cost additional gas.

## Proof of Concept
In FixedTermLoanHooks.sol _onCreateMarket function has marketAddress parameter which is type address.

```
 function _onCreateMarket(
    address deployer,
    address marketAddress,
    DeployMarketInputs calldata parameters,
    bytes calldata hooksData
  ) internal override returns (HooksConfig marketHooksConfig)
```
2.  In line 222 of FixedTermLoanHooks.sol , in _onCreateMarket . It convert marketAddress to address but it is already a address type.
```
    _hookedMarkets[address(marketAddress)] = hookedMarket;
```
## Tools Used
Manual Review

## Recommended Mitigation Steps
---     _hookedMarkets[address(marketAddress)] = hookedMarket;
+++        _hookedMarkets[marketAddress] = hookedMarket;